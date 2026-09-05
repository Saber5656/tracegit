# 01 — Go module + repository scaffold

## Summary
Create the Go module, directory layout, build tooling, and baseline repo files so every later
issue has a place to put code and a consistent way to build/test/lint.

## Context
tracegit is a Go CLI (ADR-002) laid out per DESIGN §3.2. This issue establishes that skeleton
only; no product behavior yet.

## Scope
- `go.mod`, top-level dirs, `Makefile`, `.gitignore`, `LICENSE`, and a compiling entrypoint.
- No command logic beyond a stub that prints usage/version.

## Detailed Requirements
1. `go.mod` with module path `github.com/Saber5656/tracegit` and a current Go version directive
   (`go 1.22` or newer). Add `github.com/spf13/cobra` as the only non-stdlib dependency.
2. Create directories exactly as in DESIGN §3.2: `cmd/tracegit/`, `internal/cli/`, `internal/git/`,
   `internal/record/`, `internal/config/`, `internal/render/`, `internal/version/`. Add a
   `doc.go` or placeholder in each internal package so it compiles.
3. `cmd/tracegit/main.go`: thin entrypoint that calls `internal/cli.Execute()` and exits with its
   returned code. For this issue, `Execute()` may be a stub that prints `tracegit <version>` and
   returns 0.
4. `internal/version/version.go`: exported `var Version = "dev"` (overridable via
   `-ldflags "-X github.com/Saber5656/tracegit/internal/version.Version=..."`), plus a `String()`
   helper.
5. `Makefile` targets: `build` (`go build ./cmd/tracegit`), `test` (`go test ./...`),
   `vet` (`go vet ./...`), `fmt-check` (`gofmt -l .` fails if output non-empty), `lint`
   (invokes `golangci-lint run` if present, else prints a hint). All targets `.PHONY`.
6. `.gitignore` for Go (`/tracegit`, `/dist/`, `*.out`, editor dirs).
7. `LICENSE`: MIT, copyright holder `tracegit contributors`. (Confirm license per ADR-002 open
   item; MIT is the default.)
8. Keep the existing top-level `README.md` tagline; do not overwrite user-facing docs here
   (README expansion is issue 23).

## Acceptance Criteria
- `go build ./...` succeeds.
- `go vet ./...` passes.
- `make build` produces a `tracegit` binary that prints a version line and exits 0.
- Every directory in DESIGN §3.2 exists and compiles.
- Only `github.com/spf13/cobra` (+ its transitive deps) appears in `go.mod`/`go.sum`.

## Validation
- Run `go build ./... && go vet ./... && make build && ./tracegit`.
- `gofmt -l .` prints nothing.

## Dependencies
None.

## Non-goals
- No command implementations, flags, or git interaction (later issues).
- No CI config (issue 22).

## Design References
- DESIGN §3.2, §3.3; ADR-002.
