# 22 — OSS hardening, CI, and release

## Summary
Add continuous integration, supply-chain hardening, community health files, and a release
pipeline so tracegit can be safely developed in the open and shipped as binaries.

## Context
DESIGN §9.5, §12. The repo is intended to become public (see repository OSS hardening skill for
Rulesets/branch protection, applied separately by a maintainer).

## Scope
- `.github/` workflows and config; `SECURITY.md`, `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`;
  release config (e.g. `.goreleaser.yaml`). No product-code behavior changes.

## Detailed Requirements
1. **CI workflow** (`.github/workflows/ci.yml`), on PR + push to `main`:
   - `gofmt -l .` (fail if any output), `go vet ./...`, `golangci-lint run`,
     `go test -race ./...`, `govulncheck ./...`.
   - Build matrix: linux/amd64, linux/arm64, darwin/amd64, darwin/arm64, windows/amd64.
   - Pin action versions by SHA or major tag; set minimal `permissions:` (contents: read).
2. **CodeQL** (`.github/workflows/codeql.yml`) for Go, on PR + weekly schedule.
3. **Dependabot** (`.github/dependabot.yml`) for `gomod` and `github-actions`, weekly.
4. **Community files:** `SECURITY.md` (private disclosure contact + the "no secrets in rationale"
   guidance from DESIGN §9.4, notes-are-unauthenticated caveat from ADR-001), `CONTRIBUTING.md`
   (build/test/lint commands, DCO/sign-off policy TBD), `CODE_OF_CONDUCT.md`.
5. **Release** (`.github/workflows/release.yml` + `.goreleaser.yaml`): on tag `v*`, build the
   matrix, inject version via `-ldflags`, produce archives + `SHA256SUMS`. Release is a separate,
   human-gated step (merge ≠ release, DESIGN §12).
6. Enable (document, since some require repo-admin UI) secret scanning + push protection.

## Acceptance Criteria
- CI runs all listed checks and is green on the current tree.
- CodeQL and Dependabot configs are valid and scheduled.
- `SECURITY.md`, `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md` present.
- A dry-run of the release build (`goreleaser release --snapshot --clean` or equivalent) produces
  binaries for all target platforms with checksums.
- Workflow `permissions:` are minimal and action versions pinned.

## Validation
- Push the branch and confirm CI passes; run the release snapshot locally; `govulncheck` clean.

## Dependencies
Issue 01 (meaningful once 12+ commands exist; finalize late).

## Non-goals
- Branch protection / Rulesets (maintainer action via the OSS hardening skill).
- Publishing to package managers (v2).

## Design References
- DESIGN §9.5, §12; ADR-001 (open items), ADR-002 (license).
