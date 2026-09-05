# ADR-001: Store rationale in git notes (`refs/notes/tracegit`)

- **Status:** Accepted
- **Date:** 2026-07-06
- **Deciders:** Human product owner + Fable (design)

## Context

`tracegit` must associate a rationale record with a specific commit and retrieve it later,
without rewriting history. Candidate stores were: (A) git notes, (B) commit-message trailers,
(C) an in-repo sidecar directory (e.g. `.tracegit/` JSONL), (D) an external database/service.

Requirements: keyed by commit SHA; supports arbitrarily long, structured content; does not alter
commit SHAs; git-native (no service); works offline.

## Decision

Use **git notes** on the dedicated ref `refs/notes/tracegit` as the canonical store. One note per
commit; the note body is the canonical `tracegit/v1` JSON record (DESIGN §4).

## Alternatives considered

| Option | Pros | Cons | Verdict |
|---|---|---|---|
| **A. Git notes** | Git-native; keyed by SHA; no history rewrite; arbitrary length; editable post-hoc | Not fetched/pushed by default; concurrent-write merges need care | **Chosen** |
| B. Commit trailers | Travels with commit automatically | Immutable (part of SHA); can't annotate old commits; pollutes messages; ugly for long text | Rejected as canonical (kept as v2 ingestion source) |
| C. Sidecar `.tracegit/` | Full control; diffable | Decouples from commits; drifts; adds tracked files to every project; merge conflicts | Rejected |
| D. External DB/service | Rich queries | Not git-native; needs infra; offline-hostile; against project principles | Rejected |

## Consequences

- **Sync is explicit.** Notes are not fetched/pushed by default, so v1 ships `tracegit sync
  setup/fetch/push` to configure refspecs (`+refs/notes/tracegit:refs/notes/tracegit`) and move
  the ref. No implicit network (also a security property).
- **Concurrency.** Two clones annotating the same commit produce divergent notes refs. v1 uses
  git's notes merge with a **`manual`** default strategy (abort + instruct) and offers
  `ours/theirs/union` opt-in. Auto-merge beyond this is deferred (DESIGN §13).
- **No history rewrite**, satisfying the non-destructive principle.
- **Untrusted-content boundary.** Fetched notes are untrusted input and re-enter the read-path
  sanitization/limits (see [ADR-004](./ADR-004-terminal-output-sanitization.md), DESIGN §9).
- The notes ref is configurable (`tracegit.notesRef`) for users who need isolation.

## Open items

- Note authenticity is not guaranteed (anyone who can push the ref can write notes). Signed notes
  are deferred to v2; documented in `SECURITY.md`.
