# 03 — Rationale record schema + serialization

## Summary
Define the `tracegit/v1` rationale record Go type and its canonical JSON (de)serialization,
including forward-compatible preservation of unknown fields.

## Context
DESIGN §4.1–§4.2 specify the record schema, field rules, canonical encoding, and forward-compat
policy. This is the data heart of the product; the note body is exactly this JSON.

## Scope
- `internal/record/record.go`: type + marshal/unmarshal + canonical encoding. Validation and
  limits are issue 04.

## Detailed Requirements
1. Define the `Record` struct with fields and JSON tags matching DESIGN §4.1 exactly:
   `schema, summary, rationale, agent{name,model,version}, task{id,url}, confidence, tags[],
   created_at, tracegit_version`. Use `omitempty` for all optional fields and nested empty
   structs.
2. Constant `SchemaV1 = "tracegit/v1"`.
3. `Marshal(r Record) ([]byte, error)`: emit canonical JSON per DESIGN §4.2 — UTF-8, fixed key
   order (schema, summary, rationale, agent, task, confidence, tags, created_at,
   tracegit_version), two-space indent, trailing newline. (Since `encoding/json` orders by struct
   field order, define struct fields in that order; verify output stability with a golden test.)
4. `Unmarshal(data []byte) (Record, error)`:
   - Strict-ish decode. Preserve unknown **top-level** fields into a `raw map[string]json.RawMessage`
     (or an `Extra` field) so a round-trip does not drop future fields.
   - On re-marshal, merge preserved unknown fields back in (known fields take precedence).
5. `IsKnownSchema(r) bool`: true iff `schema == "tracegit/v1"`. Records failing this are handled in
   "degraded" mode by callers (do not error here).
6. Helpers: `NewFromInputs(...)` builder used by commit/annotate that sets `schema`, `created_at`
   (RFC3339 UTC; caller injects a clock for testability), and `tracegit_version`.
7. Do not sanitize here (rendering concern, issue 06); do not enforce limits here (issue 04).

## Acceptance Criteria
- Round-trip: `Unmarshal(Marshal(r)) == r` for representative records (including all optional
  fields set and none set).
- Unknown top-level field present in input JSON survives a `Unmarshal → Marshal` round-trip.
- Canonical output is byte-stable across runs (golden file) with the documented key order and
  trailing newline.
- `NewFromInputs` sets `schema=tracegit/v1`, a UTC RFC3339 `created_at` from the injected clock,
  and the current `tracegit_version`.

## Validation
- Table-driven marshal/unmarshal tests + a golden-file test for canonical bytes.
- Unknown-field preservation test.

## Dependencies
Issue 01.

## Non-goals
- Field validation and size limits (issue 04).
- Terminal sanitization (issue 06).

## Design References
- DESIGN §4.1, §4.2; ADR-001.
