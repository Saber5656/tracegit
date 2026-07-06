# 16 — `tracegit blame` command

## Summary
The flagship command: reasoning-aware blame. Annotate each line/hunk of a file with the
originating commit's rationale summary (and optionally full rationale).

## Context
DESIGN §6.4, §7.2, §8, §9.2. Uses the porcelain parser (issue 10) + notes lookup (issue 08),
routing all untrusted text through sanitization (issue 06/07).

## Scope
- `internal/cli/blame.go`.

## Detailed Requirements
1. Positional `<file>`, optional `<rev>`. Flags: `-L <start>,<end>`, `--json`, `--rationale`,
   `-C`, `-M`.
2. Run `git.Blame(opts)` (issue 10). For each **unique** commit SHA, look up its note once
   (`notes.Show` → `ClampForRead` → `Unmarshal`); cache in a `map[sha]*record.Record` (nil when
   absent/degraded, with a degraded marker).
3. Human mode:
   - Group consecutive lines by commit. Print, per line, `shortsha  author  <summary>  | line`
     (widths aligned; summary via `SanitizeInline`+`Truncate`). Commits with no note show a muted
     `(no rationale)` marker.
   - With `--rationale`, print each commit's full rationale block once above its first hunk.
4. `--json`: array of `{ line, content, commit: {sha, author, author_time}, rationale: <record|null> }`
   in file order. Degraded records represented as `{ "error": "<reason>", "raw": "<clamped>" }`
   in place of `rationale`.
5. Errors: unblameable file (untracked/binary) → clear error, exit non-zero. Non-existent rev/path
   → propagate a clear git-derived error.
6. Performance: single blame invocation + one note lookup per unique commit (no per-line git
   calls).

## Acceptance Criteria
- `tracegit blame <file>` shows each line with its commit's summary; lines from a commit without a
  note show the no-rationale marker.
- `-L` restricts to the range; `--rationale` prints full blocks; `--json` shape matches spec and
  round-trips.
- An escape payload in a commit summary/rationale is sanitized in human output.
- Unblameable file exits non-zero with a clear message.
- No per-line git subprocess (assert lookup count by construction/caching).

## Validation
- Integration tests in `t.TempDir()`: build a file across ≥2 commits (one annotated via
  `tracegit commit`, one plain), assert blame output, JSON shape, `-L`, and the no-note marker;
  inject an escape-payload note and assert sanitization.

## Dependencies
Issues 12, 10, 08, 07, 04, 03.

## Non-goals
- History-wide listing (issue 17).

## Design References
- DESIGN §6.4, §7.2, §8, §9.2; research §2.
