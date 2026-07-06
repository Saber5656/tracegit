# ADR-004: Mandatory terminal-output sanitization for untrusted note content

- **Status:** Accepted
- **Date:** 2026-07-06
- **Deciders:** Human product owner + Fable (design)

## Context

Rationale records and git-provided fields (author name, commit summary) are attacker-influenceable:
a malicious contributor can craft a commit or push a note to `refs/notes/tracegit`. `tracegit
blame/show/log` render this content to a terminal. Raw untrusted text printed to a TTY can carry
ANSI/OSC/CSI/DCS escape sequences that spoof output, rewrite the terminal title, emit hyperlinks,
or (on some terminals) touch the clipboard — a well-known class of "terminal escape injection"
vulnerabilities in blame/log-style tools.

## Decision

Human-mode output MUST pass all note-derived and git-metadata-derived strings through a single
`render.Sanitize` function before printing. Sanitization rules (DESIGN §9.2):

- Strip C0 control chars except `\t` and `\n`.
- Strip the ESC byte (`0x1b`), BEL (`0x07`), C1 controls, and any CSI/OSC/DCS escape sequences.
- Preserve normal printable UTF-8 (all scripts, emoji).
- Bound displayed width for blame/log summaries (truncate with ellipsis).

`--json` (machine) output is NOT ANSI-sanitized but MUST always be valid, correctly-escaped JSON.

## Rationale

- The read commands are the product's main surface and the point where untrusted content meets a
  terminal — this is the highest-value security control.
- Centralizing in one function makes the control auditable and testable (issue 21) and prevents
  any command from accidentally printing raw untrusted bytes.

## Consequences

- All human-rendering code paths depend on `render.Sanitize`; the `render` package is a Wave-0/1
  foundation issue that display commands build on.
- Table-driven security tests assert escape sequences are neutralized and legitimate UTF-8
  survives.
- A minor UX cost: rationale containing intentional escape codes (rare) will display them
  stripped in human mode; the full raw bytes remain retrievable via `--json`/`export`.

## Open items

- Whether to offer an explicit `--raw` escape hatch is deferred; if added it must require an
  explicit opt-in flag and print a warning, never be the default.
