# 15 — `tracegit show` command

## Summary
Show the rationale record for a single commit, in human or JSON form, degrading gracefully when
there is no note or the note is malformed.

## Context
DESIGN §6.3, §10. First read command; establishes the read → clamp → decode → degrade pattern
reused by blame/log/export.

## Scope
- `internal/cli/show.go`.

## Detailed Requirements
1. Positional `<commit-ish>` → `ResolveCommitish` (issue 09). Flags: `--json`, `--rationale`.
2. Read: `notes.Show(sha)` (issue 08) → `ClampForRead` (issue 04) → `record.Unmarshal` (issue 03).
3. Human mode:
   - Header: short SHA, author, date, subject (subject from `git.Log`/`show`; sanitized).
   - Print `summary` (sanitized). With `--rationale`, print the full rationale block (sanitized).
   - No note → `render.Notice("no rationale recorded")`, exit 0.
   - Unknown-schema / invalid JSON → `render.Warning(...)` + raw (clamped, sanitized) body, exit 0.
4. `--json`:
   - Note present + valid → `{ "commit": "<sha>", "present": true, "record": {...} }`.
   - No note → `{ "commit": "<sha>", "present": false }`.
   - Degraded → `{ "commit": "<sha>", "present": true, "raw": "<clamped body>", "error": "<reason>" }`.
   - JSON is not ANSI-sanitized but always valid.
5. Exit 0 for all read outcomes above (absence/degraded are not errors); exit 2 only for a bad
   commit-ish.

## Acceptance Criteria
- Shows a valid record (human + `--json`).
- No-note commit prints the notice / `present:false` JSON, exit 0.
- A note with invalid JSON or unknown schema degrades (warning + raw / `error` JSON), exit 0.
- Escape payload in the record does not appear raw in human output.
- Bad commit-ish exits 2.

## Validation
- Command tests in `t.TempDir()`: valid, absent, malformed-JSON, unknown-schema, and escape-payload
  notes (write raw bytes directly via `git notes add` for the malformed cases).

## Dependencies
Issues 12, 08, 07, 09, 03, 04.

## Non-goals
- Multi-commit listing (issue 17).

## Design References
- DESIGN §6.3, §8, §9.2, §10.
