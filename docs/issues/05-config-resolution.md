# 05 — Config resolution (flags/env/git-config)

## Summary
Implement layered configuration resolution with precedence flags > env vars > `git config
tracegit.*` > defaults, producing a single resolved `Config` used by all commands.

## Context
DESIGN §5 defines the keys and precedence. Env defaults let agents set `TRACEGIT_AGENT`/`MODEL`
once per session.

## Scope
- `internal/config/config.go`.

## Detailed Requirements
1. `type Config struct` with resolved fields: `NotesRef, Agent, Model, Task, TaskURL,
   MaxRationaleBytes, GitBinary, Color` (Color enum: `auto|always|never`).
2. `type Sources struct` capturing raw inputs from each layer: parsed flags (pointers /
   "was-set" markers), an env getter, and a git-config getter.
3. `Resolve(sources) (Config, error)` applies precedence per key exactly as DESIGN §5.2:
   - Flag value if the flag was explicitly set.
   - Else env var if present and non-empty.
   - Else `git config` value (`--local` naturally overrides `--global` since git resolves that).
   - Else the documented default.
4. Reading git config uses the runner (issue 02): `git config --get tracegit.<key>` (absent key →
   not-set, not an error). Never fail hard on a missing key.
5. Defaults exactly: `NotesRef=refs/notes/tracegit`, `MaxRationaleBytes=65536`, `GitBinary=git`,
   `Color=auto`; others empty.
6. Validate resolved values: `Color` in the enum; `MaxRationaleBytes` a positive integer;
   `NotesRef` a non-empty, syntactically valid ref (starts with `refs/` or a bare short name that
   git accepts).
7. `Color=auto` resolution to a boolean happens at render time (issue 07), not here; store the
   enum.

## Acceptance Criteria
- Precedence honored per key: a flag beats env beats git-config beats default (table test with a
  fake env getter and fake git-config getter).
- Missing git-config key does not error.
- Invalid `Color` or non-positive `MaxRationaleBytes` produces a clear error.
- `NotesRef` override flows through.

## Validation
- Unit tests with injected env/git-config getters covering each precedence combination and the
  validation failures.

## Dependencies
Issues 01, 02.

## Non-goals
- Wiring cobra flags (issue 12 provides the flag layer and calls `Resolve`).
- Color TTY detection (issue 07).

## Design References
- DESIGN §5.1, §5.2.
