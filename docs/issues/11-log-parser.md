# 11 — `git log` delimited parser

## Summary
Implement `internal/git/log.go`: run `git log` with a delimited pretty format and parse commits
into structured records.

## Context
DESIGN §6.5, §7.3 and research doc §3. Parsing the human `git log` layout is unsafe; use a
unit-separator pretty format.

## Scope
- `internal/git/log.go`.

## Detailed Requirements
1. `type LogEntry struct { SHA, Author, AuthorTime, Subject string }` (AuthorTime is strict
   ISO-8601 from `%aI`).
2. `Log(ctx, opts LogOpts) ([]LogEntry, error)` where
   `LogOpts{ RevRange string; MaxCount int; Paths []string }`.
   - argv: `log [<revrange>] [-n <count>] --pretty=format:%H%x1f%an%x1f%aI%x1f%s [-- <paths>...]`.
   - Split records by newline, fields by `\x1f` (unit separator). Exactly 4 fields per record;
     a record with the wrong field count is a defensive error/skip.
3. `Author`/`Subject` stored raw (untrusted; sanitized at render).
4. Empty history (`git log` on an empty repo) returns an empty slice, not an error where git
   itself would error — normalize the "does not have any commits yet" case to empty.

## Acceptance Criteria
- A repo with N commits yields N ordered `LogEntry` values with correct SHA/author/subject.
- Path filter restricts results.
- `-n` limits count.
- Empty repo → empty slice, no crash.

## Validation
- Integration tests in `t.TempDir()`: create commits, assert parsing, path filter, and `-n`.
- Unit test of the field-splitting on a synthetic delimited buffer, including a malformed record.

## Dependencies
Issue 02.

## Non-goals
- Note lookup / rendering (issue 17).

## Design References
- DESIGN §6.5, §7.3; research §3.
