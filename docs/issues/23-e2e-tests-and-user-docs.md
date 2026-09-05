# 23 — End-to-end tests + user docs

## Summary
Add a full-lifecycle end-to-end test and write user-facing documentation (README + usage), closing
out v1.

## Context
DESIGN §11, §12. Validates the whole product working together and makes it usable by newcomers and
agents.

## Scope
- An e2e test package; expanded `README.md` and a `docs/USAGE.md` (or `docs/` usage section).
  No new product features.

## Detailed Requirements
1. **E2E test** in `t.TempDir()` against a local bare remote, exercising the documented lifecycle:
   `init → (stage) → commit (with rationale) → show → blame → log → export → sync push →
   clone → sync fetch → show/blame in the clone`. Assert the rationale survives the round-trip and
   all outputs match expectations (human + `--json`).
2. Deterministic env (`GIT_AUTHOR_*`, `GIT_COMMITTER_*`, fixed notes ref, `NO_COLOR`).
3. **README.md**: what tracegit is, install (binary + `go install`), quickstart (agent sets
   `TRACEGIT_AGENT`; `tracegit commit`; `tracegit blame`), the git-notes-not-synced-by-default
   caveat + `tracegit sync`, and a prominent "do not put secrets in rationale" warning
   (DESIGN §9.4).
4. **USAGE**: per-command reference mirroring DESIGN §6 (flags, examples, exit codes), and the
   record schema (DESIGN §4.1) for integrators.
5. Docs must state the security posture briefly and link `SECURITY.md`.

## Acceptance Criteria
- The e2e test passes locally and in CI, covering the full lifecycle including sync round-trip.
- README covers install, quickstart, sync caveat, and the secrets warning.
- USAGE documents every v1 command with at least one example each and the record schema.
- Docs and DESIGN do not contradict (schema, defaults, flags match).

## Validation
- `go test ./... -run E2E -race` green; manual read-through of README/USAGE against DESIGN §4/§6.

## Dependencies
Issues 13–20 (all commands).

## Non-goals
- Marketing site / screencasts (v2).

## Design References
- DESIGN §4.1, §6, §9.4, §11, §12.
