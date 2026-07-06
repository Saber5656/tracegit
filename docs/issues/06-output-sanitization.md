# 06 — Terminal-output sanitization

## Summary
Implement `render.Sanitize`, the security-critical function that neutralizes terminal escape
sequences and control characters in untrusted note/git content before human-mode display.

## Context
DESIGN §9.2 and ADR-004: note bodies and git author/summary fields are attacker-influenceable and
must never reach a TTY as raw bytes. This is the highest-value security control in the product.

## Scope
- `internal/render/sanitize.go`.

## Detailed Requirements
1. `Sanitize(s string) string` for human output:
   - Remove the ESC byte `0x1b` and any escape sequence it introduces (CSI `ESC [ ... final`,
     OSC `ESC ] ... (BEL|ST)`, DCS/SOS/PM/APC). Be conservative: on encountering ESC, drop until
     a plausible sequence terminator, and drop a lone trailing ESC.
   - Remove BEL `0x07`.
   - Remove all other C0 control characters (`0x00`–`0x1f`) **except** `\t` (0x09) and `\n` (0x0a).
   - Remove DEL `0x7f` and C1 controls (`0x80`–`0x9f`).
   - Preserve all other valid printable UTF-8 (Latin, CJK, RTL scripts, emoji, combining marks).
   - Replace invalid UTF-8 bytes with U+FFFD rather than emitting raw bytes.
2. `SanitizeInline(s string) string`: like `Sanitize` but also converts `\t`/`\n` to a single
   space and collapses runs — for one-line summaries in blame/log.
3. `Truncate(s string, max int) string`: rune-aware truncation to `max` display runes with a
   trailing `…`; must be applied *after* sanitization; never split a UTF-8 sequence.
4. Pure functions, no I/O, no globals. Deterministic.

## Acceptance Criteria
- A payload containing `\x1b[31m`, `\x1b]0;title\x07`, `\x07`, and stray control bytes emerges
  with all of them removed and the surrounding readable text intact.
- Legitimate UTF-8 (e.g. `日本語`, `café`, `🚀`, RTL text) passes through unchanged.
- `SanitizeInline` yields single-line output.
- `Truncate` never produces invalid UTF-8 and appends `…` only when it actually truncates.
- Invalid UTF-8 input yields U+FFFD, not raw bytes.

## Validation
- Table-driven tests with an explicit corpus of escape/control payloads and Unicode text,
  asserting exact expected outputs. Include a fuzz test (`testing.F`) asserting output contains no
  byte in the forbidden set and remains valid UTF-8.

## Dependencies
Issue 01.

## Non-goals
- Deciding when to sanitize (callers/issue 07 decide human vs JSON).
- Color handling (issue 07).

## Design References
- DESIGN §9.2; ADR-004; threat B3.
