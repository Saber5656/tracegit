# ADR-003: `tracegit commit` wrapper is the primary capture mechanism

- **Status:** Accepted
- **Date:** 2026-07-06
- **Deciders:** Human product owner + Fable (design)

## Context

Rationale must reliably reach the store keyed to the correct commit. Candidate capture paths:
(A) a `tracegit commit` wrapper that commits and writes the note atomically; (B) `tracegit
annotate` after the fact; (C) git hooks (`prepare-commit-msg`/`post-commit`) auto-capturing from
a convention; (D) ingesting a commit-message trailer written by the agent.

The human owner selected the **`tracegit commit` wrapper** as the primary path, and the v1 scope
("Standard") explicitly includes retroactive **`tracegit annotate`**.

## Decision

- **Primary:** `tracegit commit` performs `git commit` then writes the rationale note to the
  resulting `HEAD` (DESIGN §6.1). Validation runs *before* the commit; the note is written after
  the SHA is known.
- **Secondary (in v1):** `tracegit annotate <commit-ish>` adds/replaces/append rationale on any
  existing commit, enabling retroactive capture and correction (DESIGN §6.2).
- **Deferred (v2):** hook-based auto-capture (C) and trailer ingestion (D).

## Rationale

- The wrapper gives the strongest commit↔rationale association with an explicit, predictable
  workflow that agents can script.
- `annotate` covers the "commit already happened" and "fix a mistake" cases without a second
  capture convention.
- Hooks and trailer ingestion add magic/fragility and a second parallel path; deferring them lets
  v1 validate the notes model first.

## Consequences

- Atomicity is best-effort: the commit and the note are two git operations. If the note write
  fails, the commit stands and `tracegit` emits a `tracegit annotate HEAD ...` recovery hint and a
  non-zero exit (DESIGN §10). This is documented behavior, not a silent partial success.
- `--amend` moves HEAD; the note attaches to the amended SHA and any prior note is orphaned
  (documented).
- Agents are expected to set `TRACEGIT_AGENT`/`TRACEGIT_MODEL` once per session (DESIGN §5) so
  attribution is automatic.

## Open items

- Whether `commit` should offer an opt-in "note-first, then commit, rollback note on commit
  failure" transactional mode is left to v2 based on real failure rates.
