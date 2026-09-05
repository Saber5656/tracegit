# Wave 8 concrete review-resolution correction

- Repository: `Saber5656/tracegit`
- Pull request: #1
- Base SHA pinned before correction: `9eacb8b1984731cf61b7837188b86f23cf54818e`
- PR head pinned as correction parent: `ae0e469a9f4705b398ca7d9874557de8a028fcf2`
- This file is the concrete correction artifact for the existing PR and supersedes the generic wording in `docs/REVIEW_RESOLUTION.md` for these threads.
- Every entry is bound to the exact current review-thread ID, path, and line. Any later head/base/thread change invalidates the evidence and requires a fresh preflight.
- This is documentation-level contract handling for documentation-only PRs. It does not claim product implementation, runtime tests, build, CI, security, or release validation is complete.
- Focused verification, applicable QA/security review, repository full validation, and the separate merge gate remain blocking before resolve/merge.
- The PR review bot is not re-triggered.

## Gate contract

| Gate | Blocking rule |
|---|---|
| Thread identity | ID/path/line still match a current non-outdated thread |
| Normative contract | The SHALL statement below is the canonical decision |
| Focused verification | The per-thread check has terminal evidence |
| QA/security | Applicable specialist review accepts |
| Full validation | Repository-prescribed validation passes for final head |
| Merge | Current head/base/check/review/thread/policy identity passes |

## 1. Thread `PRRT_kwDOTN39T86Odfx8` — Don't hard-code the license as MIT yet

- File: `docs/DESIGN.md`
- Line: 128
- Finding basis: LICENSE MIT reads like a resolved decision, but ADR-002 still leaves the license as an open item to confirm before public release. Keep this placeholder neutral until that decision is closed.

**Normative resolution**: At `docs/DESIGN.md:128`, the canonical contract SHALL adopt the concrete requirement in this finding: LICENSE MIT reads like a resolved decision, but ADR-002 still leaves the license as an open item to confirm before public release. Keep this placeholder neutral until that decision is closed.. An unresolved owner decision blocks release and must not be presented as final.

**Focused verification gate**: Run release preflight with the decision unresolved and resolved; assert only the owner-resolved case can pass publication.

**Completion boundary**: This entry records the design-level acceptance contract only. Do not resolve the GitHub thread until the focused result, applicable specialist review, and final repository validation are terminal success for the pinned identity.

## 2. Thread `PRRT_kwDOTN39T86Odfx9` — Resolve the pre-commit summary dependency

- File: `docs/DESIGN.md`
- Line: 307
- Finding basis: tracegit commit says validation happens before git commit , but summary may be derived from the commit message when omitted. That value is not available up front for editor-driven or hook-mutated commits, so this flow can't work for every supported mode. Either require an explicit summary before commit, or move final validation/note creation until after git commit returns.

**Normative resolution**: At `docs/DESIGN.md:307`, the canonical contract SHALL adopt the concrete requirement in this finding: tracegit commit says validation happens before git commit , but summary may be derived from the commit message when omitted. That value is not available up front for editor-driven or hook-mutated commits, so this flow can't work for every supported mode. Either require an explicit summary before commit, or move final validation/note creation until after git commit returns.. The named graph edge/exception is mandatory; a missing edge blocks scheduling and acceptance.

**Focused verification gate**: Build the dependency/layering graph from `docs/DESIGN.md:307`; assert the named edge or exception is present, ordered correctly, and a negative graph is rejected.

**Completion boundary**: This entry records the design-level acceptance contract only. Do not resolve the GitHub thread until the focused result, applicable specialist review, and final repository validation are terminal success for the pinned identity.

## 3. Thread `PRRT_kwDOTN39T86Odfx-` — Do not hard-code world-readable export files

- File: `docs/DESIGN.md`
- Line: 384
- Finding basis: --output is documented as creating files with mode 0644 , which can expose exported rationale to other local users. Please use the process umask or 0600 instead.

**Normative resolution**: At `docs/DESIGN.md:384`, the canonical contract SHALL adopt the concrete requirement in this finding: --output is documented as creating files with mode 0644 , which can expose exported rationale to other local users. Please use the process umask or 0600 instead.. Unsafe, malformed, or unsupported input must fail at the stated boundary without leaking data or creating an unintended side effect.

**Focused verification gate**: Run the positive control and the exact negative/boundary case named in the finding for `docs/DESIGN.md:384`; assert the canonical result and error behavior.

**Completion boundary**: This entry records the design-level acceptance contract only. Do not resolve the GitHub thread until the focused result, applicable specialist review, and final repository validation are terminal success for the pinned identity.

## 4. Thread `PRRT_kwDOTN39T86Odfx_` — Use deep equality for the round-trip criterion

- File: `docs/issues/03-rationale-record-schema.md`
- Line: 40
- Finding basis: Record includes slice/map-backed data ( tags and preserved unknown fields), so Unmarshal(Marshal(r)) == r is not a valid Go comparison. Spell this as deep-equals (or equivalent) in the acceptance criteria.

**Normative resolution**: At `docs/issues/03-rationale-record-schema.md:40`, the canonical contract SHALL adopt the concrete requirement in this finding: Record includes slice/map-backed data ( tags and preserved unknown fields), so Unmarshal(Marshal(r)) == r is not a valid Go comparison. Spell this as deep-equals (or equivalent) in the acceptance criteria.. Unsafe, malformed, or unsupported input must fail at the stated boundary without leaking data or creating an unintended side effect.

**Focused verification gate**: Run the positive control and the exact negative/boundary case named in the finding for `docs/issues/03-rationale-record-schema.md:40`; assert the canonical result and error behavior.

**Completion boundary**: This entry records the design-level acceptance contract only. Do not resolve the GitHub thread until the focused result, applicable specialist review, and final repository validation are terminal success for the pinned identity.

## 5. Thread `PRRT_kwDOTN39T86OdfyA` — Make the fuzz check rune-aware

- File: `docs/issues/06-output-sanitization.md`
- Line: 41
- Finding basis: A raw byte blacklist will reject valid UTF-8 output, which contradicts the requirement to preserve 日本語 , emoji, etc. Please assert valid UTF-8 and absence of control runes/ESC sequences instead.

**Normative resolution**: At `docs/issues/06-output-sanitization.md:41`, the canonical contract SHALL adopt the concrete requirement in this finding: A raw byte blacklist will reject valid UTF-8 output, which contradicts the requirement to preserve 日本語 , emoji, etc. Please assert valid UTF-8 and absence of control runes/ESC sequences instead.. Unsafe, malformed, or unsupported input must fail at the stated boundary without leaking data or creating an unintended side effect.

**Focused verification gate**: Run the positive control and the exact negative/boundary case named in the finding for `docs/issues/06-output-sanitization.md:41`; assert the canonical result and error behavior.

**Completion boundary**: This entry records the design-level acceptance contract only. Do not resolve the GitHub thread until the focused result, applicable specialist review, and final repository validation are terminal success for the pinned identity.

## 6. Thread `PRRT_kwDOTN39T86OdfyC` — Add a commit-scoped lookup path for the `Show` fallback

- File: `docs/issues/08-git-notes-wrapper.md`
- Line: 26
- Finding basis: Show is required to confirm ambiguous "no note" cases with List(sha) , but the current List(ctx) API only exposes a ref-wide scan. That contract gap makes the fallback expensive and awkward to implement.

**Normative resolution**: At `docs/issues/08-git-notes-wrapper.md:26`, the canonical contract SHALL adopt the concrete requirement in this finding: Show is required to confirm ambiguous "no note" cases with List(sha) , but the current List(ctx) API only exposes a ref-wide scan. That contract gap makes the fallback expensive and awkward to implement.. Unsafe, malformed, or unsupported input must fail at the stated boundary without leaking data or creating an unintended side effect.

**Focused verification gate**: Run a safe control and each unsafe/boundary fixture named in the finding for `docs/issues/08-git-notes-wrapper.md:26`; assert rejection/containment, no sensitive-data exposure, and no partial side effect.

**Completion boundary**: This entry records the design-level acceptance contract only. Do not resolve the GitHub thread until the focused result, applicable specialist review, and final repository validation are terminal success for the pinned identity.

## 7. Thread `PRRT_kwDOTN39T86OdfyD` — Expose parser warnings explicitly

- File: `docs/issues/10-blame-porcelain-parser.md`
- Line: 31
- Finding basis: The contract promises skipped malformed lines with accumulated non-fatal warnings, but Blame(...) only returns lines , commits , and err . That leaves recoverable parse diagnostics with nowhere to go.

**Normative resolution**: At `docs/issues/10-blame-porcelain-parser.md:31`, the canonical contract SHALL adopt the concrete requirement in this finding: The contract promises skipped malformed lines with accumulated non-fatal warnings, but Blame(...) only returns lines , commits , and err . That leaves recoverable parse diagnostics with nowhere to go.. Unsafe, malformed, or unsupported input must fail at the stated boundary without leaking data or creating an unintended side effect.

**Focused verification gate**: Run the positive control and the exact negative/boundary case named in the finding for `docs/issues/10-blame-porcelain-parser.md:31`; assert the canonical result and error behavior.

**Completion boundary**: This entry records the design-level acceptance contract only. Do not resolve the GitHub thread until the focused result, applicable specialist review, and final repository validation are terminal success for the pinned identity.

## 8. Thread `PRRT_kwDOTN39T86OdfyF` — Avoid exact stderr matching for empty history

- File: `docs/issues/11-log-parser.md`
- Line: 25
- Finding basis: git log on an empty repo exits 128 and the fatal message includes the current branch name, so a full-string compare will be brittle; handle the empty-history case with a stable pre-check or a broader absence check instead.

**Normative resolution**: At `docs/issues/11-log-parser.md:25`, the canonical contract SHALL adopt the concrete requirement in this finding: git log on an empty repo exits 128 and the fatal message includes the current branch name, so a full-string compare will be brittle; handle the empty-history case with a stable pre-check or a broader absence check instead.. Unsafe, malformed, or unsupported input must fail at the stated boundary without leaking data or creating an unintended side effect.

**Focused verification gate**: Run the positive control and the exact negative/boundary case named in the finding for `docs/issues/11-log-parser.md:25`; assert the canonical result and error behavior.

**Completion boundary**: This entry records the design-level acceptance contract only. Do not resolve the GitHub thread until the focused result, applicable specialist review, and final repository validation are terminal success for the pinned identity.

## 9. Thread `PRRT_kwDOTN39T86OdfyG` — Derive `summary` from the final committed message

- File: `docs/issues/13-commit-command.md`
- Line: 29
- Finding basis: git commit --edit and commit hooks can rewrite the message, so using the pre-commit assembly can make the note’s summary disagree with the actual commit. Read back the committed object after git.Commit(...) and fall back to that text.

**Normative resolution**: At `docs/issues/13-commit-command.md:29`, the canonical contract SHALL adopt the concrete requirement in this finding: git commit --edit and commit hooks can rewrite the message, so using the pre-commit assembly can make the note’s summary disagree with the actual commit. Read back the committed object after git.Commit(...) and fall back to that text.. Unsafe, malformed, or unsupported input must fail at the stated boundary without leaking data or creating an unintended side effect.

**Focused verification gate**: Exercise normal, editor/hook/rewrite, and failure cases for `docs/issues/13-commit-command.md:29`; assert the result comes from the final canonical object/ref and recovery is actionable.

**Completion boundary**: This entry records the design-level acceptance contract only. Do not resolve the GitHub thread until the focused result, applicable specialist review, and final repository validation are terminal success for the pinned identity.

## 10. Thread `PRRT_kwDOTN39T86OdfyI` — Make the recovery hint source-agnostic

- File: `docs/issues/13-commit-command.md`
- Line: 32
- Finding basis: --rationale-file is only one input path here; the same failure path also accepts inline and stdin rationales. As written, the hint is not actionable for those cases.

**Normative resolution**: At `docs/issues/13-commit-command.md:32`, the canonical contract SHALL adopt the concrete requirement in this finding: --rationale-file is only one input path here; the same failure path also accepts inline and stdin rationales. As written, the hint is not actionable for those cases.. Unsafe, malformed, or unsupported input must fail at the stated boundary without leaking data or creating an unintended side effect.

**Focused verification gate**: Run the positive control and the exact negative/boundary case named in the finding for `docs/issues/13-commit-command.md:32`; assert the canonical result and error behavior.

**Completion boundary**: This entry records the design-level acceptance contract only. Do not resolve the GitHub thread until the focused result, applicable specialist review, and final repository validation are terminal success for the pinned identity.

## 11. Thread `PRRT_kwDOTN39T86OdfyJ` — Cache the degraded state, not just the parsed record

- File: `docs/issues/16-blame-command.md`
- Line: 27
- Finding basis: map[sha] record.Record can only represent a parsed note or nil , but this spec also requires a distinct degraded JSON shape ( error / raw ) and a human fallback for malformed notes. As written, the cache collapses “absent” and “degraded” into the same value, so later rendering cannot tell which path to take.

**Normative resolution**: At `docs/issues/16-blame-command.md:27`, the canonical contract SHALL adopt the concrete requirement in this finding: map[sha] record.Record can only represent a parsed note or nil , but this spec also requires a distinct degraded JSON shape ( error / raw ) and a human fallback for malformed notes. As written, the cache collapses “absent” and “degraded” into the same value, so later rendering cannot tell which path to take.. Unsafe, malformed, or unsupported input must fail at the stated boundary without leaking data or creating an unintended side effect.

**Focused verification gate**: Run the positive control and the exact negative/boundary case named in the finding for `docs/issues/16-blame-command.md:27`; assert the canonical result and error behavior.

**Completion boundary**: This entry records the design-level acceptance contract only. Do not resolve the GitHub thread until the focused result, applicable specialist review, and final repository validation are terminal success for the pinned identity.

## 12. Thread `PRRT_kwDOTN39T86OdfyK` — Keep the `rationale` JSON shape stable

- File: `docs/issues/17-log-command.md`
- Line: 20
- Finding basis: DESIGN.md defines tracegit log --json as commit + rationale:<record|null . Widening rationale to a degraded object will break consumers that rely on the documented schema; if you need to surface decode failures, add a sibling field instead of changing rationale .

**Normative resolution**: At `docs/issues/17-log-command.md:20`, the canonical contract SHALL adopt the concrete requirement in this finding: DESIGN.md defines tracegit log --json as commit + rationale:<record|null . Widening rationale to a degraded object will break consumers that rely on the documented schema; if you need to surface decode failures, add a sibling field instead of changing rationale .. The positive and invalid shapes must be deterministic and compatible with downstream consumers.

**Focused verification gate**: Validate canonical positive, missing, wrong-type, unknown-field, and boundary fixtures for `docs/issues/17-log-command.md:20`; assert the stated shape and failure behavior exactly.

**Completion boundary**: This entry records the design-level acceptance contract only. Do not resolve the GitHub thread until the focused result, applicable specialist review, and final repository validation are terminal success for the pinned identity.

## 13. Thread `PRRT_kwDOTN39T86OdfyN` — Drop `union` from the canonical sync path

- File: `docs/issues/19-sync-command.md`
- Line: 22
- Finding basis: Allowing union here can produce non-JSON note bodies, which breaks the canonical record contract and will cascade into show / log / export parsing failures. Keep v1 to strategies that preserve the JSON schema, or scope union away from refs/notes/tracegit .

**Normative resolution**: At `docs/issues/19-sync-command.md:22`, the canonical contract SHALL adopt the concrete requirement in this finding: Allowing union here can produce non-JSON note bodies, which breaks the canonical record contract and will cascade into show / log / export parsing failures. Keep v1 to strategies that preserve the JSON schema, or scope union away from refs/notes/tracegit .. Unsafe, malformed, or unsupported input must fail at the stated boundary without leaking data or creating an unintended side effect.

**Focused verification gate**: Run a safe control and each unsafe/boundary fixture named in the finding for `docs/issues/19-sync-command.md:22`; assert rejection/containment, no sensitive-data exposure, and no partial side effect.

**Completion boundary**: This entry records the design-level acceptance contract only. Do not resolve the GitHub thread until the focused result, applicable specialist review, and final repository validation are terminal success for the pinned identity.

## 14. Thread `PRRT_kwDOTN39T86OdfyP` — Persist the resolved notes ref, not just the explicit flag

- File: `docs/issues/20-init-command.md`
- Line: 17
- Finding basis: init is supposed to record the chosen defaults, but this wording only covers a passed --notes-ref . If the active ref comes from env or git-config precedence, the repo-local config will silently diverge from the value tracegit actually used.

**Normative resolution**: At `docs/issues/20-init-command.md:17`, the canonical contract SHALL adopt the concrete requirement in this finding: init is supposed to record the chosen defaults, but this wording only covers a passed --notes-ref . If the active ref comes from env or git-config precedence, the repo-local config will silently diverge from the value tracegit actually used.. Unsafe, malformed, or unsupported input must fail at the stated boundary without leaking data or creating an unintended side effect.

**Focused verification gate**: Exercise normal, editor/hook/rewrite, and failure cases for `docs/issues/20-init-command.md:17`; assert the result comes from the final canonical object/ref and recovery is actionable.

**Completion boundary**: This entry records the design-level acceptance contract only. Do not resolve the GitHub thread until the focused result, applicable specialist review, and final repository validation are terminal success for the pinned identity.

## 15. Thread `PRRT_kwDOTN39T86OdfyQ` — Clarify the static-binary release requirement

- File: `docs/issues/22-oss-hardening-ci-release.md`
- Line: 29
- Finding basis: Add an explicit CGO ENABLED=0 /static-link check to the release section so it matches DESIGN §12’s single-static-binary contract and avoids publishing a dynamically linked artifact.

**Normative resolution**: At `docs/issues/22-oss-hardening-ci-release.md:29`, the canonical contract SHALL adopt the concrete requirement in this finding: Add an explicit CGO ENABLED=0 /static-link check to the release section so it matches DESIGN §12’s single-static-binary contract and avoids publishing a dynamically linked artifact.. Unsafe, malformed, or unsupported input must fail at the stated boundary without leaking data or creating an unintended side effect.

**Focused verification gate**: Run the positive control and the exact negative/boundary case named in the finding for `docs/issues/22-oss-hardening-ci-release.md:29`; assert the canonical result and error behavior.

**Completion boundary**: This entry records the design-level acceptance contract only. Do not resolve the GitHub thread until the focused result, applicable specialist review, and final repository validation are terminal success for the pinned identity.

## 16. Thread `PRRT_kwDOTN39T86OdfyS` — Include `annotate` in the E2E lifecycle

- File: `docs/issues/23-e2e-tests-and-user-docs.md`
- Line: 19
- Finding basis: tracegit annotate is one of the v1 capture paths, but the proposed end-to-end flow never exercises it. That leaves the retroactive path and its sync round-trip untested.

**Normative resolution**: At `docs/issues/23-e2e-tests-and-user-docs.md:19`, the canonical contract SHALL adopt the concrete requirement in this finding: tracegit annotate is one of the v1 capture paths, but the proposed end-to-end flow never exercises it. That leaves the retroactive path and its sync round-trip untested.. Unsafe, malformed, or unsupported input must fail at the stated boundary without leaking data or creating an unintended side effect.

**Focused verification gate**: Run the positive control and the exact negative/boundary case named in the finding for `docs/issues/23-e2e-tests-and-user-docs.md:19`; assert the canonical result and error behavior.

**Completion boundary**: This entry records the design-level acceptance contract only. Do not resolve the GitHub thread until the focused result, applicable specialist review, and final repository validation are terminal success for the pinned identity.

## 17. Thread `PRRT_kwDOTN39T86OdfyV` — Remove the unsupported `cat_sort_uniq` strategy

- File: `docs/research/git-notes-and-blame.md`
- Line: 55
- Finding basis: The v1 design/ADRs only expose manual , ours , theirs , and union ; adding cat sort uniq here widens the contract without CLI support, and it would mangle JSON notes too.

**Normative resolution**: At `docs/research/git-notes-and-blame.md:55`, the canonical contract SHALL adopt the concrete requirement in this finding: The v1 design/ADRs only expose manual , ours , theirs , and union ; adding cat sort uniq here widens the contract without CLI support, and it would mangle JSON notes too.. Unsafe, malformed, or unsupported input must fail at the stated boundary without leaking data or creating an unintended side effect.

**Focused verification gate**: Run the positive control and the exact negative/boundary case named in the finding for `docs/research/git-notes-and-blame.md:55`; assert the canonical result and error behavior.

**Completion boundary**: This entry records the design-level acceptance contract only. Do not resolve the GitHub thread until the focused result, applicable specialist review, and final repository validation are terminal success for the pinned identity.

## 18. Thread `PRRT_kwDOTN39T86OdgWt` — Avoid installing notes as the remote's default push

- File: `docs/issues/19-sync-command.md`
- Line: 16
- Finding basis: In repos that run tracegit init / sync setup , adding a push refspec under remote.<r . makes Git use that refspec for plain git push <remote ; the git-push docs say the default behavior with no <refspec is configured by the remote's push option before push.default . That means a normal branch push after setup can push only refs/notes/tracegit (or fail with src refspec ... does not match any before any note exists), so this breaks existing push workflows and makes notes sync no longer explicit; keep notes pushes in tracegit sync push instead of remote push config.

**Normative resolution**: At `docs/issues/19-sync-command.md:16`, the canonical contract SHALL adopt the concrete requirement in this finding: In repos that run tracegit init / sync setup , adding a push refspec under remote.<r . makes Git use that refspec for plain git push <remote ; the git-push docs say the default behavior with no <refspec is configured by the remote's push option before push.default . That means a normal branch push after setup can push only refs/notes/tracegit (or fail with src refspec ... does not match any before any note exists), so this breaks existing push workflows and makes notes sync no longer explicit; keep notes pushes in tracegit sync push instead of remote push config.. Unsafe, malformed, or unsupported input must fail at the stated boundary without leaking data or creating an unintended side effect.

**Focused verification gate**: Run a safe control and each unsafe/boundary fixture named in the finding for `docs/issues/19-sync-command.md:16`; assert rejection/containment, no sensitive-data exposure, and no partial side effect.

**Completion boundary**: This entry records the design-level acceptance contract only. Do not resolve the GitHub thread until the focused result, applicable specialist review, and final repository validation are terminal success for the pinned identity.

## 19. Thread `PRRT_kwDOTN39T86OdgWw` — Avoid force-fetching over the canonical notes ref

- File: `docs/issues/19-sync-command.md`
- Line: 17
- Finding basis: When two clones have different local notes, the configured fetch refspec +<notesRef :<notesRef updates refs/notes/tracegit in place and the leading + forces non-fast-forward updates; git fetch -h documents force as force overwrite of local reference , and the Git refspec docs say + updates even if not fast-forward. A regular git fetch after setup can therefore replace local-only rationale notes with the remote tree before the --strategy manual conflict handling below has a chance to abort; fetch to a temporary/remote-tracking notes ref and merge, or omit + .

**Normative resolution**: At `docs/issues/19-sync-command.md:17`, the canonical contract SHALL adopt the concrete requirement in this finding: When two clones have different local notes, the configured fetch refspec +<notesRef :<notesRef updates refs/notes/tracegit in place and the leading + forces non-fast-forward updates; git fetch -h documents force as force overwrite of local reference , and the Git refspec docs say + updates even if not fast-forward. A regular git fetch after setup can therefore replace local-only rationale notes with the remote tree before the --strategy manual conflict handling below has a chance to abort; fetch to a temporary/remote-tracking notes ref and merge, or omit + .. Unsafe, malformed, or unsupported input must fail at the stated boundary without leaking data or creating an unintended side effect.

**Focused verification gate**: Run a safe control and each unsafe/boundary fixture named in the finding for `docs/issues/19-sync-command.md:17`; assert rejection/containment, no sensitive-data exposure, and no partial side effect.

**Completion boundary**: This entry records the design-level acceptance contract only. Do not resolve the GitHub thread until the focused result, applicable specialist review, and final repository validation are terminal success for the pinned identity.

## 20. Thread `PRRT_kwDOTN39T86OdgWz` — Use NUL-delimited log output

- File: `docs/issues/11-log-parser.md`
- Line: 21
- Finding basis: For commits whose subject (or author name) contains \x1f , this parser will split a single record into more than four fields and skip or misparse it; Git emits that byte verbatim from %s (verified with a commit message $'hello\x1fthere' ). Because commit metadata is untrusted input elsewhere in this design, unit separator does not actually avoid ambiguity; use NUL separators/terminators or an escaping/length-prefix scheme instead.

**Normative resolution**: At `docs/issues/11-log-parser.md:21`, the canonical contract SHALL adopt the concrete requirement in this finding: For commits whose subject (or author name) contains \x1f , this parser will split a single record into more than four fields and skip or misparse it; Git emits that byte verbatim from %s (verified with a commit message $'hello\x1fthere' ). Because commit metadata is untrusted input elsewhere in this design, unit separator does not actually avoid ambiguity; use NUL separators/terminators or an escaping/length-prefix scheme instead.. Unsafe, malformed, or unsupported input must fail at the stated boundary without leaking data or creating an unintended side effect.

**Focused verification gate**: Run a safe control and each unsafe/boundary fixture named in the finding for `docs/issues/11-log-parser.md:21`; assert rejection/containment, no sensitive-data exposure, and no partial side effect.

**Completion boundary**: This entry records the design-level acceptance contract only. Do not resolve the GitHub thread until the focused result, applicable specialist review, and final repository validation are terminal success for the pinned identity.

## 21. Thread `PRRT_kwDOTN39T86OdgW0` — Allow summaries for editor-created commit messages

- File: `docs/issues/13-commit-command.md`
- Line: 26
- Finding basis: When the user runs the advertised editor flows (plain tracegit commit or -e ) without --summary , there is no assembled commit message before git commit runs, but this step requires validating the record, including non-empty summary , before invoking Git. Those normal commit modes will be rejected before the editor can provide a first line, so either require --summary for these modes in the CLI contract or derive and validate the summary after Git creates the commit.

**Normative resolution**: At `docs/issues/13-commit-command.md:26`, the canonical contract SHALL adopt the concrete requirement in this finding: When the user runs the advertised editor flows (plain tracegit commit or -e ) without --summary , there is no assembled commit message before git commit runs, but this step requires validating the record, including non-empty summary , before invoking Git. Those normal commit modes will be rejected before the editor can provide a first line, so either require --summary for these modes in the CLI contract or derive and validate the summary after Git creates the commit.. Unsafe, malformed, or unsupported input must fail at the stated boundary without leaking data or creating an unintended side effect.

**Focused verification gate**: Exercise normal, editor/hook/rewrite, and failure cases for `docs/issues/13-commit-command.md:26`; assert the result comes from the final canonical object/ref and recovery is actionable.

**Completion boundary**: This entry records the design-level acceptance contract only. Do not resolve the GitHub thread until the focused result, applicable specialist review, and final repository validation are terminal success for the pinned identity.

## 22. Thread `PRRT_kwDOTN39T86OdgW3` — Respect Git's editor precedence

- File: `docs/issues/14-annotate-command.md`
- Line: 24
- Finding basis: For --edit , this order does not match Git's editor resolution: the git var GIT EDITOR docs say the preference is $GIT EDITOR , then core.editor , then $VISUAL , then $EDITOR , and that the value is meant to be interpreted by the shell. In repos/users with core.editor or GIT EDITOR set, tracegit would open a different editor than Git and ignore VISUAL ; resolving via git var GIT EDITOR (or matching its order/semantics) avoids surprising edit failures.

**Normative resolution**: At `docs/issues/14-annotate-command.md:24`, the canonical contract SHALL adopt the concrete requirement in this finding: For --edit , this order does not match Git's editor resolution: the git var GIT EDITOR docs say the preference is $GIT EDITOR , then core.editor , then $VISUAL , then $EDITOR , and that the value is meant to be interpreted by the shell. In repos/users with core.editor or GIT EDITOR set, tracegit would open a different editor than Git and ignore VISUAL ; resolving via git var GIT EDITOR (or matching its order/semantics) avoids surprising edit failures.. Unsafe, malformed, or unsupported input must fail at the stated boundary without leaking data or creating an unintended side effect.

**Focused verification gate**: Exercise normal, editor/hook/rewrite, and failure cases for `docs/issues/14-annotate-command.md:24`; assert the result comes from the final canonical object/ref and recovery is actionable.

**Completion boundary**: This entry records the design-level acceptance contract only. Do not resolve the GitHub thread until the focused result, applicable specialist review, and final repository validation are terminal success for the pinned identity.

## 23. Thread `PRRT_kwDOTN39T86OdgW5` — Handle copied notes after amend

- File: `docs/issues/13-commit-command.md`
- Line: 29
- Finding basis: When users have configured note rewriting, an amended commit can already have a note before this call runs: the git-notes docs state that notes.rewrite.<command covers amend and notes.rewriteRef specifies refs copied during rewrite, and I verified git commit --amend copies refs/notes/tracegit when configured. In that scenario notes.Add(... force=false) returns ErrNoteExists after the commit succeeds, so tracegit commit --amend reports a failed note write instead of replacing the copied old rationale; overwrite for the freshly amended SHA.

**Normative resolution**: At `docs/issues/13-commit-command.md:29`, the canonical contract SHALL adopt the concrete requirement in this finding: When users have configured note rewriting, an amended commit can already have a note before this call runs: the git-notes docs state that notes.rewrite.<command covers amend and notes.rewriteRef specifies refs copied during rewrite, and I verified git commit --amend copies refs/notes/tracegit when configured. In that scenario notes.Add(... force=false) returns ErrNoteExists after the commit succeeds, so tracegit commit --amend reports a failed note write instead of replacing the copied old rationale; overwrite for the freshly amended SHA.. Unsafe, malformed, or unsupported input must fail at the stated boundary without leaking data or creating an unintended side effect.

**Focused verification gate**: Exercise normal, editor/hook/rewrite, and failure cases for `docs/issues/13-commit-command.md:29`; assert the result comes from the final canonical object/ref and recovery is actionable.

**Completion boundary**: This entry records the design-level acceptance contract only. Do not resolve the GitHub thread until the focused result, applicable specialist review, and final repository validation are terminal success for the pinned identity.

## 24. Thread `PRRT_kwDOTN39T86OdgW6` — Bound note output before buffering it

- File: `docs/issues/08-git-notes-wrapper.md`
- Line: 22
- Finding basis: For a hostile fetched note that is larger than the read ceiling, git notes show will be captured into memory by the runner before callers can apply ClampForRead , so the promised bounded read path can still allocate an attacker-sized blob. Add a max-output/bounded-reader path in Notes.Show or the runner and report truncation from there instead of returning an already fully buffered []byte .

**Normative resolution**: At `docs/issues/08-git-notes-wrapper.md:22`, the canonical contract SHALL adopt the concrete requirement in this finding: For a hostile fetched note that is larger than the read ceiling, git notes show will be captured into memory by the runner before callers can apply ClampForRead , so the promised bounded read path can still allocate an attacker-sized blob. Add a max-output/bounded-reader path in Notes.Show or the runner and report truncation from there instead of returning an already fully buffered []byte .. Failure must be bounded, deterministic, and leave no partial or stale state.

**Focused verification gate**: Run success, failure, interruption, deadline, and collision fixtures for `docs/issues/08-git-notes-wrapper.md:22`; assert the bounded result, rollback/cleanup, and absence of partial state.

**Completion boundary**: This entry records the design-level acceptance contract only. Do not resolve the GitHub thread until the focused result, applicable specialist review, and final repository validation are terminal success for the pinned identity.

## 25. Thread `PRRT_kwDOTN39T86OdgW7` — Bootstrap gitBinary before building the final runner

- File: `docs/issues/12-root-command.md`
- Line: 22
- Finding basis: This order cannot honor tracegit.gitBinary from git config: resolving config needs the runner per issue 05, but the runner is constructed first using the yet-unresolved GitBinary . In repos that set tracegit.gitBinary to a non-default Git, prechecks and all later commands will still use the bootstrap binary; resolve env/default first, read config, then rebuild or update the runner with the configured path.

**Normative resolution**: At `docs/issues/12-root-command.md:22`, the canonical contract SHALL adopt the concrete requirement in this finding: This order cannot honor tracegit.gitBinary from git config: resolving config needs the runner per issue 05, but the runner is constructed first using the yet-unresolved GitBinary . In repos that set tracegit.gitBinary to a non-default Git, prechecks and all later commands will still use the bootstrap binary; resolve env/default first, read config, then rebuild or update the runner with the configured path.. Unsafe, malformed, or unsupported input must fail at the stated boundary without leaking data or creating an unintended side effect.

**Focused verification gate**: Run the positive control and the exact negative/boundary case named in the finding for `docs/issues/12-root-command.md:22`; assert the canonical result and error behavior.

**Completion boundary**: This entry records the design-level acceptance contract only. Do not resolve the GitHub thread until the focused result, applicable specialist review, and final repository validation are terminal success for the pinned identity.

## 26. Thread `PRRT_kwDOTN39T86OdgW9` — Normalize short notes refs before syncing

- File: `docs/issues/05-config-resolution.md`
- Line: 30
- Finding basis: Allowing a bare notes ref such as tracegit here makes normal notes operations use Git's --ref namespace expansion ( refs/notes/tracegit ), but sync fetch/push later uses the raw value as a refspec ( tracegit:tracegit ), which does not fetch or push refs/notes/tracegit and fails against a remote containing only the notes ref. Normalize accepted short names to full refs/notes/<name before storing them in config, or require full refs/notes/... values.

**Normative resolution**: At `docs/issues/05-config-resolution.md:30`, the canonical contract SHALL adopt the concrete requirement in this finding: Allowing a bare notes ref such as tracegit here makes normal notes operations use Git's --ref namespace expansion ( refs/notes/tracegit ), but sync fetch/push later uses the raw value as a refspec ( tracegit:tracegit ), which does not fetch or push refs/notes/tracegit and fails against a remote containing only the notes ref. Normalize accepted short names to full refs/notes/<name before storing them in config, or require full refs/notes/... values.. Unsafe, malformed, or unsupported input must fail at the stated boundary without leaking data or creating an unintended side effect.

**Focused verification gate**: Exercise normal, editor/hook/rewrite, and failure cases for `docs/issues/05-config-resolution.md:30`; assert the result comes from the final canonical object/ref and recovery is actionable.

**Completion boundary**: This entry records the design-level acceptance contract only. Do not resolve the GitHub thread until the focused result, applicable specialist review, and final repository validation are terminal success for the pinned identity.

## 27. Thread `PRRT_kwDOTN39T86OdgW-` — Check C1 controls as runes, not UTF-8 bytes

- File: `docs/issues/06-output-sanitization.md`
- Line: 41
- Finding basis: The fuzz assertion cannot coexist with the acceptance criterion that emoji and other non-ASCII UTF-8 pass unchanged: printable characters commonly contain continuation bytes in 0x80 – 0x9f (for example 🚀 includes such bytes), so a byte-level forbidden-set check would either fail valid output or force the sanitizer to corrupt it. Treat C1 controls as decoded code points U+0080 – U+009F , not raw bytes inside UTF-8 sequences.

**Normative resolution**: At `docs/issues/06-output-sanitization.md:41`, the canonical contract SHALL adopt the concrete requirement in this finding: The fuzz assertion cannot coexist with the acceptance criterion that emoji and other non-ASCII UTF-8 pass unchanged: printable characters commonly contain continuation bytes in 0x80 – 0x9f (for example 🚀 includes such bytes), so a byte-level forbidden-set check would either fail valid output or force the sanitizer to corrupt it. Treat C1 controls as decoded code points U+0080 – U+009F , not raw bytes inside UTF-8 sequences.. Unsafe, malformed, or unsupported input must fail at the stated boundary without leaking data or creating an unintended side effect.

**Focused verification gate**: Run a safe control and each unsafe/boundary fixture named in the finding for `docs/issues/06-output-sanitization.md:41`; assert rejection/containment, no sensitive-data exposure, and no partial side effect.

**Completion boundary**: This entry records the design-level acceptance contract only. Do not resolve the GitHub thread until the focused result, applicable specialist review, and final repository validation are terminal success for the pinned identity.

## 28. Thread `PRRT_kwDOTN39T86OdgXC` — Cap writable rationale size below the read ceiling

- File: `docs/issues/04-record-validation-limits.md`
- Line: 18
- Finding basis: Because RationaleMaxBytes is user-overridable and config validation only requires a positive integer, a user can write a valid record larger than ReadHardCeilingBytes (for example with --max-rationale-bytes=2097152 ), after which every read path will truncate that same record at 1 MiB and degrade it. Either enforce an upper bound on the configured write limit or make the read ceiling at least as large as the maximum writable note size.

**Normative resolution**: At `docs/issues/04-record-validation-limits.md:18`, the canonical contract SHALL adopt the concrete requirement in this finding: Because RationaleMaxBytes is user-overridable and config validation only requires a positive integer, a user can write a valid record larger than ReadHardCeilingBytes (for example with --max-rationale-bytes=2097152 ), after which every read path will truncate that same record at 1 MiB and degrade it. Either enforce an upper bound on the configured write limit or make the read ceiling at least as large as the maximum writable note size.. The positive and invalid shapes must be deterministic and compatible with downstream consumers.

**Focused verification gate**: Validate canonical positive, missing, wrong-type, unknown-field, and boundary fixtures for `docs/issues/04-record-validation-limits.md:18`; assert the stated shape and failure behavior exactly.

**Completion boundary**: This entry records the design-level acceptance contract only. Do not resolve the GitHub thread until the focused result, applicable specialist review, and final repository validation are terminal success for the pinned identity.

## Specialist handoffs

- `tech-qa`/`tech-tester`: execute all focused gates and repository full-validation; missing, pending, failed, skipped, cancelled, timed-out, or stale evidence blocks closure.
- `tech-security`/`tech-devopssec`: review credentials, egress, spawning, paths, permissions, redaction, ref safety, output limits, and export controls; non-acceptance blocks closure.
- `gate-task-evaluator`: re-pin repository, PR, base/head, merge candidate, required checks, review decision, unresolved threads, policy version, and waiver immediately before merge.

No Bot trigger, review submission, or Bot rerun is authorized.