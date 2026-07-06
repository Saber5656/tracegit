# 10 — `blame --porcelain` parser

## Summary
Implement `internal/git/blame.go`: run `git blame --porcelain` and parse it into ordered line
records with per-commit metadata, defensively.

## Context
DESIGN §6.4, §7.2 and research doc §2 specify the porcelain format and the parser contract.
Malformed output must be flagged/skipped, never panic (threat B4).

## Scope
- `internal/git/blame.go`.

## Detailed Requirements
1. `type CommitMeta struct { SHA, Author, AuthorMail string; AuthorTime int64; AuthorTZ, Summary string; Boundary bool }`
   and `type BlameLine struct { FinalLine int; SHA string; Content string }`.
2. `Blame(ctx, opts BlameOpts) (lines []BlameLine, commits map[string]CommitMeta, err error)`
   where `BlameOpts{ File string; Rev string; LStart, LEnd int; DetectMoved (-M), DetectCopied (-C) bool }`.
   - Build argv: `blame --porcelain [<rev>] [-L s,e] [-M] [-C] -- <file>`.
   - Force stable output/locale as needed (avoid locale-affected fields).
3. Parser rules per research §2.2–§2.3:
   - Header line `^[0-9a-f]{40} <orig> <final>( <count>)?$` starts a line group.
   - On first sighting of a SHA, consume the extended header block (`author`, `author-mail`,
     `author-time`, `author-tz`, `summary`, `previous`, `filename`, `boundary`, `committer*`)
     until the TAB-prefixed content line.
   - Populate `commits[sha]` once; reuse afterward.
   - Emit one `BlameLine` per content line in file order.
   - `summary`/`author` are stored raw (untrusted; sanitized at render time).
4. Defensive: an unexpected/malformed line is skipped with an accumulated non-fatal warning
   returned alongside results (or logged via `-v`); never panic. A completely unparseable stream
   returns an error.
5. Handle `boundary` and missing optional keys gracefully.

## Acceptance Criteria
- A known porcelain fixture parses to the expected ordered lines and commit metadata.
- Repeated SHAs do not re-require the metadata block.
- Boundary commits parse without error.
- A truncated/garbled fixture yields a defensive error or skipped-line warning, not a panic.

## Validation
- Fixture-driven unit tests using captured porcelain output (checked-in fixtures), including a
  multi-commit file, a boundary/root commit, and a malformed fixture.
- Integration smoke test generating real blame in `t.TempDir()`.

## Dependencies
Issue 02.

## Non-goals
- Note lookup / rendering (issue 16).

## Design References
- DESIGN §6.4, §7.2, §9 (B4); research §2.
