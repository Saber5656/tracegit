# 13 — `tracegit commit` command

## Summary
Implement the primary capture path: wrap `git commit`, then attach a validated rationale note to
the resulting commit.

## Context
DESIGN §6.1, §10; ADR-003. Validation runs before committing; the note is written after the SHA
is known; a note-write failure leaves the commit intact with a recovery hint.

## Scope
- `internal/cli/commit.go`.

## Detailed Requirements
1. Flags: `-m/--message` (repeatable), `--summary`, `--rationale`, `--rationale-file`,
   `--rationale-stdin`, `--agent`, `--model`, `--task`, `--task-url`,
   `--confidence`, `--tag` (repeatable). Rationale sources are mutually exclusive; error if more
   than one is set.
2. Pass-through git commit flags (whitelist, per DESIGN §6.1): `-a/--all`, `--amend`, `--no-edit`,
   `-S/--gpg-sign`, `--author`, `--no-verify`, `--allow-empty`, `--date`, `-p/--patch`,
   `-F/--file`, `--reset-author`, `-C/--reuse-message`, `-e/--edit`. Assemble the git argv from
   these — never from a raw string. Unknown/unlisted git flags are rejected with a clear message
   (documented; broad pass-through is a v2 consideration).
3. Build the record: `summary` defaults to the first line of the assembled commit message when
   `--summary` is omitted; pull `agent/model/task/taskURL` from flags→config (issue 05).
4. `record.ValidateForWrite` with the configured `MaxRationaleBytes` **before** committing. On
   failure: exit non-zero, commit nothing.
5. Sequence (DESIGN §6.1): validate → `git.Commit(args)` → on non-zero, propagate exit code, no
   note → `ResolveHead` → `notes.Add(sha, canonicalJSON, force=false)`.
6. On note-write failure after a successful commit: print
   `commit <shortsha> created but rationale note failed: <err>. Recover with: tracegit annotate <sha> --rationale-file <...>`
   and exit 1.
7. Respect `--amend`: resolve HEAD *after* the commit so the note attaches to the amended SHA
   (document the orphaned prior note).

## Acceptance Criteria
- `tracegit commit -m "msg" --rationale "why"` creates a commit and a note; `git notes --ref=tracegit show HEAD`
  returns the canonical record with `summary="msg"` and `rationale="why"`.
- Over-limit rationale is rejected and **no commit** is made.
- A failing `git commit` (nothing staged) exits non-zero and writes no note.
- Rationale via `--rationale-file` and `--rationale-stdin` works; specifying two sources errors.
- Unlisted git flag is rejected.

## Validation
- Command tests in `t.TempDir()`: happy path, over-limit rejection (assert no new commit),
  git-failure path, each rationale source, mutual-exclusion error, `--amend` note attachment.

## Dependencies
Issues 12, 09, 08, 04, 05, 03.

## Non-goals
- Retroactive annotation (issue 14).
- Arbitrary git-flag pass-through (v2).

## Design References
- DESIGN §6.1, §10; ADR-003; threat B1/B2.
