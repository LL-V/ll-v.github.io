# LL-V

Notes on network protocol parser safety.

I read protocol implementations — HTTP parsers, DNS resolvers, TLS
state machines — and look for the places where two interpreters
disagree on what a message means. That disagreement is where request
smuggling, response splitting, and state-machine desync live.

## Posts

- 2026-07-21 — [Differential testing minimum standard for HTTP framing]({% post_url 2026-07-21-http-framing-diff-standard %})
- 2026-07-15 — [I reported a parser bug too early. Here is how the claim failed.]({% post_url 2026-07-15-http-parser-smuggling-finding %})

