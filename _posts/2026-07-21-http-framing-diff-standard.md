# Differential testing minimum standard for HTTP framing

Date: 2026-07-21

This note is the public checklist I use before calling a parser disagreement a
security issue. It exists because I previously failed that checklist.

## Labels first

Classify every byte stream before you argue about impact:

| Class | Use |
|---|---|
| valid | negative control / baseline |
| invalid-syntax | must not be sold as a clever bypass |
| ambiguous-framing | candidate for multi-component tests |
| incomplete-stream | streaming wait; **not smuggling** |
| known-cve-seed | public regression only |

## The five gates

1. **Specification or invariant**  
   Name the RFC clause or the product contract that is broken.

2. **Complete, fragmented, and EOF cases**  
   Stream the same logical message in one shot and in pieces. Watch EOF with a
   declared length still outstanding.

3. **Negative controls**  
   Normal traffic and pure syntax garbage should not produce a “finding.”

4. **Two real components**  
   Same complete byte stream into two parsers or proxy/backend roles. Record
   message boundaries, not just one internal field.

5. **Impact oracle**  
   Auth bypass, cache poison, wrong routing, memory corruption, or another
   concrete security effect. Without this, you have an observation.

## Hard bans

- Declared chunk larger than received data → incomplete, not CWE-444.
- Text that looks like `GET /admin` inside an unfinished chunk → still body.
- `FFFFF F` without `;` → not a valid chunk extension.
- “MITRE received my email” → not “CVE pending.”
- One parser’s `content_length` field → not a differential.

## Tooling

- Offline variants: `http_smuggle_fuzzer.py`
- Observation harness: `framing_diff.py` (verdict is always observation-only
  until a human fills an impact report)
- Labeled samples: `http11-parser-corpus`

## Outcome vocabulary

Use these words and no stronger ones until the gates pass:

- observation
- differential candidate
- reported to vendor/CNA
- validated by vendor
- CVE assigned
- fixed / published

If a gate fails, publish the correction. That is research quality, not a setback.
