# Research: git notes and `blame --porcelain` mechanics tracegit relies on

> Purpose: pin down the exact git behaviors and output formats the implementation depends on, so
> implementation agents don't have to rediscover them. This is a reference for implementers, not a
> git tutorial. Verify commands against the local git (>= 2.20) during implementation.

## 1. Git notes

### 1.1 Model

- Notes attach arbitrary blob content to any object (we only annotate commits), stored under a
  notes ref — by default `refs/notes/commits`. tracegit uses a dedicated ref
  `refs/notes/tracegit`, selectable with `--ref=<name>` (short) or a full ref path.
- Attaching/editing a note does **not** change the annotated commit's SHA.
- One note object per (ref, commit) pair.

### 1.2 Commands used

| Operation | Command | Notes |
|---|---|---|
| Add (fresh) | `git notes --ref=tracegit add -F - <sha>` | Body on stdin. Fails if a note already exists. |
| Add/overwrite | `git notes --ref=tracegit add -f -F - <sha>` | `-f/--force` overwrites. |
| Show | `git notes --ref=tracegit show <sha>` | Prints body to stdout. Non-zero + stderr when no note. |
| List | `git notes --ref=tracegit list` | Lines `<note-blob-sha> <annotated-sha>`. `list <sha>` prints just the note blob sha or errors. |
| Remove | `git notes --ref=tracegit remove <sha>` | Not used in v1 write paths; relevant for tests. |
| Copy | `git notes --ref=tracegit copy <from> <to>` | Potential future use for `--amend` note migration (deferred). |

Passing the body via `-F -` (stdin) avoids temp files and OS argument-length limits for large
rationale.

### 1.3 "No note" vs error

`git notes show <sha>` on a commit without a note exits non-zero and writes an error like
`error: no note found for object <sha>.` to stderr. The runner MUST distinguish this "absent"
case (→ treat as no rationale, exit 0 on read paths) from genuine failures (bad ref, not a repo).
Implementation should match on the absence signal conservatively (non-zero exit + stderr pattern)
and, when uncertain, fall back to checking `git notes list <sha>`.

### 1.4 Sync (fetch/push)

Notes refs are **not** transferred by default fetch/push. To move them:

- Fetch: `git fetch <remote> refs/notes/tracegit:refs/notes/tracegit`
- Push: `git push <remote> refs/notes/tracegit:refs/notes/tracegit`
- Persistent config (what `tracegit sync setup` writes, idempotently):
  - `git config --add remote.<r>.fetch '+refs/notes/tracegit:refs/notes/tracegit'`
  - a matching push refspec (`remote.<r>.push` or explicit push command).

### 1.5 Concurrency / merge

Divergent notes on the same ref are reconciled with `git notes merge` and a strategy
(`manual` default, or `ours`/`theirs`/`union`/`cat_sort_uniq`). tracegit v1 defaults to `manual`
(abort on conflict, print guidance) and exposes `--strategy`. `union` concatenates note contents,
which for our JSON bodies would produce invalid JSON — so `union` is offered only with a
documented caveat and is never the default.

## 2. `git blame --porcelain`

### 2.1 Why porcelain

The porcelain format is stable and machine-parseable, unlike the default human layout. We invoke:

```
git blame --porcelain [<rev>] [-L <start>,<end>] [-C] [-M] -- <file>
```

### 2.2 Format

For each line of the file, output begins with a header line:

```
<40-hex-commit-sha> <orig-lineno> <final-lineno> [<num-lines-in-group>]
```

- The `<num-lines-in-group>` field appears only on the **first** line of a group of contiguous
  lines that share the same commit and origin.
- The **first time** a given commit SHA appears, git emits an extended block of key/value header
  lines before the content line, e.g.:
  - `author <name>`
  - `author-mail <email>`
  - `author-time <unix-epoch>`
  - `author-tz <±HHMM>`
  - `committer ...` (same set)
  - `summary <commit subject>`
  - `previous <sha> <filename>` (when applicable)
  - `filename <path>`
  - possibly `boundary` (for a boundary/root commit)
- Subsequent appearances of the same SHA emit only the header line + content line (metadata is not
  repeated).
- The content line is prefixed with a single TAB (`\t`) followed by the raw line content.

### 2.3 Parser contract (see DESIGN §7.2)

- Maintain `map[sha]CommitMeta`, filled on first sighting; reuse afterward.
- Produce ordered records `{ finalLine int, sha string, content string }`.
- Group consecutive records sharing a SHA for display; look up the note once per unique SHA.
- Treat `boundary` and missing optional keys (`previous`) as normal.
- The `summary` header is the commit subject, which is **untrusted** and must be sanitized before
  human display.
- On any line that does not match an expected shape, flag/skip defensively — never panic.

### 2.4 `-L`, `-C`, `-M`

- `-L <start>,<end>` restricts to a line range; forward verbatim.
- `-M` detects moved lines within a file; `-C` detects lines copied from other files. Both change
  which commit is attributed and may add `previous`/`filename` headers. v1 forwards these flags
  when the user passes them; default is neither.

## 3. `git log` (delimited)

To avoid parsing the human layout, use a delimited pretty format:

```
git log [<revrange>] [-n <count>] --pretty=format:'%H%x1f%an%x1f%aI%x1f%s' [-- <path>...]
```

- `%x1f` = ASCII unit separator (0x1f) between fields; records are newline-separated (a subject
  containing a newline is not possible with `%s`, which is the single subject line).
- Fields: `%H` full sha, `%an` author name, `%aI` author date (strict ISO-8601), `%s` subject.
- `%an` and `%s` are untrusted → sanitize before human display.

## 4. Version considerations

- Minimum supported git: **2.20** (porcelain blame keys and notes subcommands used here are stable
  well before this; 2.20 is a safe modern floor). Gate at startup via `git --version` parse.
- Windows: exec via argv works the same; path separators are handled by git. Line-ending and
  console-encoding differences affect only display, not the stored JSON.

## 5. Sources

These behaviors are documented in the official git manual pages (`git-notes(1)`,
`git-blame(1)` "THE PORCELAIN FORMAT", `git-log(1)` "PRETTY FORMATS"). Implementers should
confirm exact stderr strings and edge cases against the pinned local git version while writing the
runner and parsers, and encode any surprises as test fixtures.
