---
layout: post
title: "I reported a parser bug too early. Here is how the claim failed."
date: 2026-07-15
last_modified_at: 2026-07-21
---

I originally described an oversized-chunk input to `nodejs/http-parser` as a
request-smuggling precondition. That conclusion was wrong, and I prepared a
correction for the CNA request rather than leave the unsupported claim in the
record.

The input declared a chunk size of `0xFFFFF` and then supplied a short byte
sequence containing text that looked like another HTTP request. The parser
passed those bytes to its body callback, returned no syntax error for the bytes
received so far, and retained a large remaining content length.

I treated “one message parsed” and “the second request appeared in the body
callback” as evidence of a boundary failure. It was actually evidence that I
had told a streaming parser to expect about one megabyte of chunk data and had
not supplied it yet.

`http_parser` is designed to be called repeatedly as data arrives on one TCP
connection. Bytes do not become a second request merely because they spell
`GET /admin`; while the declared chunk is incomplete, those bytes are body.
The remaining content length was the strongest clue, and I interpreted it in
the wrong direction.

There were two more failures in my validation:

- I called `FFFFF F` a valid chunk extension, but RFC 9112 extensions use a
  semicolon.
- I used only one parser, so I never demonstrated the front-end/back-end
  disagreement required for request smuggling.

The corrected research rule is straightforward: a parser anomaly is not a
security finding. For framing issues, the harness needs complete and fragmented
input tests, EOF behavior, valid syntax, negative controls, and two real
components that disagree on the same byte stream. Impact has to come from the
harness, not from a story added after looking at one internal field.

Keeping this correction public is more useful than preserving the original
claim. It documents exactly which checks were missing and gives the next test a
clearer standard to meet.
