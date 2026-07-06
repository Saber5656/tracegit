# tracegit — Issue Plan (v1)

> Canonical execution plan. GitHub Issues are derived from `docs/issues/NN-*.md`. If they
> disagree, this tree wins and the Issues are stale.

## v1 completion statement

When every issue below (01–23) is implemented and validated, tracegit v1 is complete: a Go CLI
that captures structured rationale at commit time (`tracegit commit`) and retroactively
(`tracegit annotate`), stores it in `refs/notes/tracegit` keyed by commit SHA, surfaces it via
`tracegit blame|show|log`, moves it with `tracegit sync`, exports it with `tracegit export`, is
configured via flags/env/git-config, renders untrusted content safely, enforces size limits, and
ships with tests, CI, OSS hardening, and user docs — matching [DESIGN.md](./DESIGN.md). Anything
outside this set is a documented v2/deferred item or a known unknown below.

## Issue list (recommended execution order)

| # | File | Title | Wave |
|---|---|---|---|
| 01 | [01-project-scaffold.md](./issues/01-project-scaffold.md) | Go module + repository scaffold | 0 |
| 02 | [02-git-exec-runner.md](./issues/02-git-exec-runner.md) | git exec runner (argv, discovery, version gate) | 0 |
| 03 | [03-rationale-record-schema.md](./issues/03-rationale-record-schema.md) | Rationale record schema + serialization | 0 |
| 04 | [04-record-validation-limits.md](./issues/04-record-validation-limits.md) | Record validation + size limits | 0 |
| 05 | [05-config-resolution.md](./issues/05-config-resolution.md) | Config resolution (flags/env/git-config) | 0 |
| 06 | [06-output-sanitization.md](./issues/06-output-sanitization.md) | Terminal-output sanitization | 0 |
| 07 | [07-render-package.md](./issues/07-render-package.md) | Render package (human + JSON helpers) | 0 |
| 08 | [08-git-notes-wrapper.md](./issues/08-git-notes-wrapper.md) | Git notes read/write/list wrapper | 1 |
| 09 | [09-git-commit-helper.md](./issues/09-git-commit-helper.md) | Commit + rev-parse helper | 1 |
| 10 | [10-blame-porcelain-parser.md](./issues/10-blame-porcelain-parser.md) | `blame --porcelain` parser | 1 |
| 11 | [11-log-parser.md](./issues/11-log-parser.md) | `git log` delimited parser | 1 |
| 12 | [12-root-command.md](./issues/12-root-command.md) | Root command + global flags + version | 2 |
| 13 | [13-commit-command.md](./issues/13-commit-command.md) | `tracegit commit` command | 2 |
| 14 | [14-annotate-command.md](./issues/14-annotate-command.md) | `tracegit annotate` command | 2 |
| 15 | [15-show-command.md](./issues/15-show-command.md) | `tracegit show` command | 3 |
| 16 | [16-blame-command.md](./issues/16-blame-command.md) | `tracegit blame` command | 3 |
| 17 | [17-log-command.md](./issues/17-log-command.md) | `tracegit log` command | 3 |
| 18 | [18-export-command.md](./issues/18-export-command.md) | `tracegit export` command | 3 |
| 19 | [19-sync-command.md](./issues/19-sync-command.md) | `tracegit sync` command | 4 |
| 20 | [20-init-command.md](./issues/20-init-command.md) | `tracegit init` command | 4 |
| 21 | [21-security-acceptance-tests.md](./issues/21-security-acceptance-tests.md) | Security acceptance test suite | 5 |
| 22 | [22-oss-hardening-ci-release.md](./issues/22-oss-hardening-ci-release.md) | OSS hardening, CI, release | 5 |
| 23 | [23-e2e-tests-and-user-docs.md](./issues/23-e2e-tests-and-user-docs.md) | End-to-end tests + user docs | 5 |

## Dependency table

| Issue | Depends on |
|---|---|
| 01 | — |
| 02 | 01 |
| 03 | 01 |
| 04 | 03 |
| 05 | 01, 02 |
| 06 | 01 |
| 07 | 03, 06 |
| 08 | 02 |
| 09 | 02 |
| 10 | 02 |
| 11 | 02 |
| 12 | 02, 05 |
| 13 | 12, 09, 08, 04, 05 |
| 14 | 12, 08, 04, 05 |
| 15 | 12, 08, 07 |
| 16 | 12, 10, 08, 07 |
| 17 | 12, 11, 08, 07 |
| 18 | 12, 08, 03 |
| 19 | 12, 02, 05 |
| 20 | 12, 05, 19 |
| 21 | 13, 14, 15, 16, 17, 18 |
| 22 | 01 (meaningful once 12+ land) |
| 23 | 13–20 |

### Dependency graph (textual)

```
01 ─┬─ 02 ─┬─ 05 ─┐
    │      ├─ 08 ─┤
    │      ├─ 09 ─┤
    │      ├─ 10 ─┤
    │      └─ 11 ─┤
    ├─ 03 ─┬─ 04 ─┤
    │      └─ 07 ─┤ (also needs 06)
    └─ 06 ────────┤
                  ▼
                 12 (root) ─┬─ 13 commit ─┐
                            ├─ 14 annotate┤
                            ├─ 15 show    ┤
                            ├─ 16 blame   ┤
                            ├─ 17 log     ┤
                            ├─ 18 export  ┤
                            ├─ 19 sync ── 20 init
                            └─────────────┤
                                          ▼
                              21 security · 23 e2e/docs
                              22 CI/hardening (parallelizable after 01)
```

## Implementation waves

- **Wave 0 — Foundation (01–07):** no end-user behavior; pure packages with unit tests. Can be
  parallelized across agents (01 first, then 02–07). Establishes exec safety, schema, config,
  sanitization, rendering.
- **Wave 1 — Git primitives (08–11):** thin, well-tested wrappers/parsers over `git`. Parallel.
- **Wave 2 — Root + capture (12–14):** the write path becomes usable.
- **Wave 3 — Read/display (15–18):** the read path becomes usable. Parallel after 12 + parsers.
- **Wave 4 — Data movement (19–20):** sync + init.
- **Wave 5 — Hardening (21–23):** security acceptance tests, OSS/CI/release, e2e + docs. 22 can
  begin as soon as 01 exists and is finalized at the end.

## Coverage table (DESIGN.md → issues)

| DESIGN section | Covered by |
|---|---|
| §3.2 Package layout | 01, and each package issue |
| §3.3 Dependencies | 01, 22 |
| §4.1 Record schema | 03, 04 |
| §4.2 Canonical JSON encoding | 03 |
| §4.3 Notes storage | 08 |
| §5 Configuration | 05, 12 |
| §6.1 `commit` | 13 |
| §6.2 `annotate` | 14 |
| §6.3 `show` | 15 |
| §6.4 `blame` | 16 (parser 10) |
| §6.5 `log` | 17 (parser 11) |
| §6.6 `export` | 18 |
| §6.7 `sync` | 19 |
| §6.8 `init` | 20 |
| §6.9 `version` | 12 |
| §7.1 Exec model | 02 |
| §7.2 Blame parsing | 10 |
| §7.3 Log parsing | 11 |
| §7.4 Notes read/write/list | 08 |
| §8 Rendering & output | 06, 07 |
| §9 Security model | 02 (B1/B6), 04 (B2), 06 (B3), 08 (B3/B5), 10/11 (B4), 19 (B5), 21 (all), 22 (§9.5) |
| §10 Failure modes | 09, 13, 14, 15, and per-command issues |
| §11 Testing strategy | 21, 23, and unit tests in every issue |
| §12 Release & distribution | 22 |

## Validation strategy (whole product)

1. **Per-package unit tests** land with each Wave 0/1 issue (record round-trip incl. unknown-field
   preservation, sanitizer table cases, config precedence, blame/log parser fixtures incl.
   malformed inputs).
2. **Command tests** for each CLI issue using ephemeral repos (`t.TempDir()`) exercising the happy
   path + the failure modes in DESIGN §10.
3. **Security acceptance suite (21)** covering DESIGN §9.6 abuse cases as executable tests.
4. **End-to-end suite (23)**: full lifecycle `init → commit → show → blame → log → export → sync`
   against a local bare remote.
5. **CI (22)**: `gofmt -l` (fail on diff), `go vet`, `golangci-lint`, `go test -race ./...`,
   `govulncheck`, build matrix, CodeQL, Dependabot.
6. **Definition of done per issue:** its Acceptance Criteria and Validation sections pass locally
   and in CI; no unrelated files touched.

## Deferred (v2) items

Mirror of DESIGN §13: hook-based capture; trailer ingestion; `tracegit search`; TUI blame;
editor/IDE integration; signed/verifiable notes; secret scanning/redaction; advanced notes
conflict auto-merge; `--raw` render escape hatch; transactional note-first commit mode.

## Known unknowns (may spawn new issues during implementation)

1. **Blame porcelain edge cases:** exact behavior with `-C`/`-M`, boundary/root commits, and
   binary/empty files may require extra parser fixtures → possible follow-up issue.
2. **"No note" detection:** exact `git notes show` stderr/exit contract across git versions;
   fallback detection may need tuning (see research doc).
3. **Notes merge on concurrent writes:** real-world conflict UX may justify more than the `manual`
   default before v1 GA.
4. **`--amend` note migration:** whether to auto-`git notes copy` from the pre-amend SHA.
5. **Windows console encoding:** display of non-ASCII rationale in `cmd.exe`/PowerShell may need
   explicit UTF-8 handling.
6. **`golangci-lint` ruleset:** which linters to enable without excessive churn.
