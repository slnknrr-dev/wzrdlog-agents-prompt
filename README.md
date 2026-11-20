# WZRDLOG — an AGENTS.md prompt for AI coding agents

`AGENTS.md` turns an AI coding agent into a disciplined worker with memory. The agent keeps its
working notes in `WZRDLOG*` files: the task, the roadmap, failures, successes, limitations,
reusable snippets, and a dictionary of your terms. It versions them in a **local-only** git branch,
`WZRDLOG`, and keeps your real history clean.

## Why

- Agents forget everything between sessions. WZRDLOG gives them a memory that survives sessions and has its own git history.
- Agents repeat failed approaches. WZRDLOG makes them write failures down and re-read them.
- Agents push and publish too eagerly. WZRDLOG keeps git local, blocks pushing the notes with a hook, and keeps them out of npm.
- Long prompts get ignored. AGENTS.md is a gated checklist with exact commands, not prose.

## What is inside

| File | For whom | Purpose |
|------|----------|---------|
| [AGENTS.md](AGENTS.md) | The agent | The prompt: hard rules, a start checklist, scripts, file map, filing rules, commit, epic, and compaction procedures |
| [HUMANS.md](HUMANS.md) | You | What the prompt does, typical cases, handy commands, caveats. Agents are told never to read it |
| README.md | You | This overview |
| [CHANGELOG.md](CHANGELOG.md) | You | What changed between versions |
| [LICENSE](LICENSE) | You | CC BY-SA 4.0 |

## Quick start

1. Copy `AGENTS.md` (and, if you like, `HUMANS.md`) into the root of your project.
2. If your agent does not read `AGENTS.md` on its own, point to it. For Claude Code, create `CLAUDE.md` with one line:
   ```md
   @AGENTS.md
   ```
3. Start a session. On the first run, the agent:
   - checks the folder and runs `git init` if needed;
   - adds `WZRDLOG*` to `.gitignore`;
   - installs the `pre-push` guard and its scripts in `.git/wzrdlog/`;
   - asks once about your language and design preferences.

## You already have an AGENTS.md

- Keep your project rules at the top of your `AGENTS.md`, and paste the WZRDLOG protocol below them, from `# AGENTS.md — WZRDLOG protocol` to the end. The protocol is self-contained, and its sections are numbered, so it does not depend on what is above it.
- Do not save the protocol under a name that starts with `WZRDLOG`. Such a file matches `WZRDLOG*` and would be git-ignored.
- If your project already has its own `HUMANS.md`, the agent will not read it either. Move anything the agent must know into `AGENTS.md`.
- `CLAUDE.md`, `.cursorrules`, and similar files can stay. Make them reference `AGENTS.md` instead of duplicating it.

## Cost

The protocol is not free in tokens. Plan for it:

- `AGENTS.md` is about 340 lines (roughly 4–5k tokens) and is read at the start of every session.
- At start, the agent also reads `WZRDLOG_BY_OPERATOR.md`, `WZRDLOG_AND_TERMS.md`, `WZRDLOG.md`, `WZRDLOG+roadmap.md`, `WZRDLOG+failures.md`, and the themed files that match the task. These grow with the project.
- Compaction (section 10 of `AGENTS.md`) keeps these files short. Old content moves into the `WZRDLOG` branch history and costs nothing until it is needed.
- In exchange, the agent does not re-discover the project or repeat known failures, which usually costs more.

## Key ideas

- **Two histories in one repo.** Code goes to your branches. Notes go to the `WZRDLOG` branch, which is never pushed.
- **Gated checklists.** Each step is a yes/no question with an action, so the agent cannot skip ahead.
- **Canonical terms.** You speak freely, and file names stay stable.
- **Measurable primitives.** `WZRDLOG+substr-domain-13.js` has 13 building blocks, listed in its header. Optimizations show up as falling numbers.
- **Archive, not hoard.** Old content is deleted and referenced as `file@hash:L10-L20`.

## Requirements

- git, and a POSIX `sh` (Git Bash on Windows).
- `user.name` and `user.email` configured in git.

## Version

Current version: **0.1.0**. The version is also stated at the top of `AGENTS.md`. Changes are listed in [CHANGELOG.md](CHANGELOG.md).

## License

© 2026 Yury Slinkin. [CC BY-SA 4.0](LICENSE).
