# 12 — Root command + global flags + version

## Summary
Build the cobra root command, global flags, config wiring, git precheck, and the `version`
subcommand, giving all later commands a consistent foundation.

## Context
DESIGN §5, §6 (global flags), §6.9, §7.1. This replaces the stub `Execute()` from issue 01.

## Scope
- `internal/cli/root.go` and `internal/cli/version.go`. No feature subcommands yet.

## Detailed Requirements
1. `Execute() int`: construct the root command, run it, map errors to exit codes (0 ok, 2 usage
   error via cobra, 1 otherwise). Do not `os.Exit` inside library code; return the code to
   `main`.
2. Root persistent flags per DESIGN §6: `--notes-ref`, `--git-dir` (maps to git `-C`/`--git-dir`
   leading args), `--color=auto|always|never`, `-v/--verbose`, `--version`. Register `--json`
   locally on the commands that support it (not globally).
3. A `PersistentPreRunE` that:
   - Builds the `git.Runner` (issue 02) using resolved `GitBinary`.
   - Resolves `config.Config` (issue 05) from flags/env/git-config.
   - For commands that touch a repo, runs `EnsureMinVersion(2,20)` and `EnsureRepo`. `version`
     and help must work outside a repo (skip the repo check for them).
   - Stores the runner + resolved config in a context or a command-scoped struct passed to
     subcommands.
4. `version` subcommand: print tracegit version (from `internal/version`), detected git version,
   and the active notes ref; support `--json`.
5. Consistent error presentation: user-facing errors printed to stderr without Go stack traces.

## Acceptance Criteria
- `tracegit --help` and `tracegit version` work outside a git repo (no repo/precheck failure).
- Inside a repo, `PersistentPreRunE` provides a ready runner + config to subcommands.
- Unknown command / bad flag exits 2; runtime error exits 1; success exits 0.
- `--notes-ref`/`--color`/env are reflected in the resolved config used by a probe subcommand.
- `version --json` emits valid JSON with tracegit + git versions and notes ref.

## Validation
- Command tests invoking the root with args in `t.TempDir()` repos and outside a repo; assert
  exit codes and that config/runner are wired (via a hidden test-only probe command or by
  asserting `version` output).

## Dependencies
Issues 02, 05.

## Non-goals
- Feature subcommands (issues 13–20).

## Design References
- DESIGN §5, §6 (globals), §6.9, §7.1, §10.
