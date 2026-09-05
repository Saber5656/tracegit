# 02 — git exec runner (argv, discovery, version gate)

## Summary
Implement `internal/git.Runner`: the single audited place that executes the `git` binary. Uses an
explicit argument vector (never a shell), discovers/validates git, and scopes to the repository.

## Context
DESIGN §7.1 and ADR-002 mandate shelling out to git via argv only. This eliminates shell-injection
(DESIGN §9.1 B1) and PATH-hijack risk (B6). Every other package reaches git through this one.

## Scope
- `internal/git/runner.go` only. No notes/commit/blame logic (later issues use this runner).

## Detailed Requirements
1. `type Runner struct` holding the resolved git binary path and optional global leading args
   (e.g. `-C <dir>` / `--git-dir`).
2. Constructor `New(opts Options) (*Runner, error)`:
   - Resolve binary: `opts.Binary` if set, else `TRACEGIT_GIT_BINARY`, else `tracegit.gitBinary`
     (leave git-config lookup to issue 05 integration; for now accept an injected value), else
     `exec.LookPath("git")`. If unresolved → error "git executable not found on PATH".
   - Do **not** interpret the binary path through a shell.
3. `Run(ctx, args ...string) (stdout, stderr []byte, err error)` and a
   `RunStdin(ctx, stdin io.Reader, args ...string)` variant:
   - Build `exec.CommandContext(ctx, r.bin, append(r.globalArgs, args...)...)`. Never use
     `sh -c` or string concatenation of a command line.
   - Capture stdout/stderr separately. Return a typed error that exposes the exit code and
     stderr tail.
   - Set a deterministic environment when needed by callers (e.g. do not leak locale-dependent
     output for parsers — allow callers to pass `GIT_*` overrides).
4. `Version(ctx) (major, minor int, raw string, err error)`: parse `git --version`
   (`git version X.Y.Z`). `EnsureMinVersion(2, 20)` errors if lower.
5. `EnsureRepo(ctx) error`: `git rev-parse --is-inside-work-tree` must print `true`.
6. Provide `type ExitError` (or reuse `*exec.ExitError` wrapped) exposing `.Code() int` and
   `.Stderr() string` so callers (e.g. notes "no note" detection) can branch on exit status.

## Acceptance Criteria
- No code path builds a shell command string; all execs pass argv slices. (Grep for `sh -c`
  yields nothing.)
- `EnsureMinVersion` rejects `< 2.20` and accepts `>= 2.20` (table test with faked version
  strings via an injectable version parser).
- `RunStdin` feeds stdin to the child process.
- Errors from a failing git invocation expose exit code and stderr.

## Validation
- Unit tests: version parsing (table-driven incl. `git version 2.39.3 (Apple Git-145)` style),
  min-version gate, argv construction (assert the built args slice), stdin plumbing using a
  harmless command (`git --version` / `git hash-object --stdin`).
- Integration test (needs git): `EnsureRepo` true inside `t.TempDir()` after `git init`, false
  outside.

## Dependencies
Issue 01.

## Non-goals
- Notes/commit/blame/log specifics (issues 08–11).
- git-config reading (issue 05) — accept injected config values for now.

## Design References
- DESIGN §7.1, §9.1 (B1, B6); ADR-002; research doc §4.
