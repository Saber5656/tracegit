# ADR-002: Implement in Go and shell out to the `git` binary

- **Status:** Accepted
- **Date:** 2026-07-06
- **Deciders:** Human product owner + Fable (design)

## Context

`tracegit` is a distributable developer CLI that orchestrates git (commit, notes, blame, log) and
stores/retrieves small structured records. It targets OSS release across macOS, Linux, and
Windows. Two orthogonal choices: the implementation language, and how the tool talks to git
(shell out to the `git` binary vs. embed a git library such as `go-git` or libgit2 bindings).

## Decision

1. Implement v1 in **Go** (module `github.com/Saber5656/tracegit`), distributed as a single
   static binary via cross-compilation.
2. **Shell out to the user's `git` binary** through one audited exec wrapper (`internal/git`),
   using an explicit argument vector — never a shell.

## Rationale

- **Distribution:** Go produces a single dependency-free binary and cross-compiles trivially —
  ideal for a CLI installed into many environments and CI.
- **Contributor ramp:** Go is widely known and simple to read, aiding an OSS contributor base.
- **Behavioral fidelity:** Using the user's real `git` guarantees blame/notes/log semantics,
  config, hooks, credentials, and version behavior match what the user already has, instead of a
  reimplementation drifting from git.
- **Small surface:** Shelling out keeps the dependency graph minimal (cobra + stdlib) and
  concentrates the security-sensitive exec logic in one package.
- **Security:** Argv-only exec eliminates shell-injection risk (DESIGN §9.1, B1).

## Alternatives considered

| Option | Verdict |
|---|---|
| Rust + libgit2 | Rejected for v1: heavier contributor ramp; in-process git not needed given git-CLI fidelity goal. Reconsider only if profiling shows exec overhead dominates. |
| Go + `go-git` (pure-Go) | Rejected: reimplements git semantics (notes/blame coverage incomplete/divergent), larger dep, must track git compatibility. |
| TypeScript/Node | Rejected: runtime/bundling burden for a system CLI. |
| Python | Rejected: packaging a distributable CLI is awkward; slow startup. |

## Consequences

- Hard runtime dependency on `git` (>= 2.20), version-gated at startup with a clear error.
- Exec overhead per git call is acceptable for an interactive/CI tool; note lookups are cached
  per invocation to avoid N× `git notes show`.
- `TRACEGIT_GIT_BINARY` / `tracegit.gitBinary` lets users pin the git executable (mitigates PATH
  hijack, DESIGN §9.1 B6).
- License: MIT proposed (open item — confirm before public release).

## Open items

- Confirm MIT vs Apache-2.0 before the repo is made public.
