# Wave 8 concrete review-resolution correction v2

- Repository: `Saber5656/tracegit`
- Pull request: #1
- Current PR head pinned for this review: `d862ae9f17c6cb284d21faca0fd38f81a7d23eb2`
- Current PR base pinned for this review: `9eacb8b1984731cf61b7837188b86f23cf54818e`
- Previous correction artifact blob SHA: `5ffa38306872f5e17ecd838de6b861efd51fb931`
- This v2 artifact supersedes the earlier generic resolution addenda for the exact threads below.
- The current head/base identity above is part of the review input; any later change invalidates this evidence and requires a fresh review.
- This is documentation-level handling for documentation-only PRs. It is not a claim that product implementation, runtime tests, build, CI, security, or release validation is complete.
- No PR review bot is re-triggered.

## Per-repository blocking handoffs

| Repository | QA/full-validation handoff | Security/privacy handoff |
|---|---|---|
| `Saber5656/earmark#1` | `tech-qa`/`tech-tester` must execute API/schema, codec, TTS, lifecycle, launchd, queue, notification, docs-lint, and repository-full-validation gates for this head/base. | `tech-security`/`tech-devopssec` must accept credential URL, remote egress, temp-file, output, path, static-audit, and packaging boundaries. |
| `Saber5656/tracegit#1` | `tech-qa`/`tech-tester` must execute record/schema, parser, CLI, editor, note, sync, E2E, docs-lint, and repository-full-validation gates for this head/base. | `tech-security`/`tech-devopssec` must accept note-size, file-mode, ref-update, sanitization, output-bound, and Git execution boundaries. |
| `Saber5656/mcplay#1` | `tech-qa`/`tech-tester` must execute schema, recorder, replay, dependency, CLI, protocol, docs-lint, and repository-full-validation gates for this head/base. | `tech-security`/`tech-devopssec` must accept spawn, path, permission, redaction, embedded-command, export, persistence, and threat-model boundaries. |

These handoffs are blocking: missing, pending, failed, skipped, cancelled, timed-out, stale, or non-accepting specialist/full-validation evidence prevents thread resolution and merge.

## Thread contracts

### 1. Thread `PRRT_kwDOTN39T86Odfx8` — Don't hard-code the license as MIT yet

- File: `docs/DESIGN.md`
- Line: 128
- Finding basis: LICENSE MIT reads like a resolved decision, but ADR-002 still leaves the license as an open item to confirm before public release. Keep this placeholder neutral until that decision is closed.

**Normative resolution**: At `docs/DESIGN.md:128`, the canonical contract SHALL adopt this exact decision: The license SHALL remain an owner-gated unresolved decision until explicitly approved; release metadata and artifacts SHALL not label it MIT while unresolved, and the release gate SHALL block.

**Focused verification gate**: Run release preflight with unresolved and owner-resolved license states; assert unresolved state blocks and no artifact labels it MIT.

**Completion boundary**: This contract is design-level only. Resolve the GitHub thread only after the focused gate, applicable specialist handoff, and repository full validation are terminal success for the current head/base identity.

### 2. Thread `PRRT_kwDOTN39T86Odfx9` — Resolve the pre-commit summary dependency

- File: `docs/DESIGN.md`
- Line: 307
- Finding basis: tracegit commit says validation happens before git commit , but summary may be derived from the commit message when omitted. That value is not available up front for editor-driven or hook-mutated commits, so this flow can't work for every supported mode. Either require an explicit summary before commit, or move final validation/note creation until after git commit returns.

**Normative resolution**: At `docs/DESIGN.md:307`, the canonical contract SHALL adopt this exact decision: Explicit `--summary` is validated before commit for the non-editor mode; editor/hook modes SHALL commit, read the final commit message, validate that final summary, and only then write the note.

**Focused verification gate**: Exercise explicit-summary, editor, plain commit, and hook-rewrite fixtures; assert final-message validation/note creation follows the selected mode.

**Completion boundary**: This contract is design-level only. Resolve the GitHub thread only after the focused gate, applicable specialist handoff, and repository full validation are terminal success for the current head/base identity.

### 3. Thread `PRRT_kwDOTN39T86Odfx-` — Do not hard-code world-readable export files

- File: `docs/DESIGN.md`
- Line: 384
- Finding basis: --output is documented as creating files with mode 0644 , which can expose exported rationale to other local users. Please use the process umask or 0600 instead.

**Normative resolution**: At `docs/DESIGN.md:384`, the canonical contract SHALL make this requirement normative: --output is documented as creating files with mode 0644 , which can expose exported rationale to other local users. Please use the process umask or 0600 instead.. Unsafe, malformed, or unsupported inputs SHALL fail at the stated boundary without leaking data or leaving an unintended partial side effect.

**Focused verification gate**: Export under permissive/restrictive umasks and inspect the resulting mode; assert no world-readable output and no publication on permission failure.

**Completion boundary**: This contract is design-level only. Resolve the GitHub thread only after the focused gate, applicable specialist handoff, and repository full validation are terminal success for the current head/base identity.

### 4. Thread `PRRT_kwDOTN39T86Odfx_` — Use deep equality for the round-trip criterion

- File: `docs/issues/03-rationale-record-schema.md`
- Line: 40
- Finding basis: Record includes slice/map-backed data ( tags and preserved unknown fields), so Unmarshal(Marshal(r)) == r is not a valid Go comparison. Spell this as deep-equals (or equivalent) in the acceptance criteria.

**Normative resolution**: At `docs/issues/03-rationale-record-schema.md:40`, the canonical contract SHALL adopt this exact decision: The acceptance criterion SHALL use `reflect.DeepEqual` or an equivalent deep structural comparison for `Unmarshal(Marshal(record))`, including slices and maps.

**Focused verification gate**: Round-trip records with nil/empty/non-empty slices and map-backed unknown fields; assert deep structural equality, not pointer/`==` equality.

**Completion boundary**: This contract is design-level only. Resolve the GitHub thread only after the focused gate, applicable specialist handoff, and repository full validation are terminal success for the current head/base identity.

### 5. Thread `PRRT_kwDOTN39T86OdfyA` — Make the fuzz check rune-aware

- File: `docs/issues/06-output-sanitization.md`
- Line: 41
- Finding basis: A raw byte blacklist will reject valid UTF-8 output, which contradicts the requirement to preserve 日本語 , emoji, etc. Please assert valid UTF-8 and absence of control runes/ESC sequences instead.

**Normative resolution**: At `docs/issues/06-output-sanitization.md:41`, the canonical contract SHALL make this requirement normative: A raw byte blacklist will reject valid UTF-8 output, which contradicts the requirement to preserve 日本語 , emoji, etc. Please assert valid UTF-8 and absence of control runes/ESC sequences instead.. Unsafe, malformed, or unsupported inputs SHALL fail at the stated boundary without leaking data or leaving an unintended partial side effect.

**Focused verification gate**: Fuzz actual C0/C1/ESC runes, invalid UTF-8, Japanese, and emoji; assert unsafe code points are handled while valid Unicode remains unchanged.

**Completion boundary**: This contract is design-level only. Resolve the GitHub thread only after the focused gate, applicable specialist handoff, and repository full validation are terminal success for the current head/base identity.

### 6. Thread `PRRT_kwDOTN39T86OdfyC` — Add a commit-scoped lookup path for the `Show` fallback

- File: `docs/issues/08-git-notes-wrapper.md`
- Line: 26
- Finding basis: Show is required to confirm ambiguous "no note" cases with List(sha) , but the current List(ctx) API only exposes a ref-wide scan. That contract gap makes the fallback expensive and awkward to implement.

**Normative resolution**: At `docs/issues/08-git-notes-wrapper.md:26`, the canonical contract SHALL make this requirement normative: Show is required to confirm ambiguous "no note" cases with List(sha) , but the current List(ctx) API only exposes a ref-wide scan. That contract gap makes the fallback expensive and awkward to implement.. Unsafe, malformed, or unsupported inputs SHALL fail at the stated boundary without leaking data or leaving an unintended partial side effect.

**Focused verification gate**: Run present, absent, ambiguous-stderr, malformed, and Git-error `Show` cases; assert commit-scoped fallback and bounded output distinguish absence from error.

**Completion boundary**: This contract is design-level only. Resolve the GitHub thread only after the focused gate, applicable specialist handoff, and repository full validation are terminal success for the current head/base identity.

### 7. Thread `PRRT_kwDOTN39T86OdfyD` — Expose parser warnings explicitly

- File: `docs/issues/10-blame-porcelain-parser.md`
- Line: 31
- Finding basis: The contract promises skipped malformed lines with accumulated non-fatal warnings, but Blame(...) only returns lines , commits , and err . That leaves recoverable parse diagnostics with nowhere to go.

**Normative resolution**: At `docs/issues/10-blame-porcelain-parser.md:31`, the canonical contract SHALL make this requirement normative: The contract promises skipped malformed lines with accumulated non-fatal warnings, but Blame(...) only returns lines , commits , and err . That leaves recoverable parse diagnostics with nowhere to go.. Unsafe, malformed, or unsupported inputs SHALL fail at the stated boundary without leaking data or leaving an unintended partial side effect.

**Focused verification gate**: Parse valid, one-malformed, and fully-malformed blame streams; assert warnings for skipped lines, fatal error for unusable input, and no panic.

**Completion boundary**: This contract is design-level only. Resolve the GitHub thread only after the focused gate, applicable specialist handoff, and repository full validation are terminal success for the current head/base identity.

### 8. Thread `PRRT_kwDOTN39T86OdfyF` — Avoid exact stderr matching for empty history

- File: `docs/issues/11-log-parser.md`
- Line: 25
- Finding basis: git log on an empty repo exits 128 and the fatal message includes the current branch name, so a full-string compare will be brittle; handle the empty-history case with a stable pre-check or a broader absence check instead.

**Normative resolution**: At `docs/issues/11-log-parser.md:25`, the canonical contract SHALL make this requirement normative: git log on an empty repo exits 128 and the fatal message includes the current branch name, so a full-string compare will be brittle; handle the empty-history case with a stable pre-check or a broader absence check instead.. Unsafe, malformed, or unsupported inputs SHALL fail at the stated boundary without leaking data or leaving an unintended partial side effect.

**Focused verification gate**: Run empty repository, empty branch, missing repository, permission failure, and normal history; assert only confirmed empty history returns empty output.

**Completion boundary**: This contract is design-level only. Resolve the GitHub thread only after the focused gate, applicable specialist handoff, and repository full validation are terminal success for the current head/base identity.

### 9. Thread `PRRT_kwDOTN39T86OdfyG` — Derive `summary` from the final committed message

- File: `docs/issues/13-commit-command.md`
- Line: 29
- Finding basis: git commit --edit and commit hooks can rewrite the message, so using the pre-commit assembly can make the note’s summary disagree with the actual commit. Read back the committed object after git.Commit(...) and fall back to that text.

**Normative resolution**: At `docs/issues/13-commit-command.md:29`, the canonical contract SHALL adopt this exact decision: The note summary SHALL be read from the final committed object after `git.Commit`, so editor and hook rewrites cannot leave the note inconsistent with the commit.

**Focused verification gate**: Use editor and commit-hook rewrites; assert the note summary equals the final commit object and a readback failure prevents a misleading note.

**Completion boundary**: This contract is design-level only. Resolve the GitHub thread only after the focused gate, applicable specialist handoff, and repository full validation are terminal success for the current head/base identity.

### 10. Thread `PRRT_kwDOTN39T86OdfyI` — Make the recovery hint source-agnostic

- File: `docs/issues/13-commit-command.md`
- Line: 32
- Finding basis: --rationale-file is only one input path here; the same failure path also accepts inline and stdin rationales. As written, the hint is not actionable for those cases.

**Normative resolution**: At `docs/issues/13-commit-command.md:32`, the canonical contract SHALL make this requirement normative: --rationale-file is only one input path here; the same failure path also accepts inline and stdin rationales. As written, the hint is not actionable for those cases.. Unsafe, malformed, or unsupported inputs SHALL fail at the stated boundary without leaking data or leaving an unintended partial side effect.

**Focused verification gate**: Trigger the same failure via file, inline, stdin, and editor rationale sources; assert each recovery hint names the actual source.

**Completion boundary**: This contract is design-level only. Resolve the GitHub thread only after the focused gate, applicable specialist handoff, and repository full validation are terminal success for the current head/base identity.

### 11. Thread `PRRT_kwDOTN39T86OdfyJ` — Cache the degraded state, not just the parsed record

- File: `docs/issues/16-blame-command.md`
- Line: 27
- Finding basis: map[sha] record.Record can only represent a parsed note or nil , but this spec also requires a distinct degraded JSON shape ( error / raw ) and a human fallback for malformed notes. As written, the cache collapses “absent” and “degraded” into the same value, so later rendering cannot tell which path to take.

**Normative resolution**: At `docs/issues/16-blame-command.md:27`, the canonical contract SHALL make this requirement normative: map[sha] record.Record can only represent a parsed note or nil , but this spec also requires a distinct degraded JSON shape ( error / raw ) and a human fallback for malformed notes. As written, the cache collapses “absent” and “degraded” into the same value, so later rendering cannot tell which path to take.. Unsafe, malformed, or unsupported inputs SHALL fail at the stated boundary without leaking data or leaving an unintended partial side effect.

**Focused verification gate**: Cache absent, valid, malformed, oversized, and repeated note lookups; assert tagged states remain distinct and raw evidence stays bounded.

**Completion boundary**: This contract is design-level only. Resolve the GitHub thread only after the focused gate, applicable specialist handoff, and repository full validation are terminal success for the current head/base identity.

### 12. Thread `PRRT_kwDOTN39T86OdfyK` — Keep the `rationale` JSON shape stable

- File: `docs/issues/17-log-command.md`
- Line: 20
- Finding basis: DESIGN.md defines tracegit log --json as commit + rationale:<record|null . Widening rationale to a degraded object will break consumers that rely on the documented schema; if you need to surface decode failures, add a sibling field instead of changing rationale .

**Normative resolution**: At `docs/issues/17-log-command.md:20`, the canonical contract SHALL make this requirement normative: DESIGN.md defines tracegit log --json as commit + rationale:<record|null . Widening rationale to a degraded object will break consumers that rely on the documented schema; if you need to surface decode failures, add a sibling field instead of changing rationale .. Unsafe, malformed, or unsupported inputs SHALL fail at the stated boundary without leaking data or leaving an unintended partial side effect.

**Focused verification gate**: Serialize valid, absent, and malformed notes; validate `rationale: record|null` and assert decode diagnostics use a sibling field.

**Completion boundary**: This contract is design-level only. Resolve the GitHub thread only after the focused gate, applicable specialist handoff, and repository full validation are terminal success for the current head/base identity.

### 13. Thread `PRRT_kwDOTN39T86OdfyN` — Drop `union` from the canonical sync path

- File: `docs/issues/19-sync-command.md`
- Line: 22
- Finding basis: Allowing union here can produce non-JSON note bodies, which breaks the canonical record contract and will cascade into show / log / export parsing failures. Keep v1 to strategies that preserve the JSON schema, or scope union away from refs/notes/tracegit .

**Normative resolution**: At `docs/issues/19-sync-command.md:22`, the canonical contract SHALL adopt this exact decision: `union` SHALL be rejected for the canonical `refs/notes/tracegit` v1 path before mutation; only the documented JSON-preserving strategies are accepted.

**Focused verification gate**: Run manual/ours/theirs and `union` conflicts; assert supported strategies preserve JSON and `union` is rejected before ref mutation.

**Completion boundary**: This contract is design-level only. Resolve the GitHub thread only after the focused gate, applicable specialist handoff, and repository full validation are terminal success for the current head/base identity.

### 14. Thread `PRRT_kwDOTN39T86OdfyP` — Persist the resolved notes ref, not just the explicit flag

- File: `docs/issues/20-init-command.md`
- Line: 17
- Finding basis: init is supposed to record the chosen defaults, but this wording only covers a passed --notes-ref . If the active ref comes from env or git-config precedence, the repo-local config will silently diverge from the value tracegit actually used.

**Normative resolution**: At `docs/issues/20-init-command.md:17`, the canonical contract SHALL adopt this exact decision: `tracegit init` SHALL resolve CLI > environment > Git config > default, persist the resulting full notes ref, and use the same value for subsequent operations.

**Focused verification gate**: Initialize with CLI, environment, Git config, and default refs; assert persisted and subsequent-operation refs equal the resolved full ref.

**Completion boundary**: This contract is design-level only. Resolve the GitHub thread only after the focused gate, applicable specialist handoff, and repository full validation are terminal success for the current head/base identity.

### 15. Thread `PRRT_kwDOTN39T86OdfyQ` — Clarify the static-binary release requirement

- File: `docs/issues/22-oss-hardening-ci-release.md`
- Line: 29
- Finding basis: Add an explicit CGO ENABLED=0 /static-link check to the release section so it matches DESIGN §12’s single-static-binary contract and avoids publishing a dynamically linked artifact.

**Normative resolution**: At `docs/issues/22-oss-hardening-ci-release.md:29`, the canonical contract SHALL adopt this exact decision: The release gate SHALL build with `CGO_ENABLED=0` and verify a static binary; a dynamically linked or CGO-enabled artifact SHALL fail.

**Focused verification gate**: Build the release binary with the prescribed flags and inspect it with static-link checks; assert a dynamic control fails.

**Completion boundary**: This contract is design-level only. Resolve the GitHub thread only after the focused gate, applicable specialist handoff, and repository full validation are terminal success for the current head/base identity.

### 16. Thread `PRRT_kwDOTN39T86OdfyS` — Include `annotate` in the E2E lifecycle

- File: `docs/issues/23-e2e-tests-and-user-docs.md`
- Line: 19
- Finding basis: tracegit annotate is one of the v1 capture paths, but the proposed end-to-end flow never exercises it. That leaves the retroactive path and its sync round-trip untested.

**Normative resolution**: At `docs/issues/23-e2e-tests-and-user-docs.md:19`, the canonical contract SHALL make this requirement normative: tracegit annotate is one of the v1 capture paths, but the proposed end-to-end flow never exercises it. That leaves the retroactive path and its sync round-trip untested.. Unsafe, malformed, or unsupported inputs SHALL fail at the stated boundary without leaking data or leaving an unintended partial side effect.

**Focused verification gate**: Run E2E once through `commit` and once through `annotate`, then clean-clone sync; assert equivalent notes/show/blame results.

**Completion boundary**: This contract is design-level only. Resolve the GitHub thread only after the focused gate, applicable specialist handoff, and repository full validation are terminal success for the current head/base identity.

### 17. Thread `PRRT_kwDOTN39T86OdfyV` — Remove the unsupported `cat_sort_uniq` strategy

- File: `docs/research/git-notes-and-blame.md`
- Line: 55
- Finding basis: The v1 design/ADRs only expose manual , ours , theirs , and union ; adding cat sort uniq here widens the contract without CLI support, and it would mangle JSON notes too.

**Normative resolution**: At `docs/research/git-notes-and-blame.md:55`, the canonical contract SHALL make this requirement normative: The v1 design/ADRs only expose manual , ours , theirs , and union ; adding cat sort uniq here widens the contract without CLI support, and it would mangle JSON notes too.. Unsafe, malformed, or unsupported inputs SHALL fail at the stated boundary without leaking data or leaving an unintended partial side effect.

**Focused verification gate**: Enumerate supported strategies and run `cat_sort_uniq`; assert the removed value fails without changing notes.

**Completion boundary**: This contract is design-level only. Resolve the GitHub thread only after the focused gate, applicable specialist handoff, and repository full validation are terminal success for the current head/base identity.

### 18. Thread `PRRT_kwDOTN39T86OdgWt` — Avoid installing notes as the remote's default push

- File: `docs/issues/19-sync-command.md`
- Line: 16
- Finding basis: In repos that run tracegit init / sync setup , adding a push refspec under remote.<r . makes Git use that refspec for plain git push <remote ; the git-push docs say the default behavior with no <refspec is configured by the remote's push option before push.default . That means a normal branch push after setup can push only refs/notes/tracegit (or fail with src refspec ... does not match any before any note exists), so this breaks existing push workflows and makes notes sync no longer explicit; keep notes pushes in tracegit sync push instead of remote push config.

**Normative resolution**: At `docs/issues/19-sync-command.md:16`, the canonical contract SHALL adopt this exact decision: `tracegit init`/`sync setup` SHALL not change `remote.<name>.push`; notes are pushed only by the explicit `tracegit sync push` command.

**Focused verification gate**: Inspect remote config after setup and perform normal branch push plus explicit notes push; assert only the explicit command changes notes.

**Completion boundary**: This contract is design-level only. Resolve the GitHub thread only after the focused gate, applicable specialist handoff, and repository full validation are terminal success for the current head/base identity.

### 19. Thread `PRRT_kwDOTN39T86OdgWw` — Avoid force-fetching over the canonical notes ref

- File: `docs/issues/19-sync-command.md`
- Line: 17
- Finding basis: When two clones have different local notes, the configured fetch refspec +<notesRef :<notesRef updates refs/notes/tracegit in place and the leading + forces non-fast-forward updates; git fetch -h documents force as force overwrite of local reference , and the Git refspec docs say + updates even if not fast-forward. A regular git fetch after setup can therefore replace local-only rationale notes with the remote tree before the --strategy manual conflict handling below has a chance to abort; fetch to a temporary/remote-tracking notes ref and merge, or omit + .

**Normative resolution**: At `docs/issues/19-sync-command.md:17`, the canonical contract SHALL adopt this exact decision: Fetch SHALL use a non-forcing temporary/remote-tracking notes ref and apply the selected conflict strategy afterward; it SHALL not overwrite the canonical local notes ref before conflict handling.

**Focused verification gate**: Create divergent notes and inspect the canonical ref before conflict resolution; assert no force refspec and no pre-resolution overwrite.

**Completion boundary**: This contract is design-level only. Resolve the GitHub thread only after the focused gate, applicable specialist handoff, and repository full validation are terminal success for the current head/base identity.

### 20. Thread `PRRT_kwDOTN39T86OdgWz` — Use NUL-delimited log output

- File: `docs/issues/11-log-parser.md`
- Line: 21
- Finding basis: For commits whose subject (or author name) contains \x1f , this parser will split a single record into more than four fields and skip or misparse it; Git emits that byte verbatim from %s (verified with a commit message $'hello\x1fthere' ). Because commit metadata is untrusted input elsewhere in this design, unit separator does not actually avoid ambiguity; use NUL separators/terminators or an escaping/length-prefix scheme instead.

**Normative resolution**: At `docs/issues/11-log-parser.md:21`, the canonical contract SHALL make this requirement normative: For commits whose subject (or author name) contains \x1f , this parser will split a single record into more than four fields and skip or misparse it; Git emits that byte verbatim from %s (verified with a commit message $'hello\x1fthere' ). Because commit metadata is untrusted input elsewhere in this design, unit separator does not actually avoid ambiguity; use NUL separators/terminators or an escaping/length-prefix scheme instead.. Unsafe, malformed, or unsupported inputs SHALL fail at the stated boundary without leaking data or leaving an unintended partial side effect.

**Focused verification gate**: Create metadata containing `\x1f`, newline, Unicode, and empty fields; assert NUL parsing yields one correct record per commit.

**Completion boundary**: This contract is design-level only. Resolve the GitHub thread only after the focused gate, applicable specialist handoff, and repository full validation are terminal success for the current head/base identity.

### 21. Thread `PRRT_kwDOTN39T86OdgW0` — Allow summaries for editor-created commit messages

- File: `docs/issues/13-commit-command.md`
- Line: 26
- Finding basis: When the user runs the advertised editor flows (plain tracegit commit or -e ) without --summary , there is no assembled commit message before git commit runs, but this step requires validating the record, including non-empty summary , before invoking Git. Those normal commit modes will be rejected before the editor can provide a first line, so either require --summary for these modes in the CLI contract or derive and validate the summary after Git creates the commit.

**Normative resolution**: At `docs/issues/13-commit-command.md:26`, the canonical contract SHALL adopt this exact decision: Editor-created commits SHALL be allowed without `--summary`; the final subject is read and validated after Git creates the commit, while `--summary` remains an optional explicit override.

**Focused verification gate**: Run editor and `-e` flows with/without `--summary`, empty subject, and hook rewrite; assert valid final subjects succeed and notes match the final object.

**Completion boundary**: This contract is design-level only. Resolve the GitHub thread only after the focused gate, applicable specialist handoff, and repository full validation are terminal success for the current head/base identity.

### 22. Thread `PRRT_kwDOTN39T86OdgW3` — Respect Git's editor precedence

- File: `docs/issues/14-annotate-command.md`
- Line: 24
- Finding basis: For --edit , this order does not match Git's editor resolution: the git var GIT EDITOR docs say the preference is $GIT EDITOR , then core.editor , then $VISUAL , then $EDITOR , and that the value is meant to be interpreted by the shell. In repos/users with core.editor or GIT EDITOR set, tracegit would open a different editor than Git and ignore VISUAL ; resolving via git var GIT EDITOR (or matching its order/semantics) avoids surprising edit failures.

**Normative resolution**: At `docs/issues/14-annotate-command.md:24`, the canonical contract SHALL make this requirement normative: For --edit , this order does not match Git's editor resolution: the git var GIT EDITOR docs say the preference is $GIT EDITOR , then core.editor , then $VISUAL , then $EDITOR , and that the value is meant to be interpreted by the shell. In repos/users with core.editor or GIT EDITOR set, tracegit would open a different editor than Git and ignore VISUAL ; resolving via git var GIT EDITOR (or matching its order/semantics) avoids surprising edit failures.. Unsafe, malformed, or unsupported inputs SHALL fail at the stated boundary without leaking data or leaving an unintended partial side effect.

**Focused verification gate**: Set `GIT_EDITOR`, `core.editor`, `VISUAL`, and `EDITOR` independently and together; assert selection matches `git var GIT_EDITOR`.

**Completion boundary**: This contract is design-level only. Resolve the GitHub thread only after the focused gate, applicable specialist handoff, and repository full validation are terminal success for the current head/base identity.

### 23. Thread `PRRT_kwDOTN39T86OdgW5` — Handle copied notes after amend

- File: `docs/issues/13-commit-command.md`
- Line: 29
- Finding basis: When users have configured note rewriting, an amended commit can already have a note before this call runs: the git-notes docs state that notes.rewrite.<command covers amend and notes.rewriteRef specifies refs copied during rewrite, and I verified git commit --amend copies refs/notes/tracegit when configured. In that scenario notes.Add(... force=false) returns ErrNoteExists after the commit succeeds, so tracegit commit --amend reports a failed note write instead of replacing the copied old rationale; overwrite for the freshly amended SHA.

**Normative resolution**: At `docs/issues/13-commit-command.md:29`, the canonical contract SHALL make this requirement normative: When users have configured note rewriting, an amended commit can already have a note before this call runs: the git-notes docs state that notes.rewrite.<command covers amend and notes.rewriteRef specifies refs copied during rewrite, and I verified git commit --amend copies refs/notes/tracegit when configured. In that scenario notes.Add(... force=false) returns ErrNoteExists after the commit succeeds, so tracegit commit --amend reports a failed note write instead of replacing the copied old rationale; overwrite for the freshly amended SHA.. Unsafe, malformed, or unsupported inputs SHALL fail at the stated boundary without leaking data or leaving an unintended partial side effect.

**Focused verification gate**: Configure note rewrite, amend a noted commit, and compare old/new SHAs; assert only the new SHA is overwritten with final rationale.

**Completion boundary**: This contract is design-level only. Resolve the GitHub thread only after the focused gate, applicable specialist handoff, and repository full validation are terminal success for the current head/base identity.

### 24. Thread `PRRT_kwDOTN39T86OdgW6` — Bound note output before buffering it

- File: `docs/issues/08-git-notes-wrapper.md`
- Line: 22
- Finding basis: For a hostile fetched note that is larger than the read ceiling, git notes show will be captured into memory by the runner before callers can apply ClampForRead , so the promised bounded read path can still allocate an attacker-sized blob. Add a max-output/bounded-reader path in Notes.Show or the runner and report truncation from there instead of returning an already fully buffered []byte .

**Normative resolution**: At `docs/issues/08-git-notes-wrapper.md:22`, the canonical contract SHALL make this requirement normative: For a hostile fetched note that is larger than the read ceiling, git notes show will be captured into memory by the runner before callers can apply ClampForRead , so the promised bounded read path can still allocate an attacker-sized blob. Add a max-output/bounded-reader path in Notes.Show or the runner and report truncation from there instead of returning an already fully buffered []byte .. Unsafe, malformed, or unsupported inputs SHALL fail at the stated boundary without leaking data or leaving an unintended partial side effect.

**Focused verification gate**: Read notes at, below, and above the hard ceiling; assert bounded consumption, explicit truncation, and no oversized buffered result.

**Completion boundary**: This contract is design-level only. Resolve the GitHub thread only after the focused gate, applicable specialist handoff, and repository full validation are terminal success for the current head/base identity.

### 25. Thread `PRRT_kwDOTN39T86OdgW7` — Bootstrap gitBinary before building the final runner

- File: `docs/issues/12-root-command.md`
- Line: 22
- Finding basis: This order cannot honor tracegit.gitBinary from git config: resolving config needs the runner per issue 05, but the runner is constructed first using the yet-unresolved GitBinary . In repos that set tracegit.gitBinary to a non-default Git, prechecks and all later commands will still use the bootstrap binary; resolve env/default first, read config, then rebuild or update the runner with the configured path.

**Normative resolution**: At `docs/issues/12-root-command.md:22`, the canonical contract SHALL make this requirement normative: This order cannot honor tracegit.gitBinary from git config: resolving config needs the runner per issue 05, but the runner is constructed first using the yet-unresolved GitBinary . In repos that set tracegit.gitBinary to a non-default Git, prechecks and all later commands will still use the bootstrap binary; resolve env/default first, read config, then rebuild or update the runner with the configured path.. Unsafe, malformed, or unsupported inputs SHALL fail at the stated boundary without leaking data or leaving an unintended partial side effect.

**Focused verification gate**: Set a non-default `tracegit.gitBinary`; assert bootstrap Git reads config before final runner construction and the configured binary is used.

**Completion boundary**: This contract is design-level only. Resolve the GitHub thread only after the focused gate, applicable specialist handoff, and repository full validation are terminal success for the current head/base identity.

### 26. Thread `PRRT_kwDOTN39T86OdgW9` — Normalize short notes refs before syncing

- File: `docs/issues/05-config-resolution.md`
- Line: 30
- Finding basis: Allowing a bare notes ref such as tracegit here makes normal notes operations use Git's --ref namespace expansion ( refs/notes/tracegit ), but sync fetch/push later uses the raw value as a refspec ( tracegit:tracegit ), which does not fetch or push refs/notes/tracegit and fails against a remote containing only the notes ref. Normalize accepted short names to full refs/notes/<name before storing them in config, or require full refs/notes/... values.

**Normative resolution**: At `docs/issues/05-config-resolution.md:30`, the canonical contract SHALL adopt this exact decision: Short notes names SHALL be normalized to `refs/notes/<name>` before persistence and sync; invalid non-notes refs SHALL be rejected.

**Focused verification gate**: Resolve short/full/malformed refs from every precedence source; assert persistence and sync always use the same full `refs/notes/...` value.

**Completion boundary**: This contract is design-level only. Resolve the GitHub thread only after the focused gate, applicable specialist handoff, and repository full validation are terminal success for the current head/base identity.

### 27. Thread `PRRT_kwDOTN39T86OdgW-` — Check C1 controls as runes, not UTF-8 bytes

- File: `docs/issues/06-output-sanitization.md`
- Line: 41
- Finding basis: The fuzz assertion cannot coexist with the acceptance criterion that emoji and other non-ASCII UTF-8 pass unchanged: printable characters commonly contain continuation bytes in 0x80 – 0x9f (for example 🚀 includes such bytes), so a byte-level forbidden-set check would either fail valid output or force the sanitizer to corrupt it. Treat C1 controls as decoded code points U+0080 – U+009F , not raw bytes inside UTF-8 sequences.

**Normative resolution**: At `docs/issues/06-output-sanitization.md:41`, the canonical contract SHALL make this requirement normative: The fuzz assertion cannot coexist with the acceptance criterion that emoji and other non-ASCII UTF-8 pass unchanged: printable characters commonly contain continuation bytes in 0x80 – 0x9f (for example 🚀 includes such bytes), so a byte-level forbidden-set check would either fail valid output or force the sanitizer to corrupt it. Treat C1 controls as decoded code points U+0080 – U+009F , not raw bytes inside UTF-8 sequences.. Unsafe, malformed, or unsupported inputs SHALL fail at the stated boundary without leaking data or leaving an unintended partial side effect.

**Focused verification gate**: Sanitize actual C1 runes and UTF-8 emoji/Japanese; assert code-point policy and byte-preserving valid Unicode.

**Completion boundary**: This contract is design-level only. Resolve the GitHub thread only after the focused gate, applicable specialist handoff, and repository full validation are terminal success for the current head/base identity.

### 28. Thread `PRRT_kwDOTN39T86OdgXC` — Cap writable rationale size below the read ceiling

- File: `docs/issues/04-record-validation-limits.md`
- Line: 18
- Finding basis: Because RationaleMaxBytes is user-overridable and config validation only requires a positive integer, a user can write a valid record larger than ReadHardCeilingBytes (for example with --max-rationale-bytes=2097152 ), after which every read path will truncate that same record at 1 MiB and degrade it. Either enforce an upper bound on the configured write limit or make the read ceiling at least as large as the maximum writable note size.

**Normative resolution**: At `docs/issues/04-record-validation-limits.md:18`, the canonical contract SHALL make this requirement normative: Because RationaleMaxBytes is user-overridable and config validation only requires a positive integer, a user can write a valid record larger than ReadHardCeilingBytes (for example with --max-rationale-bytes=2097152 ), after which every read path will truncate that same record at 1 MiB and degrade it. Either enforce an upper bound on the configured write limit or make the read ceiling at least as large as the maximum writable note size.. Unsafe, malformed, or unsupported inputs SHALL fail at the stated boundary without leaking data or leaving an unintended partial side effect.

**Focused verification gate**: Set write limits below, equal to, and above the read ceiling and write boundary records; assert incompatible settings fail and accepted records are never truncated.

**Completion boundary**: This contract is design-level only. Resolve the GitHub thread only after the focused gate, applicable specialist handoff, and repository full validation are terminal success for the current head/base identity.

## Merge boundary

- `gate-task-evaluator` must re-fetch PR state, current head/base, required-check inventory, review decision, unresolved thread state, policy version, and merge candidate immediately before any merge mutation.
- `github_mergeable` or a successful CodeRabbit status is not merge authorization.
- The current task instruction allows at most one PR Bot review; this artifact authorizes no Bot trigger or rerun.