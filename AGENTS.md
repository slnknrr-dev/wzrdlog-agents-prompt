# AGENTS.md — WZRDLOG protocol

Version: 0.1.0

**Never read `HUMANS.md`** next to this file, or in any folder. It is for people, not for you.

This file is a gated checklist. Run each step in order. A step is closed only when its
question is answered "yes" or its action is done. Do not skip ahead. Do not improvise
where an exact command or rule is given.

Words used below:
- **operator**: the human you work with.
- **prod branch**: whatever branch the project is on. It is not `WZRDLOG`.
- **WZRDLOG branch**: the local-only branch `refs/heads/WZRDLOG`, which holds only `WZRDLOG*` files.
- **canonical term**: a term from `WZRDLOG_AND_TERMS.md`.

---

## 0. Hard rules (always in force)

| # | Rule |
|---|------|
| H1 | Git is **local only**. Never suggest a push, a remote, or a hosting service. |
| H2 | Push only when the operator explicitly asks for it. Then push **only the named branch**: `git push <remote> <branch>`. Never use `--all`, `--mirror`, or `--tags` unless the operator asks for that exact flag. |
| H3 | **Never push `WZRDLOG`**, even if asked. Tell the operator the branch is local-only. |
| H4 | Never run `git clean -x`, `git clean -X`, `git clean -fdx`, or `git stash -a`/`--all`. They delete the ignored `WZRDLOG*` files. |
| H5 | Never read, create, or edit `BACKLOG*.md` unless the operator asks. The only exception is section 8. |
| H6 | Speak the language the operator used in their first message. Change it only when the operator explicitly asks. The stored choice is in `WZRDLOG_BY_OPERATOR.md`. |
| H7 | File-name slugs (`<topic>`, `<theme>`, `<epic>`, `<action>`, `<result>`) are lowercase Latin kebab-case and must be canonical terms. File content is written in the operator's language. |
| H8 | `WZRDLOG*` files are never committed to the prod branch. They are committed only by `wzrdlog/commit.sh` (section 2). |

---

## 1. START checklist (run at the start of every session, before any work)

```
S1  Is cwd inside a git repo?  (git rev-parse --show-toplevel)
      yes → use that top-level as ROOT. Go to S2.
      no  → go to S2 with ROOT = cwd. After S2 passes: git init. Go to S3.
S2  Is ROOT a safe project folder?
      Not safe: home dir, drive root, system dir, or a folder far above what the operator named.
      unsafe → ask the operator to confirm ROOT. STOP until they answer.
S3  Do the scripts in <git-common-dir>/wzrdlog/ exist with the exact content of section 2?
      no → create or overwrite them from section 2.
S4  Does the pre-push hook contain the marker WZRDLOG-GUARD?
      Hook path: "$(git rev-parse --git-path hooks)/pre-push"
      no → sh "<git-common-dir>/wzrdlog/install-hook.sh"
S5  Does ROOT/.gitignore contain the line  WZRDLOG*  ?
      no → append it. Never add per-folder .gitignore files for this.
S6  Does ROOT/.npmignore exist?
      yes, without the line WZRDLOG* → append it.
      no → do NOTHING. Never create .npmignore: npm would stop using .gitignore.
S7  Does refs/heads/WZRDLOG exist?
      yes → sh "<git-common-dir>/wzrdlog/restore.sh"   (restores missing files only)
      no  → it will be created by the first commit.sh run.
            Tell the operator once: "Local branch WZRDLOG keeps my working notes.
            It is never pushed; a pre-push hook blocks it."
S8  Is anything uncommitted?  (git status --porcelain; also WZRDLOG* vs WZRDLOG branch)
      WZRDLOG* changed → sh commit.sh "wzrdlog: recover unfinished session"
      prod files changed, unknown origin → ASK the operator: commit as WIP, or leave as is.
                                         Never commit them without an answer.
S9  Read WZRDLOG_BY_OPERATOR.md.
      missing → ask the operator, in their language and in one short message:
                language, UI design overrides, anything else they want remembered.
                Create the file from the answers (template in section 4).
                No answer → write the defaults.
S10 Read WZRDLOG_AND_TERMS.md.  missing → create it from the template.
S11 Read WZRDLOG.md and WZRDLOG+roadmap.md.  missing → create them from the templates.
S12 Read WZRDLOG+failures.md.  (required: first read of AGENTS.md in this session)
S13 Read only the themed WZRDLOG+*-<theme>.* files whose theme matches the current task.
S14 Only now start the operator's request.
```

`<git-common-dir>` means `$(git rev-parse --git-common-dir)`. Run scripts with `sh` (on Windows, Git Bash).
If `git commit-tree` fails because `user.name` or `user.email` is not set, tell the operator. Do not set them yourself.

---

## 2. Scripts (write these files exactly; they live in `.git`, so they are never versioned)

### `<git-common-dir>/wzrdlog/commit.sh`

```sh
#!/bin/sh
# Snapshot every WZRDLOG* file into refs/heads/WZRDLOG without touching the prod index or branch.
# usage: sh commit.sh "message"
set -e
cd "$(git rev-parse --show-toplevel)"
MSG=${1:-"wzrdlog: snapshot"}
DIR="$(git rev-parse --git-common-dir)/wzrdlog"
mkdir -p "$DIR"
IDX="$DIR/index.$$"
rm -f "$IDX"
trap 'rm -f "$IDX"' EXIT
if ! GIT_INDEX_FILE="$IDX" git add -f -- ':(glob)**/WZRDLOG*' ':(exclude,glob)**/node_modules/**' 2>/dev/null; then
  echo "WZRDLOG: no WZRDLOG* files to commit"; exit 0
fi
TREE=$(GIT_INDEX_FILE="$IDX" git write-tree)
PARENT=$(git rev-parse -q --verify refs/heads/WZRDLOG || true)
if [ -n "$PARENT" ]; then
  if [ "$(git rev-parse "$PARENT^{tree}")" = "$TREE" ]; then
    echo "WZRDLOG: nothing to commit ($PARENT)"; exit 0
  fi
  NEW=$(git commit-tree "$TREE" -p "$PARENT" -m "$MSG")
else
  NEW=$(git commit-tree "$TREE" -m "$MSG")
fi
git update-ref refs/heads/WZRDLOG "$NEW" ${PARENT:+"$PARENT"}
echo "WZRDLOG: $NEW"
```

### `<git-common-dir>/wzrdlog/restore.sh`

```sh
#!/bin/sh
# Restore WZRDLOG* files that exist in refs/heads/WZRDLOG but are missing on disk. Never overwrites.
set -e
cd "$(git rev-parse --show-toplevel)"
git rev-parse -q --verify refs/heads/WZRDLOG >/dev/null || { echo "WZRDLOG: no branch"; exit 0; }
git -c core.quotePath=false ls-tree -r --name-only refs/heads/WZRDLOG | while IFS= read -r f; do
  if [ ! -e "$f" ]; then
    mkdir -p "$(dirname "$f")"
    git show "refs/heads/WZRDLOG:$f" > "$f"
    echo "restored: $f"
  fi
done
```

### `<git-common-dir>/wzrdlog/install-hook.sh`

```sh
#!/bin/sh
# Install the pre-push guard. An existing hook is kept and chained, never overwritten.
set -e
HOOKS="$(git rev-parse --git-path hooks)"
mkdir -p "$HOOKS"
H="$HOOKS/pre-push"
if [ -f "$H" ] && grep -q WZRDLOG-GUARD "$H"; then echo "guard present"; exit 0; fi
[ -f "$H" ] && mv "$H" "$H.wzrdlog-chained"
cat > "$H" <<'EOF'
#!/bin/sh
# WZRDLOG-GUARD: refuse any push that involves refs/heads/WZRDLOG.
IN=$(cat)
HIT=$(printf '%s\n' "$IN" | while read -r lref lsha rref rsha; do
  if [ "$lref" = refs/heads/WZRDLOG ] || [ "$rref" = refs/heads/WZRDLOG ]; then echo hit; fi
done)
if [ -n "$HIT" ]; then
  echo "pre-push: branch WZRDLOG is local-only. Push refused." >&2
  exit 1
fi
if [ -x "$0.wzrdlog-chained" ]; then
  if [ -n "$IN" ]; then printf '%s\n' "$IN" | "$0.wzrdlog-chained" "$@"; else "$0.wzrdlog-chained" "$@" </dev/null; fi
  exit $?
fi
exit 0
EOF
chmod +x "$H"
[ -f "$H.wzrdlog-chained" ] && chmod +x "$H.wzrdlog-chained"
echo "guard installed: $H"
```

If `core.hooksPath` is set (husky and similar tools), `--git-path hooks` already resolves to it. Step S4 re-checks the guard every session, because such tools may regenerate the hooks.

---

## 3. File map

All files live in ROOT unless a themed file clearly belongs to a subfolder. All of them match `WZRDLOG*`.

| File | Holds |
|------|-------|
| `WZRDLOG_BY_OPERATOR.md` | Operator preferences: language, design overrides, standing wishes. |
| `WZRDLOG_AND_TERMS.md` | Dictionary: operator's words → canonical term → where to look. Never reset, only grows and gets refined. |
| `WZRDLOG.md` | The current task of the current epic. The first line links to `WZRDLOG_AND_TERMS.md`. |
| `WZRDLOG+roadmap.md` | The overall roadmap. Links to the other files and holds the archive index. |
| `WZRDLOG+roadmap-<n>-<epic>.md` | An archived roadmap of a finished epic. |
| `WZRDLOG+<topic>-<theme>.md` | Design notes for one topic. |
| `WZRDLOG+failures.md` / `WZRDLOG+failures-<topic>.md` | What did not work, wrong approaches, deferred work, stubs. |
| `WZRDLOG+limitations.md` / `WZRDLOG+limitations-<theme>.md` | Known problems and constraints that are accepted, non-obvious behaviour. **No deferred work and no stubs.** |
| `WZRDLOG+successes.md` / `WZRDLOG+successes-<theme>.md` | Good principles, fast correct paths, notes on primitives. Link to failures and primitives. |
| `WZRDLOG+<action>-<result>-<N>.<ext>` | A saved primitive (section 6). Any extension: `.js`, `.html`, `.py`, … |

---

## 4. Templates

`WZRDLOG_BY_OPERATOR.md`
```md
# Operator
language: <language of the operator's first message>
ui-design: default            # or overrides, e.g. "rounded corners allowed", "brand color #…"
other:
- <standing wish>
```

`WZRDLOG_AND_TERMS.md`
```md
# Terms
| Operator says | Canonical term | Where to look |
|---------------|----------------|---------------|
| login, sign-in, auth | auth | WZRDLOG+auth-flow.md |
```

`WZRDLOG.md`
```md
Terms: [WZRDLOG_AND_TERMS.md](WZRDLOG_AND_TERMS.md)
# Epic: <epic>
## Task
<what has to be done>
## Steps
- [ ] …
## Status
active | interrupted: <where it stopped> | done
```

`WZRDLOG+roadmap.md`
```md
# Roadmap — epic: <epic>
Current task: WZRDLOG.md @ <WZRDLOG commit hash>
## Items
- [ ] <item> → WZRDLOG+<topic>-<theme>.md
## Archive
- 1 <epic> → WZRDLOG+roadmap-1-<epic>.md
```

---

## 5. Filing rule (ask the questions in order; the first "yes" wins)

```
F1 Is it a word the operator uses for something?             → WZRDLOG_AND_TERMS.md
F2 Is it an operator preference?                             → WZRDLOG_BY_OPERATOR.md
F3 Is it a reusable piece of code or markup?                 → primitive file (section 6)
F4 Could it be fixed, but was not done, or was done badly?
   (failed attempt, wrong path, deferred work, stub, TODO)   → failures
F5 Is it a known problem or constraint that stays as is,
   or something non-obvious?                                 → limitations
F6 Did it work well? A principle, a fast path?               → successes
F7 Is it design or planning for a topic?                     → WZRDLOG+<topic>-<theme>.md
```

- Does it belong to one canonical theme? Then use the themed file (`…-<theme>.md`). Otherwise use the general one.
- New theme? Add the term to `WZRDLOG_AND_TERMS.md` first, then create the file.
- When the operator uses a new word, map it to an existing canonical term or create one. Record it in F1.

---

## 6. Primitives

Name: `WZRDLOG+<action>-<result>-<N>.<ext>`, for example `WZRDLOG+substr-domain-13.js`.

Every primitive starts with this header, written in the file's own comment syntax:
```
WZRDLOG primitive
does: <action> → <result>
contains:
  1. <building block>
  2. <building block>
  …
```

- **N = the number of lines under `contains`.** Nothing else. Count them; never estimate.
- What counts as a building block: in code, each distinct operation or call the result needs. In markup or mockups, each distinct component or element. In other formats, each indivisible unit of that format.
- **N went down** (optimized): save the new file, delete the old one, run `commit.sh`. In successes, record `<action>-<result>: <new N> (from <old N> @ <hash>)`.
- **N went up**: before saving, write the reason in successes or failures.
- Two approaches to the same `<action>-<result>` may exist side by side with different N. Record which one wins, and why.
- In successes and failures, name a primitive without N (`substr-domain`) and add its current N.

---

## 7. Commit checklist

Commit triggers: (a) the task is done, (b) the epic changes, (c) the operator interrupts or switches topic, (d) S8 found leftovers.

```
C1 Update WZRDLOG.md status (done / interrupted: <where>).
C2 Filing (section 5) for everything learned in this task.
C3 Prod files changed by you in this task?
      yes → git add <those files>; git commit -m "<what changed>"
            On trigger (c), prefix the message with "WIP: ".
            Never git add WZRDLOG* (they are ignored; never use -f for them).
C4 sh "<git-common-dir>/wzrdlog/commit.sh" "wzrdlog: <task or reason>"
C5 Do not suggest a push (H1).
```

---

## 8. Epic change checklist

```
E1 Run the commit checklist (section 7).
E2 HASH = the current refs/heads/WZRDLOG commit (git rev-parse refs/heads/WZRDLOG).
   It must contain the current WZRDLOG.md. If WZRDLOG.md has uncommitted changes, run commit.sh first.
E3 n = the last number in the roadmap "Archive" list + 1  (none → 1).
E4 Name the new epic yourself, using a canonical term (add it to the terms file if it is new).
E5 Copy WZRDLOG+roadmap.md → WZRDLOG+roadmap-<n>-<old epic>.md.
   Its first line: "WZRDLOG.md @ <HASH>".
E6 Rewrite WZRDLOG+roadmap.md and WZRDLOG.md for the new epic.
   Add the archive line to "Archive".
E7 Read WZRDLOG+failures.md (required after every roadmap change).
E8 sh commit.sh "wzrdlog: epic <n> archived, start <new epic>"
```

---

## 9. BACKLOG exception

`BACKLOG*.md` is public: it is tracked in git and meant for developers. Keep it short and to the point.

```
B1 git ls-files --error-unmatch BACKLOG.md
      succeeds (tracked) → rule H5 applies. STOP.
      fails (not tracked) → you may (re)write it. Continue.
B2 If BACKLOG.md exists, commit it as is to the prod branch first (this keeps the original).
B3 Read it, rewrite it (simple, for a developer), and commit it to the prod branch.
B4 Design the WZRDLOG files from it (roadmap, topics, terms).
B5 sh commit.sh "wzrdlog: designed from BACKLOG.md"
```

---

## 10. Compaction (you decide when)

Signals: you can no longer read the file in one pass, it holds outdated entries, or entries repeat.

```
K1 sh commit.sh "wzrdlog: before compaction of <file>"
K2 HASH = git rev-parse refs/heads/WZRDLOG
K3 Shorten, merge, or delete the file. Where removed content matters, leave a reference:
     <file>@<HASH>:L<from>-L<to>        read it back with: git show <HASH>:<file>
K4 sh commit.sh "wzrdlog: compacted <file>"
```

---

## 11. UI design defaults (only if the project has a UI)

Apply these unless `WZRDLOG_BY_OPERATOR.md` overrides them:
- Strict geometry: `border-radius: 0`. No rounded corners anywhere.
- No gradients.
- Palette is based on sea-wave (teal, e.g. `#008B8B`) and turquoise (e.g. `#30D5C8`), on neutral backgrounds.
- Ship a light theme and a dark theme from the start.

Projects without a UI ignore this section.
