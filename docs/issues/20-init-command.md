# 20 — `tracegit init` command

## Summary
Prepare a repository for tracegit: persist chosen defaults into local git config and (unless
disabled) configure notes sync refspecs. Idempotent.

## Context
DESIGN §6.8. Lowers first-use friction; wraps `sync setup` (issue 19).

## Scope
- `internal/cli/init.go`.

## Detailed Requirements
1. Flags: `--agent <name>`, `--remote <r>` (default `origin`), `--no-sync-setup`.
2. Write provided defaults to `git config --local`: `tracegit.agent` (if `--agent` given), and
   ensure `tracegit.notesRef` is recorded when a non-default `--notes-ref` was passed globally.
3. Unless `--no-sync-setup`, invoke the same logic as `sync setup <remote>` (issue 19),
   idempotently. If the remote does not exist, warn (not fatal) and skip sync setup, telling the
   user to run `tracegit sync setup` after adding a remote.
4. Re-running `init` must not duplicate config or fail.
5. Print a short summary of what was configured.

## Acceptance Criteria
- `tracegit init --agent claude-code` writes `tracegit.agent=claude-code` to local config.
- With a configured remote, sync refspecs are added once (idempotent on re-run).
- With no remote, init warns and still succeeds (exit 0), skipping sync setup.
- `--no-sync-setup` skips refspec configuration.

## Validation
- Integration tests in `t.TempDir()`: init with/without a remote, assert config keys and refspec
  presence, re-run idempotency, `--no-sync-setup` behavior.

## Dependencies
Issues 12, 05, 19.

## Non-goals
- Global (`--global`) configuration (v2; v1 is repo-local).

## Design References
- DESIGN §6.8.
