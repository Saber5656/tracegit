# 14 — `tracegit annotate` command

## Summary
Attach, replace, or append a rationale note on an existing commit, enabling retroactive capture
and correction.

## Context
DESIGN §6.2, §10. Part of v1 "Standard" scope (ADR-003). Refuses to clobber existing notes unless
told to.

## Scope
- `internal/cli/annotate.go`.

## Detailed Requirements
1. Positional arg `<commit-ish>`, resolved via `git.ResolveCommitish` (issue 09) to a full SHA;
   ambiguous/non-commit → exit 2.
2. Reuse the rationale/metadata flags from issue 13 (`--summary`, `--rationale[-file|-stdin]`,
   `--agent`, `--model`, `--task`, `--task-url`, `--confidence`, `--tag`).
3. Modes:
   - Default: if a note already exists → refuse (exit 1) with a hint to use `--force`/`--append`.
   - `--force`: overwrite (`notes.Add(force=true)`).
   - `--append`: read the existing record, append the new `rationale` to the existing one with a
     separator (e.g. `\n\n---\n\n`), keep other fields unless overridden by flags, re-validate.
   - `--edit`: open `$EDITOR`→`$GIT_EDITOR`→`core.editor`→platform default with the
     current/derived record as canonical JSON; validate the edited result before writing; abort on
     empty/unchanged/invalid.
4. Build/merge the record, `record.ValidateForWrite`, then write via `notes.Add`.
5. `--edit` temp files must be created with `0600` perms and removed afterward (threat B7).

## Acceptance Criteria
- Annotating a note-less commit creates the note.
- Annotating an annotated commit without a mode flag refuses (exit 1); `--force` overwrites;
  `--append` concatenates rationale and re-validates size limits.
- `--edit` round-trips through a fake editor (test injects an editor command) and validates.
- Invalid `<commit-ish>` exits 2.
- Edited temp file is `0600` and cleaned up.

## Validation
- Command tests in `t.TempDir()`: create/force/append/edit paths (fake `EDITOR` that mutates the
  file), invalid ref, and over-limit-after-append rejection.

## Dependencies
Issues 12, 08, 04, 05, 03, 09.

## Non-goals
- Committing (issue 13).

## Design References
- DESIGN §6.2, §10, §9 (B7); ADR-003.
