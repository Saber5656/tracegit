# 21 — Security acceptance test suite

## Summary
Encode the DESIGN §9.6 abuse cases as an executable, cross-cutting security test suite so
implementation agents cannot skip the security expectations.

## Context
DESIGN §9 defines the threat model and abuse cases. This issue makes them enforceable tests rather
than prose, and is a gate for v1 release.

## Scope
- A dedicated test package (e.g. `internal/security_test` / `test/security`) exercising the built
  CLI and packages end-to-end. No production code changes (may reveal gaps that spawn fix issues).

## Detailed Requirements
Cover each abuse case (DESIGN §9.6) as a named test:
1. **Command injection:** run `tracegit commit`/`annotate` with agent name, message, and tag
   values containing shell metacharacters (`; rm -rf /`, backticks, `$(...)`, newlines) and assert
   they are stored verbatim and no shell interpretation occurs (no file side effects; args reach
   git as-is). Confirms argv-only exec (B1).
2. **ANSI/OSC escape injection:** write notes whose `summary`/`rationale` contain
   `\x1b[2J`, `\x1b]0;pwned\x07`, BEL, and cursor-movement sequences; run `show`/`blame`/`log` in
   human mode and assert none of those bytes appear in stdout (B3, ADR-004).
3. **Size DoS:** attempt to write over-limit rationale (rejected, no commit); place an
   oversized raw note (> ceiling) and assert reads truncate + flag without unbounded allocation
   (B2, §9.3).
4. **Malformed / unknown-schema note:** write invalid JSON and a `schema: tracegit/v2` note; assert
   `show`/`blame`/`log`/`export` degrade gracefully (warning/`error` field), never crash, exit 0
   on reads.
5. **Malformed git output:** feed the blame/log parsers garbled fixtures (unit-level) and assert
   defensive handling, no panic (B4).
6. **Hostile fetched notes:** simulate fetching notes from a bare remote containing escape/oversized
   payloads (via a local bare remote) and assert reads apply the same sanitization/limits (B5).

## Acceptance Criteria
- All six categories have passing tests.
- No test triggers a panic or writes outside `t.TempDir()`.
- Escape bytes never reach human stdout; oversized inputs never allocate unboundedly.
- Any gap discovered is filed as a follow-up fix issue referencing the failing case.

## Validation
- `go test ./... -run Security -race` green in CI (wired in issue 22).

## Dependencies
Issues 13, 14, 15, 16, 17, 18 (and parsers 10/11).

## Non-goals
- Fixing newly discovered product bugs (handled as spawned issues).

## Design References
- DESIGN §9 (all), §9.6; ADR-004.
