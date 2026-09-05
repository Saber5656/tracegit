# tracegit — Design (v1)

> **Status:** Draft for implementation
> **Audience:** Implementation agents and human reviewers
> **Canonical source of truth:** this file and the rest of `docs/`. GitHub Issues and PRs are derived artifacts.

`tracegit` is a **reasoning-aware `git blame`**. It captures the *summary and rationale* behind
each commit — especially commits authored by AI coding agents — and lets you retrieve that
"why" later through blame-, show-, and log-style views.

Standard `git blame` answers *who changed this line and when*. `tracegit` additionally answers
*why it was changed and what the author was reasoning about*, by attaching structured rationale
records to commits and surfacing them line-by-line.

---

## 1. Product overview

### 1.1 Problem

AI coding agents produce a high volume of commits. The reasoning behind each change (what the
agent was told, what alternatives it weighed, what it was uncertain about) is usually lost the
moment the session ends. Commit messages capture *what* but rarely the durable *why*, and they
cannot be enriched after the fact without rewriting history.

### 1.2 Solution

A small, dependency-light Go CLI that:

1. **Captures** a structured rationale record at commit time (`tracegit commit`) or retroactively
   (`tracegit annotate`).
2. **Stores** it in git notes (`refs/notes/tracegit`), keyed by the commit SHA — without
   rewriting history.
3. **Surfaces** it through `tracegit blame`, `tracegit show`, and `tracegit log`.
4. **Moves** it between clones with explicit `tracegit sync` helpers, and **exports** it for
   analysis with `tracegit export`.

### 1.3 Design principles

- **Git-native.** Rationale lives in git notes keyed by SHA. No external database, no service.
- **Non-destructive.** Never rewrite commit history. Notes attach to existing commits.
- **Shell out to `git`.** Do not embed a git implementation. Invoke the `git` binary with an
  explicit argument vector (never through a shell). This keeps the tool small and behavior
  identical to the user's git.
- **Secure by default.** No implicit network access. All rendered rationale is untrusted input
  and is sanitized before hitting a terminal. Size limits are enforced on read and write.
- **Degrade gracefully.** A missing, malformed, or oversized note must never crash a read command
  and must never block the underlying git operation.
- **Small, composable commands.** Each command does one thing and offers a stable `--json` mode.

---

## 2. Scope

### 2.1 v1 scope (this design)

| Area | In v1 |
|---|---|
| Capture | `tracegit commit` (wrapper), `tracegit annotate` (retroactive) |
| Read | `tracegit blame`, `tracegit show`, `tracegit log` |
| Data movement | `tracegit sync fetch/push/setup`, `tracegit export` |
| Setup | `tracegit init`, `tracegit version` |
| Storage | Git notes at `refs/notes/tracegit`, JSON record schema `tracegit/v1` |
| Config | Flags > env vars > `git config tracegit.*` > defaults |
| Output | Human (TTY-aware) and `--json` for all read commands |
| Security | Argv-only exec, control-char sanitization, size limits, no auto-network |

### 2.2 v1 non-goals

- No interactive TUI.
- No editor/IDE integration.
- No rationale full-text search command (`tracegit export` + external tools cover this in v1).
- No automatic capture via git hooks.
- No ingestion of rationale from commit-message trailers.
- No cryptographic signing/verification of notes.
- No web UI or hosted service.
- No multi-notes-ref federation or per-branch notes.

### 2.3 v2 / deferred (documented, not built)

See [§13 Deferred](#13-deferred-v2-ideas). Highlights: hook-based capture, trailer ingestion,
`tracegit search`, TUI blame, editor integration, signed notes, and rationale redaction/secret
scanning.

### 2.4 Known unknowns

Tracked in [ISSUE_PLAN.md](./ISSUE_PLAN.md#known-unknowns). Notable: exact `git blame --porcelain`
edge cases (boundary commits, `-C`/`-M` rename detection), git notes merge behavior on
concurrent writes, and Windows path/exec quirks.

---

## 3. Architecture

### 3.1 Component diagram

```
                    ┌──────────────────────────────────────────┐
                    │                 CLI (cobra)                │
                    │  root · commit · annotate · blame · show   │
                    │  log · export · sync · init · version      │
                    └───────────────┬────────────────────────────┘
                                    │ uses
        ┌───────────────┬───────────┼─────────────┬───────────────┐
        ▼               ▼           ▼             ▼               ▼
  ┌───────────┐  ┌────────────┐ ┌─────────┐  ┌──────────┐  ┌────────────┐
  │  config   │  │  record    │ │ render  │  │  git     │  │  version   │
  │ (resolve) │  │ (schema +  │ │(human/  │  │ (exec    │  │            │
  │           │  │  validate) │ │ json +  │  │  wrapper)│  │            │
  │           │  │            │ │sanitize)│  │          │  │            │
  └───────────┘  └────────────┘ └─────────┘  └────┬─────┘  └────────────┘
                                                   │ argv (no shell)
                                                   ▼
                                            ┌─────────────┐
                                            │  git binary │
                                            └──────┬──────┘
                                                   ▼
                                    refs/notes/tracegit  (per-commit JSON)
```

### 3.2 Package layout (Go)

```
tracegit/
├── go.mod                        # module: github.com/Saber5656/tracegit
├── Makefile                      # build, test, lint, vet targets
├── LICENSE                       # MIT (see ADR-002 open item)
├── README.md
├── .gitignore
├── cmd/
│   └── tracegit/
│       └── main.go               # thin entrypoint → internal/cli.Execute()
├── internal/
│   ├── cli/                      # one file per command
│   │   ├── root.go               # root cmd, global flags, git precheck
│   │   ├── commit.go
│   │   ├── annotate.go
│   │   ├── blame.go
│   │   ├── show.go
│   │   ├── log.go
│   │   ├── export.go
│   │   ├── sync.go
│   │   ├── init.go
│   │   └── version.go
│   ├── git/                      # git process wrapper — the ONLY package that execs git
│   │   ├── runner.go             # exec.Command argv helper, git discovery + version gate
│   │   ├── notes.go              # notes add/show/list/remove on a configurable ref
│   │   ├── commit.go             # commit + rev-parse
│   │   ├── blame.go              # `blame --porcelain` parser
│   │   └── log.go                # `log` parser
│   ├── record/                   # rationale record: schema, serde, validation, limits
│   │   ├── record.go
│   │   ├── validate.go
│   │   └── limits.go
│   ├── config/
│   │   └── config.go             # precedence resolution
│   ├── render/
│   │   ├── sanitize.go           # control-char / ANSI neutralization (security-critical)
│   │   ├── human.go              # TTY-aware human output for blame/show/log
│   │   └── json.go               # stable JSON output structs
│   └── version/
│       └── version.go            # version string, injected at build time via -ldflags
└── docs/                         # (this planning tree)
```

**Dependency rule:** only `internal/git` may execute the `git` binary. Every other package that
needs git talks to it through `internal/git`. This concentrates the exec-safety surface in one
audited package.

### 3.3 External dependencies

- Go standard library.
- `github.com/spf13/cobra` (CLI framework) and its transitive `github.com/spf13/pflag`.
- No other runtime dependencies in v1. Any addition requires an ADR.
- The `git` binary (>= 2.20) must be present on `PATH` at runtime.

---

## 4. Data model

### 4.1 Rationale record — schema `tracegit/v1`

One JSON object per commit, stored as the git note body for that commit under
`refs/notes/tracegit`.

```jsonc
{
  "schema": "tracegit/v1",          // REQUIRED. Exact string. Guards forward-compat.
  "summary": "string",              // REQUIRED. One-line human summary of the change. <= 1 KiB.
  "rationale": "string",            // OPTIONAL. Multi-line markdown "why". <= 64 KiB (configurable).
  "agent": {                        // OPTIONAL. Who/what authored the change.
    "name": "string",               //   e.g. "claude-code", "codex", "human"
    "model": "string",              //   e.g. "claude-opus-4-8"
    "version": "string"             //   agent/tool version
  },
  "task": {                         // OPTIONAL. Work item linkage.
    "id": "string",                 //   e.g. "42", "PROJ-17"
    "url": "string"                 //   e.g. issue/PR URL. Validated as http(s) on write.
  },
  "confidence": "low|medium|high",  // OPTIONAL. Author's confidence in the change.
  "tags": ["string"],               // OPTIONAL. Freeform labels. <= 16 tags, each <= 64 bytes.
  "created_at": "RFC3339",          // Set by tracegit on write. UTC.
  "tracegit_version": "string"      // Set by tracegit on write. Producer version.
}
```

**Field rules**

| Field | Required | Type | Constraint |
|---|---|---|---|
| `schema` | yes | string | Must equal `tracegit/v1` on write. On read, unknown schema → treat as opaque/degraded. |
| `summary` | yes | string | Non-empty after trim. Max 1 KiB (1024 bytes UTF-8). |
| `rationale` | no | string | Max `tracegit.maxRationaleBytes` (default 65536). |
| `agent.name` | no | string | Max 128 bytes. |
| `agent.model` | no | string | Max 128 bytes. |
| `agent.version` | no | string | Max 64 bytes. |
| `task.id` | no | string | Max 128 bytes. |
| `task.url` | no | string | Must parse as absolute `http`/`https` URL if present. Max 2048 bytes. |
| `confidence` | no | enum | One of `low`, `medium`, `high`. |
| `tags` | no | string[] | ≤ 16 items; each non-empty, ≤ 64 bytes, no control chars. |
| `created_at` | set by tool | string | RFC3339, UTC (`Z`). |
| `tracegit_version` | set by tool | string | Producer version. |

**Forward compatibility.** On read, unknown top-level fields are preserved (kept in a raw map)
so a v1 tool round-tripping a future record does not silently drop data. On write, `tracegit`
emits only known fields plus any preserved unknowns. Records whose `schema` is not `tracegit/v1`
are shown in a "degraded" mode: raw JSON is displayed, structured views are skipped, and writes
that would overwrite them require `--force`.

### 4.2 Canonical JSON encoding

- UTF-8, no BOM.
- Keys emitted in a fixed order (schema, summary, rationale, agent, task, confidence, tags,
  created_at, tracegit_version) for stable diffs.
- Two-space indentation. Trailing newline.
- Empty optional objects/arrays are omitted (`omitempty` semantics) to keep notes small.

### 4.3 Storage in git notes

- Notes ref: `refs/notes/tracegit` (override with `tracegit.notesRef` / `--notes-ref` /
  `TRACEGIT_NOTES_REF`).
- One note per commit. The note body is the canonical JSON from §4.2.
- Writing uses `git notes --ref=<ref> add -F - <sha>` (record piped on stdin) with `-f` only
  when overwrite is explicitly requested.
- Reading uses `git notes --ref=<ref> show <sha>`.
- Listing uses `git notes --ref=<ref> list` (yields `<note-blob-sha> <commit-sha>` pairs).

---

## 5. Configuration

### 5.1 Resolution precedence (highest wins)

1. Command-line flags.
2. Environment variables.
3. `git config` (`--local` overrides `--global` per git's own rules).
4. Built-in defaults.

### 5.2 Keys

| Concept | Flag | Env var | git config key | Default |
|---|---|---|---|---|
| Notes ref | `--notes-ref` | `TRACEGIT_NOTES_REF` | `tracegit.notesRef` | `refs/notes/tracegit` |
| Agent name | `--agent` | `TRACEGIT_AGENT` | `tracegit.agent` | *(unset)* |
| Agent model | `--model` | `TRACEGIT_MODEL` | `tracegit.model` | *(unset)* |
| Task id | `--task` | `TRACEGIT_TASK` | `tracegit.task` | *(unset)* |
| Task URL | `--task-url` | `TRACEGIT_TASK_URL` | `tracegit.taskUrl` | *(unset)* |
| Max rationale bytes | `--max-rationale-bytes` | `TRACEGIT_MAX_RATIONALE_BYTES` | `tracegit.maxRationaleBytes` | `65536` |
| git binary | *(none)* | `TRACEGIT_GIT_BINARY` | `tracegit.gitBinary` | `git` (resolved on `PATH`) |
| Color | `--color=auto\|always\|never` | `TRACEGIT_COLOR` | `tracegit.color` | `auto` |

Environment defaults let an agent set `TRACEGIT_AGENT`/`TRACEGIT_MODEL` once per session so every
`tracegit commit` is attributed without repeating flags.

---

## 6. CLI reference (v1)

Global flags (root): `--notes-ref`, `--git-dir` (pass-through to git via `-C`/`--git-dir`
semantics), `--color`, `--json` (where supported), `-v/--verbose`, `--version`.

### 6.1 `tracegit commit`

Wraps `git commit`, then writes a rationale note to the resulting commit.

```
tracegit commit [-m <msg>]... [--rationale <text> | --rationale-file <path> | --rationale-stdin]
                [--summary <text>] [--agent <name>] [--model <m>] [--task <id>] [--task-url <url>]
                [--confidence low|medium|high] [--tag <t>]...
                [-- <git commit args>]
                [pass-through: -a/--all, --amend, --no-edit, -S/--gpg-sign, --author=<a>,
                               --no-verify, --allow-empty, --date=<d>, -p/--patch, -F/--file,
                               --reset-author, -C/--reuse-message, -e/--edit]
```

Behavior:

1. Resolve config; build a `tracegit/v1` record from flags. `summary` defaults to the first line
   of the commit message when `--summary` is omitted.
2. Validate the record and enforce size limits **before** committing (fail early, nothing
   written).
3. Exec `git commit <passed args>`. If git exits non-zero (e.g. aborted editor, failed hook),
   `tracegit` exits with the same code and writes **no** note.
4. `git rev-parse HEAD` → new commit SHA.
5. Write the note to that SHA (no `-f`; a fresh commit has no note).
6. If the note write fails, the commit still stands. Print a clear recovery instruction
   (`tracegit annotate HEAD ...`) and exit non-zero.

Edge cases: `--amend` changes HEAD's SHA; `tracegit` re-resolves HEAD after commit, so the note
attaches to the amended commit (the pre-amend note, if any, is orphaned — documented). Empty
`summary` after applying defaults is an error.

### 6.2 `tracegit annotate <commit-ish>`

Attach or replace a rationale note on an existing commit.

```
tracegit annotate <commit-ish> [--rationale... | --summary...] [--force] [--edit] [--append]
```

- Default: refuse if a note already exists (exit non-zero) unless `--force` (replace) or
  `--append` (append to `rationale` with a separator).
- `--edit` opens `$EDITOR` (then `$GIT_EDITOR`, then `core.editor`, then a platform default) with
  the current/derived record as JSON for editing; the result is validated before writing.
- `<commit-ish>` is resolved with `git rev-parse --verify <ish>^{commit}`; a non-commit or
  ambiguous ref is a hard error.

### 6.3 `tracegit show <commit-ish>`

Show the rationale record for a commit.

```
tracegit show <commit-ish> [--json] [--rationale]
```

- Human mode: commit header (sha, author, date, subject) + summary; `--rationale` adds the full
  rationale body.
- `--json`: emits the record (or a `{ "present": false }` object when no note exists).
- No note → human mode prints a one-line "no rationale recorded" notice and exits 0.
- Degraded record (unknown schema / invalid JSON) → prints raw body with a warning banner,
  exit 0.

### 6.4 `tracegit blame <file>`

Reasoning-aware blame.

```
tracegit blame <file> [<rev>] [-L <start>,<end>] [--json] [--rationale] [-C] [-M]
```

- Runs `git blame --porcelain [<rev>] [-L ...] [-C] [-M] -- <file>` and parses it (§7.2).
- Groups consecutive lines by commit; for each commit, looks up its note once (cached).
- Human mode: each line prefixed with short SHA + author + one-line summary (sanitized).
  `--rationale` prints the full rationale as a foldable block above each commit's first line.
- `--json`: array of line records `{ line, content, commit: {sha, author, author_time},
  rationale: <record|null> }`.
- A file with no blame (untracked/binary) → clear error, exit non-zero.

### 6.5 `tracegit log`

Rationale-aware log.

```
tracegit log [<revrange>] [-n <count>] [--json] [--rationale] [-- <path>...]
```

- Runs `git log` with a stable pretty format (§7.3), parses commits, attaches notes.
- Human mode: per commit, header + summary; `--rationale` adds the body.
- `--json`: array of `{ commit: {...}, rationale: <record|null> }`.

### 6.6 `tracegit export`

Bulk-export rationale records.

```
tracegit export [--format json|jsonl] [--output <path>] [<revrange>]
```

- Iterates notes on the configured ref (optionally intersected with `<revrange>`), emits an
  array (`json`) or one record per line (`jsonl`).
- Each emitted item includes the commit SHA: `{ "commit": "<sha>", "record": {...} }`.
- Degraded records are emitted with `{ "commit": "<sha>", "raw": "<body>", "error": "<reason>" }`.
- `--output` writes to a file (created 0644); default is stdout. Never sanitizes (machine output),
  but always valid JSON/JSONL.

### 6.7 `tracegit sync`

Move the notes ref between remotes. **Never runs implicitly.**

```
tracegit sync setup [<remote>]      # add fetch+push refspecs for the notes ref to remote config
tracegit sync fetch [<remote>]      # git fetch <remote> <notesRef>:<notesRef>
tracegit sync push  [<remote>]      # git push  <remote> <notesRef>:<notesRef>
```

- `<remote>` defaults to `origin`.
- `setup` adds `+refs/notes/tracegit:refs/notes/tracegit` to `remote.<r>.fetch` and configures a
  matching push refspec, both idempotently (no duplicate lines).
- `fetch` uses git's notes merge only when explicitly requested (`--strategy <ours|theirs|union|manual>`,
  default `manual` → abort on conflict with instructions). See ADR-001 for the concurrency note.

### 6.8 `tracegit init`

Prepare a repository.

```
tracegit init [--agent <name>] [--remote <r>] [--no-sync-setup]
```

- Writes chosen defaults into `git config --local tracegit.*`.
- Unless `--no-sync-setup`, runs `sync setup` for the given/`origin` remote.
- Idempotent; safe to re-run.

### 6.9 `tracegit version`

Prints `tracegit` version, git version detected, and notes ref in use. `--json` supported.

---

## 7. Git integration details

### 7.1 Exec model

- All git invocations go through `internal/git.Runner`, which builds `*exec.Cmd` with an explicit
  argv slice — **never** a shell string, never `sh -c`.
- The git binary is resolved once: `TRACEGIT_GIT_BINARY` / `tracegit.gitBinary` if set, else
  `exec.LookPath("git")`. Absence is a clear startup error.
- Minimum git version 2.20 is checked via `git --version` on first use; older → hard error with
  guidance.
- Working directory / repo scoping: the process inherits CWD; `--git-dir`/`-C` style options are
  forwarded as leading git args. Repository presence is verified with
  `git rev-parse --is-inside-work-tree`.
- stdin is used to pass record bodies to `git notes add -F -` (avoids temp files and arg-length
  limits).

### 7.2 `git blame --porcelain` parsing

The porcelain format emits, per line, a header line `<40-hex-sha> <orig-line> <final-line>
[<num-lines>]` followed by key-value metadata (`author`, `author-mail`, `author-time`,
`author-tz`, `summary`, `previous`, `filename`, ...) the **first** time a commit is seen, then a
TAB-prefixed content line. Parser requirements:

- Maintain a map of SHA → commit metadata (author, author-time) populated on first sighting.
- Emit ordered line records: `{ finalLine, sha, content }`.
- Tolerate `boundary` markers and repeated SHAs without re-reading metadata.
- Never assume metadata ordering beyond "header before its content line."
- Reject/þ flag lines that don't match expected shapes rather than panicking.

### 7.3 `git log` parsing

Use a delimiter-based pretty format to avoid ambiguity, e.g.
`--pretty=format:%H%x1f%an%x1f%aI%x1f%s` (unit-separator `\x1f` between fields, records separated
by `\x1e` or newline). Parse into `{ sha, author, author_time, subject }`. Never parse the
default human `git log` layout.

### 7.4 Notes read/write/list

- `Add(ref, sha, body, force)` → `git notes --ref=<ref> add [-f] -F - <sha>` with `body` on stdin.
- `Show(ref, sha)` → `git notes --ref=<ref> show <sha>`; distinguishes "no note" (specific git
  exit/stderr) from real errors.
- `List(ref)` → `git notes --ref=<ref> list` → `[]{noteBlob, commit}`.
- All note bodies read back are treated as untrusted (§9).

---

## 8. Rendering & output

- **Human mode** is TTY-aware: color and box-drawing only when stdout is a terminal and
  `--color`/`TRACEGIT_COLOR` allows; otherwise plain text.
- **`--json` mode** emits stable, documented structs (see per-command shapes in §6). JSON output
  is the contract for scripting and must not change shape within v1 without a schema note.
- All human-mode strings that originate from note content or git-provided author/summary fields
  pass through `render.Sanitize` first (§9.2).
- Exit codes: `0` success; `1` generic failure; `2` usage error; git's own exit code is
  propagated where `tracegit` is a thin wrapper (commit).

---

## 9. Security model

`tracegit` is intended to be released as open source and will be run inside developers' repos and
CI. Security is a core requirement, not a later hardening pass.

### 9.1 Trust boundaries

| # | Boundary | Untrusted input | Primary risk | Mitigation |
|---|---|---|---|---|
| B1 | CLI args / env | messages, rationale, agent name, tags, paths | Command injection | Argv-only exec; **no shell** anywhere; no string interpolation into commands. |
| B2 | Rationale content (write) | user/agent text | Size DoS, malformed data | Enforce size limits + validation *before* writing (§4.1). |
| B3 | Note content (read) | note bodies (incl. fetched from remotes) | Terminal escape injection, parser crash, size DoS | Sanitize on render (§9.2); strict, bounded JSON decode; graceful degrade. |
| B4 | git output parsing | blame/log stdout | Parser crash / desync | Defensive parsers; bounded; flag-not-panic on malformed input. |
| B5 | Network (sync) | remote notes ref | Pulling hostile notes | No implicit network; explicit `sync` only; fetched notes re-enter B3. |
| B6 | git binary / PATH | resolved executable | PATH hijack | Prefer explicit `TRACEGIT_GIT_BINARY`; document PATH trust; version-gate. |
| B7 | Filesystem (export/edit) | `--output`, `$EDITOR` temp files | Overwrite / temp leakage | Explicit paths only; 0600 temp files for `--edit`; `--output` never auto-created outside CWD-relative path given by user. |

### 9.2 Terminal-output sanitization (security-critical)

Note content and git author/summary strings are attacker-influenceable (a malicious contributor
can push a note or craft a commit). When rendered to a TTY they could carry ANSI/OSC escape
sequences that spoof output, rewrite the terminal title, or (with some terminals) trigger
clipboard/hyperlink abuse.

`render.Sanitize` MUST, for human output:

- Strip or escape all C0 control characters except `\t` and `\n`.
- Strip C1 controls and the ESC (`0x1b`) byte and any CSI/OSC/DCS sequences.
- Strip the BEL (`0x07`) and other terminal-control bytes.
- Leave normal printable UTF-8 (including non-Latin scripts and emoji) intact.
- Optionally cap displayed line width for blame/log summaries (truncate with ellipsis) — bounded
  memory.

`--json` output does **not** ANSI-sanitize (it is machine output) but MUST always be valid,
properly-escaped JSON.

### 9.3 Size limits

- `summary` ≤ 1 KiB; `rationale` ≤ `tracegit.maxRationaleBytes` (default 64 KiB); tags ≤ 16 ×
  64 B; `task.url` ≤ 2048 B. Enforced on write.
- On read, notes larger than a hard ceiling (`4 × maxRationaleBytes`, absolute cap 1 MiB) are
  truncated for display and flagged, never loaded unboundedly.
- JSON decode uses a bounded reader.

### 9.4 Secret handling

- `tracegit` does not, in v1, scan rationale for secrets. It DOES prominently document that
  rationale is stored in git notes and can be pushed to remotes, and that **secrets must not be
  placed in rationale**. (`SECURITY.md` + `README` + `commit`/`annotate` help text.)
- `tracegit sync push` is always explicit and never automatic, reducing accidental exfiltration.
- No credentials are ever read, stored, or logged by `tracegit`. Network auth is entirely git's.

### 9.5 Dependency & supply-chain risk

- Minimal dependency set (cobra + stdlib). Adding a dependency requires an ADR.
- CI runs `go vet`, `gofmt -l` (fail on diff), a linter (`golangci-lint`), `go test -race`, and
  `govulncheck`.
- Dependabot for Go modules and GitHub Actions; CodeQL (Go) on PR and schedule; secret scanning +
  push protection enabled on the repo.
- Release artifacts built reproducibly via a pinned, minimal workflow; checksums published.

### 9.6 Abuse cases (must be covered by tests — see issue 21)

1. Command injection via crafted agent name / message / tag → no shell means no injection; test
   asserts arguments reach git verbatim.
2. ANSI/OSC escape payload in rationale/summary → sanitized in human output; test asserts escapes
   removed.
3. Oversized rationale on write → rejected; oversized note on read → truncated + flagged.
4. Malformed JSON / unknown-schema note → degraded display, no crash, exit 0 on read paths.
5. Malformed `git blame`/`git log` output → parser flags/skips, no panic.
6. Fetched hostile notes → treated identically to local untrusted notes (B3 mitigations).

---

## 10. Failure modes & error handling

| Situation | Behavior |
|---|---|
| `git` not found | Startup error with install guidance; exit 2. |
| git version < 2.20 | Error naming detected version; exit 2. |
| Not inside a work tree | Error; exit 2. |
| `git commit` fails / aborted | Propagate git's exit code; write no note. |
| Note write fails after commit | Commit stands; print `tracegit annotate HEAD ...` recovery; exit 1. |
| Note already exists (annotate, no `--force`) | Refuse; exit 1; suggest `--force`/`--append`. |
| No note on read (show/blame/log) | Graceful: "no rationale recorded"; exit 0. |
| Malformed/unknown-schema note | Degraded display + warning; exit 0. |
| Oversized note on read | Truncate + flag; exit 0. |
| Ambiguous/invalid `<commit-ish>` | Error; exit 2. |
| `sync` with no configured remote | Error naming missing remote; exit 2. |

---

## 11. Testing strategy

- **Unit tests** for `record` (validation, limits, round-trip incl. unknown-field preservation),
  `render.Sanitize` (escape/control-char table-driven cases), `config` (precedence), and the
  blame/log parsers (fixture-driven, including malformed inputs).
- **Integration tests** using ephemeral git repos created in `t.TempDir()`: run real `git`,
  exercise `commit → show → blame → log → export → sync (to a bare local remote)`.
- **Security tests** (issue 21) covering §9.6 abuse cases.
- **Race detector** (`go test -race`) in CI.
- Deterministic: tests set `GIT_AUTHOR_*`/`GIT_COMMITTER_*` and a fixed notes ref.

---

## 12. Release & distribution

- Single static binary per platform (linux/amd64, linux/arm64, darwin/amd64, darwin/arm64,
  windows/amd64) via `goreleaser` (or equivalent), with SHA256 checksums.
- Version injected via `-ldflags -X .../internal/version.Version=...`.
- Merge to `main` ≠ release: tagging a release is a separate, human-gated step.
- `LICENSE`, `SECURITY.md`, `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md` present before public release.

---

## 13. Deferred (v2 ideas)

| Idea | Why deferred |
|---|---|
| Hook-based auto-capture (`prepare-commit-msg`/`post-commit`) | Adds fragility/magic; wrapper is more predictable for v1. |
| Trailer ingestion (`Tracegit-Rationale:`) | Second capture path; validate the notes model first. |
| `tracegit search` (full-text over rationale) | `export` + external tools suffice for v1. |
| Interactive TUI blame | Large UI surface; not core to the value prop. |
| Editor/IDE integration (VS Code) | Depends on stable JSON contract shipping first. |
| Signed / verifiable notes | Notes signing is awkward in git; needs its own design. |
| Secret scanning / redaction of rationale | Needs a careful UX; documented warning suffices for v1. |
| Notes conflict auto-merge strategies beyond `manual` | Requires field experience with concurrency. |

---

## 14. Cross-references

- Storage decision: [ADR-001](./decisions/ADR-001-git-notes-storage.md)
- Language & git-access decision: [ADR-002](./decisions/ADR-002-go-and-shell-out-to-git.md)
- Capture-mechanism decision: [ADR-003](./decisions/ADR-003-commit-wrapper-capture.md)
- Terminal sanitization requirement: [ADR-004](./decisions/ADR-004-terminal-output-sanitization.md)
- Git mechanics we rely on: [research/git-notes-and-blame.md](./research/git-notes-and-blame.md)
- Execution plan & waves: [ISSUE_PLAN.md](./ISSUE_PLAN.md)
