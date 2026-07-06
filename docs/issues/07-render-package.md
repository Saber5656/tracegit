# 07 — Render package (human + JSON helpers)

## Summary
Provide shared rendering helpers used by all read commands: TTY/color detection, human formatting
primitives (that route untrusted text through `Sanitize`), and stable JSON output structs.

## Context
DESIGN §8 and §9.2. Centralizing rendering guarantees no command prints raw untrusted bytes and
that `--json` shapes are consistent and stable.

## Scope
- `internal/render/human.go` and `internal/render/json.go`. (Sanitizer is issue 06.)

## Detailed Requirements
1. `ColorEnabled(mode config.Color, out *os.File) bool`: `always`→true, `never`→false,
   `auto`→true iff `out` is a terminal (use `golang.org/x/term`? Prefer stdlib
   `os.File`+`term.IsTerminal` — but avoid new deps: implement with `(fileInfo.Mode()&os.ModeCharDevice)!=0`)
   and `NO_COLOR` env is unset.
2. Human helpers (each MUST sanitize untrusted args via issue 06 before writing):
   - `Summary(w, sha, author, when, summary string, color bool)` — one-line blame/log entry.
   - `RationaleBlock(w, rationale string, color bool)` — multi-line block (uses `Sanitize`, not
     `SanitizeInline`).
   - `Warning(w, msg string)` / `Notice(w, msg string)` — for degraded/no-note cases.
   - A short-SHA helper (first 8 hex).
3. JSON output structs mirroring DESIGN §6 shapes, e.g. `type CommitJSON struct{SHA, Author,
   AuthorTime string}`, `type BlameLineJSON`, `type LogEntryJSON`, `type ShowJSON`. Provide
   `WriteJSON(w io.Writer, v any) error` that emits indented JSON + trailing newline and never
   sanitizes (machine output) but always valid JSON.
4. Rendering functions take an `io.Writer` (testable); no direct `os.Stdout` use inside helpers.
5. Author/summary/rationale coming from git or notes are treated as untrusted; SHA/timestamps
   produced by git are trusted structurally but still length-checked.

## Acceptance Criteria
- Any untrusted string passed to a human helper is sanitized (test: escape payload in `summary`
  does not appear in output).
- `ColorEnabled` returns false when writing to a non-terminal (pipe) and when `NO_COLOR` is set.
- `WriteJSON` output parses back to an equal value and is stable/indented with a trailing newline.
- Helpers write to the provided `io.Writer`.

## Validation
- Unit tests writing to `bytes.Buffer`; assert sanitization applied and JSON round-trips; color
  detection tests using pipes vs a faked char-device.

## Dependencies
Issues 03, 06.

## Non-goals
- Command wiring (issues 15–18).
- Fetching data from git (issues 08–11).

## Design References
- DESIGN §8, §9.2.
