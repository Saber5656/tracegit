# 08 — Git notes read/write/list wrapper

## Summary
Implement `internal/git/notes.go`: add/show/list rationale notes on a configurable notes ref,
built on the runner (issue 02), with correct "no note" detection and untrusted-read handling.

## Context
DESIGN §4.3, §7.4 and research doc §1. Git notes are the canonical store (ADR-001). Read content
is untrusted (threat B3/B5).

## Scope
- `internal/git/notes.go`.

## Detailed Requirements
1. `type Notes struct { r *Runner; ref string }`; constructor takes the resolved notes ref
   (default `refs/notes/tracegit`).
2. `Add(ctx, sha string, body []byte, force bool) error`:
   - `git notes --ref=<ref> add [-f] -F - <sha>` with `body` on stdin (via `RunStdin`).
   - Without `force`, a pre-existing note causes git to fail; surface a typed
     `ErrNoteExists` so callers (annotate) can react.
3. `Show(ctx, sha string) (body []byte, present bool, err error)`:
   - `git notes --ref=<ref> show <sha>`.
   - Distinguish "no note" (→ `present=false, err=nil`) from real errors by inspecting exit
     code + stderr per research doc §1.3; when ambiguous, confirm via `List(sha)`.
4. `List(ctx) ([]Entry, error)` where `Entry{NoteBlob, Commit string}` parsed from
   `git notes --ref=<ref> list` lines.
5. `Remove(ctx, sha string) error` (used by tests) → `git notes --ref=<ref> remove <sha>`.
6. All reads return raw bytes; callers apply `ClampForRead` (issue 04) and sanitization (issue 06).
   This package does not parse the JSON.
7. Never build shell strings; always argv (inherited from runner).

## Acceptance Criteria
- Add then Show returns the exact bytes written.
- Show on a commit with no note returns `present=false, err=nil` (no spurious error).
- Add without force on an existing note returns `ErrNoteExists`; with force it overwrites.
- List returns all annotated commits.
- Large bodies (e.g. 200 KiB) pass through stdin without arg-length errors.

## Validation
- Integration tests in `t.TempDir()` with real git: init repo, commit, add/show/list/remove,
  and the no-note and exists cases. Set fixed `GIT_AUTHOR_*`/`GIT_COMMITTER_*`.

## Dependencies
Issue 02.

## Non-goals
- Record (de)serialization (issue 03) — callers pass/receive bytes.
- Sync/fetch/push (issue 19).

## Design References
- DESIGN §4.3, §7.4, §9 (B3/B5); ADR-001; research §1.
