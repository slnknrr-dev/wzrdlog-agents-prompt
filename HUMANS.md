# HUMANS.md — what the WZRDLOG prompt does for you

`AGENTS.md` is a prompt for AI coding agents. This file explains it for people.
Agents are told never to read this file, so you can write anything here.

## In one paragraph

The agent keeps its own working notes in `WZRDLOG*` files: the task, the roadmap, what failed,
what worked, known limits, reusable snippets, and a dictionary of your words. These notes live
next to your code and are versioned in a separate **local-only** git branch called `WZRDLOG`.
Your real git history stays clean, nothing is ever pushed without your request, and the agent
picks up where it left off in any new session.

## What you get

- **Memory across sessions.** A new session reads the notes and continues. You do not have to explain the project again.
- **Clean history.** `WZRDLOG*` is in `.gitignore`. The notes never enter your branches or your npm package.
- **History of the notes themselves.** Every change to the notes is a commit in the `WZRDLOG` branch, so you can browse, diff, and restore it.
- **No accidental publishing.** The agent never suggests a push. A `pre-push` hook refuses any push of `WZRDLOG`.
- **Talk your way.** The agent maps your words to fixed terms (`WZRDLOG_AND_TERMS.md`). You can say "login", "sign-in", or "auth", and the files are still named consistently.
- **Failures are remembered.** Dead ends are written down, and the agent re-reads them before new work, so it does not repeat them.
- **Measurable snippets.** Reusable pieces carry their size in the file name (see below).
- **Your language.** The agent answers in the language you started in, and keeps it until you ask to change it.

## The files

| File | What is in it | Should you read it? |
|------|---------------|---------------------|
| `WZRDLOG.md` | The current task and its status | Yes, to see what the agent thinks it is doing |
| `WZRDLOG+roadmap.md` | The plan for the current epic, with links | Yes |
| `WZRDLOG+roadmap-<n>-<epic>.md` | Plans of finished epics | When looking back |
| `WZRDLOG_BY_OPERATOR.md` | Your preferences | **Edit it** to change the agent's defaults |
| `WZRDLOG_AND_TERMS.md` | Your words → the agent's terms → where to look | Useful when the agent misunderstands you |
| `WZRDLOG+failures*.md` | What did not work, plus deferred work and stubs | Good review material |
| `WZRDLOG+limitations*.md` | Known, accepted problems and gotchas | Before relying on a feature |
| `WZRDLOG+successes*.md` | What worked, principles, fast paths | To learn the project's good patterns |
| `WZRDLOG+<topic>-<theme>.md` | Design notes per topic | When working on that topic |
| `WZRDLOG+<action>-<result>-<N>.<ext>` | Reusable code or markup snippets | Copy from them freely |
| `BACKLOG*.md` | **Public**, short to-do list for developers | Yours; the agent does not touch it |

"Failures" vs "limitations": if something *could* be fixed but was not (a failed attempt, deferred work, a stub), it goes to failures. If it is accepted as it is, it goes to limitations.

## Typical cases

**Starting in a new folder.**
The agent checks the folder is sensible (not your home directory or a drive root) and runs `git init`.
It adds `WZRDLOG*` to `.gitignore`, installs the push guard, asks you once about language and design
preferences, and tells you about the local `WZRDLOG` branch.

**Coming back the next day.**
A new session restores any missing note files from the `WZRDLOG` branch, reads the task, roadmap, and failures, and continues.

**You interrupted the agent, or the session broke.**
The agent saves its notes and marks the task `interrupted: <where>`. Its own code changes are committed as `WIP: …`.
If it later finds uncommitted changes it did not make (possibly yours), it **asks** before committing them.

**Switching to a new big stage (epic).**
The old roadmap is archived as `WZRDLOG+roadmap-<n>-<epic>.md`, pinned to the exact commit of the task file.
A fresh roadmap starts, and the agent re-reads failures first.

**Your project already has an untracked `BACKLOG.md`.**
The agent commits it as is, rewrites it into a short developer-friendly list, commits that, and plans its notes from it.
Once `BACKLOG.md` is tracked in git, the agent leaves it alone unless you ask.

**You ask to push.**
The agent pushes only the branch you named, never `--all` or `--mirror`, and never `WZRDLOG`.

**You publish to npm.**
`.npmignore` gets `WZRDLOG*` only if it already exists. The agent never creates one, because a new
`.npmignore` would make npm ignore your `.gitignore` and could publish files like `.env`.

**Notes are getting long.**
The agent commits, trims the file, and leaves references like `WZRDLOG+failures.md@abc1234:L12-L30`.
Nothing is lost: run `git show abc1234:WZRDLOG+failures.md`.

## Snippets and the number in their name

`WZRDLOG+substr-domain-13.js` means: an action (`substr`) gives a result (`domain`) using **13 building blocks**.
Each snippet starts with a `contains:` list, and the number is simply the length of that list, so anyone can check it.

- A smaller number means a simpler version. When the agent optimizes `…-13.js` into `…-7.js`, the old file goes into the `WZRDLOG` history and the gain is noted in successes.
- If the number grows, the agent must write down why.
- It works for any file type, for example `WZRDLOG+mockup-login-9.html` for a mockup with 9 components.

## UI design defaults

These apply only to projects with a UI: sharp corners, no gradients, sea-wave and turquoise colors,
and light and dark themes from the start. To change them, write your overrides in `WZRDLOG_BY_OPERATOR.md`, for example:

```md
ui-design: rounded corners allowed; brand color #FF5500
```

## Handy commands

```bash
git log --oneline WZRDLOG
```

```bash
git show WZRDLOG:WZRDLOG.md
```

```bash
git diff WZRDLOG~1 WZRDLOG
```

Back up the notes, since they are never pushed:

```bash
git bundle create wzrdlog-backup.bundle WZRDLOG
```

Restore from that backup in another clone:

```bash
git fetch wzrdlog-backup.bundle WZRDLOG:WZRDLOG
```

## Things to know

- The notes exist only on your machine. Deleting the folder or `.git` deletes them. Use a bundle backup if they matter.
- `git clean -fdx` (and `-X`, `git stash --all`) deletes the note files. The agent will not run these; if you do, the next session restores the files from the branch, but uncommitted note changes are lost.
- The push guard is a git hook. It does not stop `git push --no-verify`, and it is not copied with a clone.
- Hook managers such as husky may regenerate hooks. The agent re-checks the guard every session.
- Notes are committed with your git identity. If `user.name` or `user.email` is not set, the agent asks you to set it.
- The scripts live in `.git/wzrdlog/` and are recreated by the agent if missing.
