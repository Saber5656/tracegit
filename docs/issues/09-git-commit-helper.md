# 09 — Commit + rev-parse helper

## Summary
Implement `internal/git/commit.go`: run `git commit` with a caller-supplied argv and resolve the
resulting commit SHA, so `tracegit commit` can attach a note to the right commit.

## Context
DESIGN §6.1, §7.1, §10. The commit wrapper (ADR-003) commits first, then writes the note to
`HEAD`. `--amend` moves HEAD, so the SHA must be resolved *after* the commit.

## Scope
- `internal/git/commit.go`.

## Detailed Requirements
1. `Commit(ctx, args []string) (exitCode int, err error)`:
   - Runs `git commit <args>` passing the caller's argv verbatim (the CLI layer, issue 13,
     assembles/whitelists these). stdin/stdout/stderr are wired through so an interactive editor
     (`-e`) and hooks work — i.e. inherit the parent's std streams for this call.
   - Return git's exit code; on non-zero, the caller writes no note.
2. `ResolveHead(ctx) (sha string, err error)`: `git rev-parse HEAD` → 40-hex SHA (trimmed).
3. `ResolveCommitish(ctx, ish string) (sha string, err error)`:
   - `git rev-parse --verify <ish>^{commit}`; error clearly if it does not resolve to exactly one
     commit (used by annotate/show).
4. Because `git commit` may need a TTY/editor, provide a mode that inherits std streams (for real
   commit) and a capturing mode (for tests using `-m`).

## Acceptance Criteria
- A successful `-m` commit returns exit 0 and `ResolveHead` returns the new SHA.
- A failed commit (e.g. nothing staged, or `--no-verify` hook rejection simulated) returns a
  non-zero exit code and no panic.
- `ResolveCommitish` resolves `HEAD`, short SHAs, and tags to a full commit SHA; errors on a
  non-commit or ambiguous ref.

## Validation
- Integration tests in `t.TempDir()`: stage a file, `Commit(["-m","msg"])`, assert `ResolveHead`
  matches `git rev-parse HEAD`; test the empty-index failure path; test `ResolveCommitish` on
  HEAD and a bad ref.

## Dependencies
Issue 02.

## Non-goals
- Building/whitelisting commit args from flags (issue 13).
- Writing notes (issue 08).

## Design References
- DESIGN §6.1, §7.1, §10; ADR-003.
