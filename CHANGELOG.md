# Changelog

Versions follow [Semantic Versioning](https://semver.org/):
- **major**: a change that breaks existing `WZRDLOG*` files or the `WZRDLOG` branch layout;
- **minor**: a new rule, file type, or checklist step;
- **patch**: a fix or a clarification.

## 0.1.0 — 2026-11-20

First version.

- Hard rules: git is local only, no push unless the operator asks, `WZRDLOG` is never pushed, no `git clean -x`, BACKLOG is off-limits, and the operator's language is kept.
- Start checklist S1–S14: safe `git init`, scripts, `pre-push` guard, `.gitignore` / `.npmignore`, restore, recovery of unfinished sessions, reading order.
- Scripts in `.git/wzrdlog/`: `commit.sh` (snapshot into the orphan branch `WZRDLOG` through a separate index), `restore.sh`, and `install-hook.sh` (chains an existing hook).
- Files: `WZRDLOG.md`, the roadmap and its archive, failures, limitations, successes, topic files, `WZRDLOG_BY_OPERATOR.md`, `WZRDLOG_AND_TERMS.md`.
- Filing rule F1–F7.
- Primitives `WZRDLOG+<action>-<result>-<N>.<ext>`, where N is the number of lines in `contains`.
- Commit, epic change, BACKLOG, and compaction checklists.
- UI design defaults for projects with a UI.
