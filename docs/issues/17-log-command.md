# 17 — `tracegit log` command

## Summary
Rationale-aware log: list commits with their rationale summary (and optionally full rationale).

## Context
DESIGN §6.5, §7.3, §8. Uses the log parser (issue 11) + notes lookup (issue 08).

## Scope
- `internal/cli/log.go`.

## Detailed Requirements
1. Optional `<revrange>`; flags: `-n/--max-count`, `--json`, `--rationale`, and trailing
   `-- <path>...` path filter.
2. `git.Log(opts)` (issue 11) → for each commit, note lookup (cached) with the same
   read→clamp→decode→degrade pipeline as `show`.
3. Human mode: per commit, header (short SHA, author, date, subject sanitized) + summary; with
   `--rationale`, the full rationale block. Commits without a note show `(no rationale)`.
4. `--json`: array of `{ commit: {sha, author, author_time, subject}, rationale: <record|null|degraded> }`.
5. Empty history → empty output (human) / `[]` (json), exit 0.

## Acceptance Criteria
- Lists N commits with summaries; note-less commits marked; annotated commits show their summary.
- `-n` limits; path filter restricts; `<revrange>` honored.
- `--json` shape matches spec.
- Escape payload sanitized in human output.
- Empty repo → `[]` / no output, exit 0.

## Validation
- Integration tests in `t.TempDir()`: multiple commits (mixed annotated/plain), assert human +
  JSON, `-n`, path filter, empty-repo case, sanitization.

## Dependencies
Issues 12, 11, 08, 07, 04, 03.

## Non-goals
- Per-line attribution (issue 16).

## Design References
- DESIGN §6.5, §7.3, §8, §9.2; research §3.
