# 18 — `tracegit export` command

## Summary
Bulk-export all rationale records (optionally within a rev range) as JSON or JSONL for backup,
analysis, or migration.

## Context
DESIGN §6.6. In v1 this also substitutes for a dedicated search command (deferred): users pipe
export output to external tools.

## Scope
- `internal/cli/export.go`.

## Detailed Requirements
1. Flags: `--format json|jsonl` (default `json`), `--output <path>` (default stdout),
   optional `<revrange>`.
2. Enumerate annotated commits via `notes.List` (issue 08). If `<revrange>` is given, intersect
   with `git rev-list <revrange>` so only in-range commits are exported.
3. For each commit: read note → `ClampForRead` → `Unmarshal`.
   - Valid: emit `{ "commit": "<sha>", "record": {...} }`.
   - Degraded: emit `{ "commit": "<sha>", "raw": "<clamped body>", "error": "<reason>" }`.
4. `json`: a single top-level array. `jsonl`: one object per line. Always valid, correctly-escaped
   JSON. **No ANSI sanitization** (machine output).
5. `--output` writes to the given path (mode `0644`), created/truncated; default stdout. Do not
   create directories; error clearly if the parent dir is missing.
6. Deterministic ordering: sort by commit topological/`rev-list` order when a range is given, else
   by SHA, so output is stable/testable.

## Acceptance Criteria
- Exports all records as a valid JSON array; `jsonl` emits one object per line.
- Rev-range intersection limits output to in-range commits.
- Degraded notes are emitted with `raw`+`error`, not skipped, and output stays valid JSON.
- `--output` writes the file; missing parent dir errors clearly.
- Output ordering is deterministic across runs.

## Validation
- Integration tests in `t.TempDir()`: several annotated commits + one raw/malformed note; assert
  JSON array validity, JSONL line count, rev-range filtering, degraded emission, and `--output`
  file contents.

## Dependencies
Issues 12, 08, 03, 04.

## Non-goals
- Full-text search (v2).
- Importing records (v2).

## Design References
- DESIGN §6.6, §9.3.
