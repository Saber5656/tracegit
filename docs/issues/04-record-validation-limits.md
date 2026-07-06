# 04 — Record validation + size limits

## Summary
Validate a `Record` against the field rules and enforce size limits, so oversized or malformed
rationale is rejected **before** anything is written to git notes.

## Context
DESIGN §4.1 (field rules) and §9.3 (size limits), plus threat B2 (size DoS / malformed write).
Validation must run before the commit in `tracegit commit` (fail early, write nothing).

## Scope
- `internal/record/validate.go` and `internal/record/limits.go`.

## Detailed Requirements
1. `Limits` struct with defaults: `SummaryMaxBytes=1024`, `RationaleMaxBytes=65536` (overridable),
   `TagMaxCount=16`, `TagMaxBytes=64`, `AgentNameMaxBytes=128`, `ModelMaxBytes=128`,
   `AgentVersionMaxBytes=64`, `TaskIDMaxBytes=128`, `TaskURLMaxBytes=2048`,
   `ReadHardCeilingBytes=1<<20` (1 MiB). `RationaleMaxBytes` comes from config (issue 05).
2. `ValidateForWrite(r Record, lim Limits) error` (aggregates all violations into one error):
   - `schema` must equal `tracegit/v1`.
   - `summary` non-empty after trimming whitespace; ≤ `SummaryMaxBytes`.
   - `rationale` ≤ `RationaleMaxBytes`.
   - `confidence` empty or one of `low|medium|high`.
   - `tags`: ≤ `TagMaxCount`; each non-empty, ≤ `TagMaxBytes`, containing no control characters.
   - `agent.*`, `task.id` within their byte limits.
   - `task.url` (if set): parses as an absolute `http`/`https` URL; ≤ `TaskURLMaxBytes`.
   - Byte limits measured on UTF-8 length; reject invalid UTF-8 in any string field.
3. `ClampForRead(data []byte, lim Limits) (body []byte, truncated bool)`: if a note body read from
   git exceeds `min(4*RationaleMaxBytes, ReadHardCeilingBytes)`, truncate for display and report
   `truncated=true`. Never allocate unbounded.
4. Validation errors must be actionable (name the field and the limit).

## Acceptance Criteria
- Empty/whitespace-only summary rejected; over-limit summary/rationale rejected.
- `>16` tags, an over-long tag, or a tag with a control char rejected.
- Non-http(s) or unparseable `task.url` rejected; valid https URL accepted.
- Invalid UTF-8 rejected.
- `ClampForRead` truncates beyond the ceiling and flags it; leaves small inputs untouched.
- All violations in one record are reported together, not one-at-a-time.

## Validation
- Table-driven tests for each rule (pass + fail cases), including boundary sizes (exactly at
  limit vs one byte over).

## Dependencies
Issue 03. (Limit value for rationale is wired from issue 05 config at call sites.)

## Non-goals
- Reading/writing notes (issue 08).
- Sanitizing for display (issue 06).

## Design References
- DESIGN §4.1, §9.3; threat B2.
