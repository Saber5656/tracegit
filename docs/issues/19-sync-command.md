# 19 — `tracegit sync` command

## Summary
Move the notes ref between the local repo and a remote via explicit `setup`, `fetch`, and `push`
subcommands. Never runs implicitly.

## Context
DESIGN §6.7, §9 (B5); ADR-001; research §1.4–§1.5. Notes are not transferred by default git
fetch/push, and fetched notes are untrusted.

## Scope
- `internal/cli/sync.go`.

## Detailed Requirements
1. Subcommands (all take an optional `[<remote>]`, default `origin`):
   - `setup`: idempotently add fetch + push refspecs for the notes ref to `remote.<r>.*` config
     (`+<notesRef>:<notesRef>`). Do not create duplicate lines (check existing config first).
   - `fetch`: `git fetch <remote> <notesRef>:<notesRef>`.
   - `push`: `git push <remote> <notesRef>:<notesRef>`.
2. `fetch` conflict handling: expose `--strategy manual|ours|theirs|union` (default `manual`).
   On a notes-merge conflict with `manual`, abort and print guidance; `union` is allowed only with
   a printed caveat that it can produce non-JSON note bodies (research §1.5).
3. Missing remote → exit 2 naming the missing remote. No credential handling: auth is entirely
   git's (SSH/HTTPS helpers). tracegit never reads or logs credentials.
4. Never auto-invoke sync from any other command.

## Acceptance Criteria
- `sync setup` adds the refspecs once; re-running does not duplicate them.
- `push` then `fetch` against a local bare remote moves notes both ways.
- `fetch --strategy manual` on a conflicting note aborts with guidance (no silent overwrite).
- Missing remote exits 2.

## Validation
- Integration tests in `t.TempDir()` with a local bare remote (`git init --bare`): configure a
  remote, `setup` (assert config idempotency), commit+annotate, `push`, clone/fetch into a second
  repo, assert the note is present; construct a conflict and assert `manual` aborts.

## Dependencies
Issues 12, 02, 05.

## Non-goals
- Automatic sync / hooks (v2).
- Advanced conflict auto-resolution beyond the offered strategies (v2).

## Design References
- DESIGN §6.7, §9 (B5); ADR-001; research §1.4–§1.5.
