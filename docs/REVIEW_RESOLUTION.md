# Review resolution addendum

- Repository: `Saber5656/tracegit`
- Pull request: #1
- Original PR head before this resolution addendum: `a65bc4a248bcd47d6e5700855f9372fead223245`
- The immutable current PR head is supplied by the parent task's fresh GitHub read immediately before review/reply/resolve; any later head change invalidates this evidence and requires a fresh review.
- Scope: each exact review thread below has a normative resolution and a focused verification gate.
- This is design-level handling only; it does not claim implementation, test, build, CI, or security validation is complete.
- Per task instruction, the PR review bot is not re-triggered after these responses/resolutions.

## 1. Thread `PRRT_kwDOTN39T86Odfx8` — Don't hard-code the license as MIT yet.

- File: `docs/DESIGN.md`
- Line: 128
- Finding summary: `Don't hard-code the license as MIT yet.`

**Normative resolution**: Revise the referenced release/CI contract so “Don't hard-code the license as MIT yet.” is explicit, reproducible, and gated before publication; keep unresolved owner decisions as gates rather than silently choosing a value.

**Focused verification gate**: Run the release/CI dry-run and its negative gate cases from a clean checkout; assert the required pin, branch/event guard, artifact, and publication precondition.

**Completion boundary**: this section records a design/acceptance contract for later implementation and repository full-validation gates. It does not claim implementation, test, build, CI, security, or release validation is complete.

## 2. Thread `PRRT_kwDOTN39T86Odfx9` — Resolve the pre-commit summary dependency.

- File: `docs/DESIGN.md`
- Line: 307
- Finding summary: `Resolve the pre-commit summary dependency.`

**Normative resolution**: Revise the dependency/critical-path contract so the prerequisite named by “Resolve the pre-commit summary dependency.” is explicit, ordered before every consumer, and included in the schedule/validation gate.

**Focused verification gate**: Verify the dependency table and execution order from a clean graph: the prerequisite is present, precedes every consumer, and the resulting critical path has no missing edge.

**Completion boundary**: this section records a design/acceptance contract for later implementation and repository full-validation gates. It does not claim implementation, test, build, CI, security, or release validation is complete.

## 3. Thread `PRRT_kwDOTN39T86Odfx-` — Do not hard-code world-readable export files.

- File: `docs/DESIGN.md`
- Line: 384
- Finding summary: `Do not hard-code world-readable export files.`

**Normative resolution**: Revise the referenced contract so it normatively enforces “Do not hard-code world-readable export files.”, including the affected positive path, the named negative/boundary path, and compatibility with the linked canonical sections.

**Focused verification gate**: Run a focused check for the named behavior with a normal case, the finding's boundary/negative case, and a regression case; assert the documented result and cross-reference consistency.

**Completion boundary**: this section records a design/acceptance contract for later implementation and repository full-validation gates. It does not claim implementation, test, build, CI, security, or release validation is complete.

## 4. Thread `PRRT_kwDOTN39T86Odfx_` — ' '

- File: `docs/issues/03-rationale-record-schema.md`
- Line: 40
- Finding summary: `' '`

**Normative resolution**: Revise the referenced contract so it normatively resolves “' '” and does not leave the reported ambiguity or failure mode to implementation choice.

**Focused verification gate**: Run a focused check for the named behavior with a normal case, the finding's boundary/negative case, and a regression case; assert the documented result and cross-reference consistency.

**Completion boundary**: this section records a design/acceptance contract for later implementation and repository full-validation gates. It does not claim implementation, test, build, CI, security, or release validation is complete.

## 5. Thread `PRRT_kwDOTN39T86OdfyA` — Make the fuzz check rune-aware.

- File: `docs/issues/06-output-sanitization.md`
- Line: 41
- Finding summary: `Make the fuzz check rune-aware.`

**Normative resolution**: Revise the referenced contract so it normatively enforces “Make the fuzz check rune-aware.”, including the affected positive path, the named negative/boundary path, and compatibility with the linked canonical sections.

**Focused verification gate**: Run a focused check for the named behavior with a normal case, the finding's boundary/negative case, and a regression case; assert the documented result and cross-reference consistency.

**Completion boundary**: this section records a design/acceptance contract for later implementation and repository full-validation gates. It does not claim implementation, test, build, CI, security, or release validation is complete.

## 6. Thread `PRRT_kwDOTN39T86OdfyC` — Add a commit-scoped lookup path for the `Show` fallback.

- File: `docs/issues/08-git-notes-wrapper.md`
- Line: 26
- Finding summary: `Add a commit-scoped lookup path for the `Show` fallback.`

**Normative resolution**: Revise the referenced contract so “Add a commit-scoped lookup path for the `Show` fallback.” is an explicit fail-closed security boundary: validate the input before use, preserve safe behavior, and reject or redact the unsafe case without leaking data or bypassing the stated policy.

**Focused verification gate**: Run focused positive and adversarial cases for the named boundary, including the unsafe input and a safe control; assert rejection/redaction, no side effect, and no sensitive data in output.

**Completion boundary**: this section records a design/acceptance contract for later implementation and repository full-validation gates. It does not claim implementation, test, build, CI, security, or release validation is complete.

## 7. Thread `PRRT_kwDOTN39T86OdfyD` — Expose parser warnings explicitly.

- File: `docs/issues/10-blame-porcelain-parser.md`
- Line: 31
- Finding summary: `Expose parser warnings explicitly.`

**Normative resolution**: Revise the referenced release/CI contract so “Expose parser warnings explicitly.” is explicit, reproducible, and gated before publication; keep unresolved owner decisions as gates rather than silently choosing a value.

**Focused verification gate**: Run the release/CI dry-run and its negative gate cases from a clean checkout; assert the required pin, branch/event guard, artifact, and publication precondition.

**Completion boundary**: this section records a design/acceptance contract for later implementation and repository full-validation gates. It does not claim implementation, test, build, CI, security, or release validation is complete.

## 8. Thread `PRRT_kwDOTN39T86OdfyF` — Avoid exact stderr matching for empty history.

- File: `docs/issues/11-log-parser.md`
- Line: 25
- Finding summary: `Avoid exact stderr matching for empty history.`

**Normative resolution**: Revise the referenced contract so it normatively enforces “Avoid exact stderr matching for empty history.”, including the affected positive path, the named negative/boundary path, and compatibility with the linked canonical sections.

**Focused verification gate**: Run a focused check for the named behavior with a normal case, the finding's boundary/negative case, and a regression case; assert the documented result and cross-reference consistency.

**Completion boundary**: this section records a design/acceptance contract for later implementation and repository full-validation gates. It does not claim implementation, test, build, CI, security, or release validation is complete.

## 9. Thread `PRRT_kwDOTN39T86OdfyG` — Derive `summary` from the final committed message.

- File: `docs/issues/13-commit-command.md`
- Line: 29
- Finding summary: `Derive `summary` from the final committed message.`

**Normative resolution**: Revise the referenced contract so it normatively enforces “Derive `summary` from the final committed message.”, including the affected positive path, the named negative/boundary path, and compatibility with the linked canonical sections.

**Focused verification gate**: Run a focused check for the named behavior with a normal case, the finding's boundary/negative case, and a regression case; assert the documented result and cross-reference consistency.

**Completion boundary**: this section records a design/acceptance contract for later implementation and repository full-validation gates. It does not claim implementation, test, build, CI, security, or release validation is complete.

## 10. Thread `PRRT_kwDOTN39T86OdfyI` — Make the recovery hint source-agnostic.

- File: `docs/issues/13-commit-command.md`
- Line: 32
- Finding summary: `Make the recovery hint source-agnostic.`

**Normative resolution**: Revise the referenced contract so it normatively enforces “Make the recovery hint source-agnostic.”, including the affected positive path, the named negative/boundary path, and compatibility with the linked canonical sections.

**Focused verification gate**: Run a focused check for the named behavior with a normal case, the finding's boundary/negative case, and a regression case; assert the documented result and cross-reference consistency.

**Completion boundary**: this section records a design/acceptance contract for later implementation and repository full-validation gates. It does not claim implementation, test, build, CI, security, or release validation is complete.

## 11. Thread `PRRT_kwDOTN39T86OdfyJ` — Cache the degraded state, not just the parsed record.

- File: `docs/issues/16-blame-command.md`
- Line: 27
- Finding summary: `Cache the degraded state, not just the parsed record.`

**Normative resolution**: Revise the referenced contract so it normatively resolves “Cache the degraded state, not just the parsed record.” and does not leave the reported ambiguity or failure mode to implementation choice.

**Focused verification gate**: Run a focused check for the named behavior with a normal case, the finding's boundary/negative case, and a regression case; assert the documented result and cross-reference consistency.

**Completion boundary**: this section records a design/acceptance contract for later implementation and repository full-validation gates. It does not claim implementation, test, build, CI, security, or release validation is complete.

## 12. Thread `PRRT_kwDOTN39T86OdfyK` — Keep the `rationale` JSON shape stable.

- File: `docs/issues/17-log-command.md`
- Line: 20
- Finding summary: `Keep the `rationale` JSON shape stable.`

**Normative resolution**: Revise the referenced contract so “Keep the `rationale` JSON shape stable.” is represented by one canonical, schema-valid input/output shape; reconcile all linked sections and define validation and failure behavior.

**Focused verification gate**: Parse/validate the canonical example and exercise valid, invalid, missing, and extra-field/shape cases; assert all consumers observe the same documented contract.

**Completion boundary**: this section records a design/acceptance contract for later implementation and repository full-validation gates. It does not claim implementation, test, build, CI, security, or release validation is complete.

## 13. Thread `PRRT_kwDOTN39T86OdfyN` — Drop `union` from the canonical sync path.

- File: `docs/issues/19-sync-command.md`
- Line: 22
- Finding summary: `Drop `union` from the canonical sync path.`

**Normative resolution**: Revise the referenced contract so “Drop `union` from the canonical sync path.” is an explicit fail-closed security boundary: validate the input before use, preserve safe behavior, and reject or redact the unsafe case without leaking data or bypassing the stated policy.

**Focused verification gate**: Run focused positive and adversarial cases for the named boundary, including the unsafe input and a safe control; assert rejection/redaction, no side effect, and no sensitive data in output.

**Completion boundary**: this section records a design/acceptance contract for later implementation and repository full-validation gates. It does not claim implementation, test, build, CI, security, or release validation is complete.

## 14. Thread `PRRT_kwDOTN39T86OdfyP` — Persist the resolved notes ref, not just the explicit flag.

- File: `docs/issues/20-init-command.md`
- Line: 17
- Finding summary: `Persist the resolved notes ref, not just the explicit flag.`

**Normative resolution**: Revise the referenced release/CI contract so “Persist the resolved notes ref, not just the explicit flag.” is explicit, reproducible, and gated before publication; keep unresolved owner decisions as gates rather than silently choosing a value.

**Focused verification gate**: Run the release/CI dry-run and its negative gate cases from a clean checkout; assert the required pin, branch/event guard, artifact, and publication precondition.

**Completion boundary**: this section records a design/acceptance contract for later implementation and repository full-validation gates. It does not claim implementation, test, build, CI, security, or release validation is complete.

## 15. Thread `PRRT_kwDOTN39T86OdfyQ` — /*.svg' -g '!

- File: `docs/issues/22-oss-hardening-ci-release.md`
- Line: 29
- Finding summary: `/*.svg' -g '!`

**Normative resolution**: Revise the referenced contract so it normatively resolves “/*.svg' -g '!” and does not leave the reported ambiguity or failure mode to implementation choice.

**Focused verification gate**: Run a focused check for the named behavior with a normal case, the finding's boundary/negative case, and a regression case; assert the documented result and cross-reference consistency.

**Completion boundary**: this section records a design/acceptance contract for later implementation and repository full-validation gates. It does not claim implementation, test, build, CI, security, or release validation is complete.

## 16. Thread `PRRT_kwDOTN39T86OdfyS` — Include `annotate` in the E2E lifecycle.

- File: `docs/issues/23-e2e-tests-and-user-docs.md`
- Line: 19
- Finding summary: `Include `annotate` in the E2E lifecycle.`

**Normative resolution**: Revise the referenced contract so it normatively enforces “Include `annotate` in the E2E lifecycle.”, including the affected positive path, the named negative/boundary path, and compatibility with the linked canonical sections.

**Focused verification gate**: Run a focused check for the named behavior with a normal case, the finding's boundary/negative case, and a regression case; assert the documented result and cross-reference consistency.

**Completion boundary**: this section records a design/acceptance contract for later implementation and repository full-validation gates. It does not claim implementation, test, build, CI, security, or release validation is complete.

## 17. Thread `PRRT_kwDOTN39T86OdfyV` — Remove the unsupported `cat_sort_uniq` strategy.

- File: `docs/research/git-notes-and-blame.md`
- Line: 55
- Finding summary: `Remove the unsupported `cat_sort_uniq` strategy.`

**Normative resolution**: Revise the referenced contract so it normatively enforces “Remove the unsupported `cat_sort_uniq` strategy.”, including the affected positive path, the named negative/boundary path, and compatibility with the linked canonical sections.

**Focused verification gate**: Run a focused check for the named behavior with a normal case, the finding's boundary/negative case, and a regression case; assert the documented result and cross-reference consistency.

**Completion boundary**: this section records a design/acceptance contract for later implementation and repository full-validation gates. It does not claim implementation, test, build, CI, security, or release validation is complete.

## 18. Thread `PRRT_kwDOTN39T86OdgWt` — Avoid installing notes as the remote's default push

- File: `docs/issues/19-sync-command.md`
- Line: 16
- Finding summary: `Avoid installing notes as the remote's default push`

**Normative resolution**: Revise the referenced release/CI contract so “Avoid installing notes as the remote's default push” is explicit, reproducible, and gated before publication; keep unresolved owner decisions as gates rather than silently choosing a value.

**Focused verification gate**: Run the release/CI dry-run and its negative gate cases from a clean checkout; assert the required pin, branch/event guard, artifact, and publication precondition.

**Completion boundary**: this section records a design/acceptance contract for later implementation and repository full-validation gates. It does not claim implementation, test, build, CI, security, or release validation is complete.

## 19. Thread `PRRT_kwDOTN39T86OdgWw` — Avoid force-fetching over the canonical notes ref

- File: `docs/issues/19-sync-command.md`
- Line: 17
- Finding summary: `Avoid force-fetching over the canonical notes ref`

**Normative resolution**: Revise the referenced contract so it normatively enforces “Avoid force-fetching over the canonical notes ref”, including the affected positive path, the named negative/boundary path, and compatibility with the linked canonical sections.

**Focused verification gate**: Run a focused check for the named behavior with a normal case, the finding's boundary/negative case, and a regression case; assert the documented result and cross-reference consistency.

**Completion boundary**: this section records a design/acceptance contract for later implementation and repository full-validation gates. It does not claim implementation, test, build, CI, security, or release validation is complete.

## 20. Thread `PRRT_kwDOTN39T86OdgWz` — Use NUL-delimited log output

- File: `docs/issues/11-log-parser.md`
- Line: 21
- Finding summary: `Use NUL-delimited log output`

**Normative resolution**: Revise the referenced contract so “Use NUL-delimited log output” is a normative boundedness/atomicity guarantee with one defined deadline or commit point, deterministic failure behavior, and cleanup of partial state.

**Focused verification gate**: Run focused success, timeout/limit, concurrent/partial-failure cases; assert the documented deadline/cap, deterministic error, cleanup, and no clobber or leaked state.

**Completion boundary**: this section records a design/acceptance contract for later implementation and repository full-validation gates. It does not claim implementation, test, build, CI, security, or release validation is complete.

## 21. Thread `PRRT_kwDOTN39T86OdgW0` — Allow summaries for editor-created commit messages

- File: `docs/issues/13-commit-command.md`
- Line: 26
- Finding summary: `Allow summaries for editor-created commit messages`

**Normative resolution**: Revise the referenced contract so it normatively enforces “Allow summaries for editor-created commit messages”, including the affected positive path, the named negative/boundary path, and compatibility with the linked canonical sections.

**Focused verification gate**: Run a focused check for the named behavior with a normal case, the finding's boundary/negative case, and a regression case; assert the documented result and cross-reference consistency.

**Completion boundary**: this section records a design/acceptance contract for later implementation and repository full-validation gates. It does not claim implementation, test, build, CI, security, or release validation is complete.

## 22. Thread `PRRT_kwDOTN39T86OdgW3` — Respect Git's editor precedence

- File: `docs/issues/14-annotate-command.md`
- Line: 24
- Finding summary: `Respect Git's editor precedence`

**Normative resolution**: Revise the referenced contract so it normatively resolves “Respect Git's editor precedence” and does not leave the reported ambiguity or failure mode to implementation choice.

**Focused verification gate**: Run a focused check for the named behavior with a normal case, the finding's boundary/negative case, and a regression case; assert the documented result and cross-reference consistency.

**Completion boundary**: this section records a design/acceptance contract for later implementation and repository full-validation gates. It does not claim implementation, test, build, CI, security, or release validation is complete.

## 23. Thread `PRRT_kwDOTN39T86OdgW5` — Handle copied notes after amend

- File: `docs/issues/13-commit-command.md`
- Line: 29
- Finding summary: `Handle copied notes after amend`

**Normative resolution**: Revise the referenced contract so it normatively enforces “Handle copied notes after amend”, including the affected positive path, the named negative/boundary path, and compatibility with the linked canonical sections.

**Focused verification gate**: Run a focused check for the named behavior with a normal case, the finding's boundary/negative case, and a regression case; assert the documented result and cross-reference consistency.

**Completion boundary**: this section records a design/acceptance contract for later implementation and repository full-validation gates. It does not claim implementation, test, build, CI, security, or release validation is complete.

## 24. Thread `PRRT_kwDOTN39T86OdgW6` — Bound note output before buffering it

- File: `docs/issues/08-git-notes-wrapper.md`
- Line: 22
- Finding summary: `Bound note output before buffering it`

**Normative resolution**: Revise the referenced contract so it normatively resolves “Bound note output before buffering it” and does not leave the reported ambiguity or failure mode to implementation choice.

**Focused verification gate**: Run a focused check for the named behavior with a normal case, the finding's boundary/negative case, and a regression case; assert the documented result and cross-reference consistency.

**Completion boundary**: this section records a design/acceptance contract for later implementation and repository full-validation gates. It does not claim implementation, test, build, CI, security, or release validation is complete.

## 25. Thread `PRRT_kwDOTN39T86OdgW7` — Bootstrap gitBinary before building the final runner

- File: `docs/issues/12-root-command.md`
- Line: 22
- Finding summary: `Bootstrap gitBinary before building the final runner`

**Normative resolution**: Revise the referenced release/CI contract so “Bootstrap gitBinary before building the final runner” is explicit, reproducible, and gated before publication; keep unresolved owner decisions as gates rather than silently choosing a value.

**Focused verification gate**: Run the release/CI dry-run and its negative gate cases from a clean checkout; assert the required pin, branch/event guard, artifact, and publication precondition.

**Completion boundary**: this section records a design/acceptance contract for later implementation and repository full-validation gates. It does not claim implementation, test, build, CI, security, or release validation is complete.

## 26. Thread `PRRT_kwDOTN39T86OdgW9` — Normalize short notes refs before syncing

- File: `docs/issues/05-config-resolution.md`
- Line: 30
- Finding summary: `Normalize short notes refs before syncing`

**Normative resolution**: Revise the referenced release/CI contract so “Normalize short notes refs before syncing” is explicit, reproducible, and gated before publication; keep unresolved owner decisions as gates rather than silently choosing a value.

**Focused verification gate**: Run the release/CI dry-run and its negative gate cases from a clean checkout; assert the required pin, branch/event guard, artifact, and publication precondition.

**Completion boundary**: this section records a design/acceptance contract for later implementation and repository full-validation gates. It does not claim implementation, test, build, CI, security, or release validation is complete.

## 27. Thread `PRRT_kwDOTN39T86OdgW-` — Check C1 controls as runes, not UTF-8 bytes

- File: `docs/issues/06-output-sanitization.md`
- Line: 41
- Finding summary: `Check C1 controls as runes, not UTF-8 bytes`

**Normative resolution**: Revise the referenced contract so it normatively resolves “Check C1 controls as runes, not UTF-8 bytes” and does not leave the reported ambiguity or failure mode to implementation choice.

**Focused verification gate**: Run a focused check for the named behavior with a normal case, the finding's boundary/negative case, and a regression case; assert the documented result and cross-reference consistency.

**Completion boundary**: this section records a design/acceptance contract for later implementation and repository full-validation gates. It does not claim implementation, test, build, CI, security, or release validation is complete.

## 28. Thread `PRRT_kwDOTN39T86OdgXC` — Cap writable rationale size below the read ceiling

- File: `docs/issues/04-record-validation-limits.md`
- Line: 18
- Finding summary: `Cap writable rationale size below the read ceiling`

**Normative resolution**: Revise the referenced contract so “Cap writable rationale size below the read ceiling” is a normative boundedness/atomicity guarantee with one defined deadline or commit point, deterministic failure behavior, and cleanup of partial state.

**Focused verification gate**: Run focused success, timeout/limit, concurrent/partial-failure cases; assert the documented deadline/cap, deterministic error, cleanup, and no clobber or leaked state.

**Completion boundary**: this section records a design/acceptance contract for later implementation and repository full-validation gates. It does not claim implementation, test, build, CI, security, or release validation is complete.
