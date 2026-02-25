# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**coding-pm** is an OpenClaw Skill that turns the OpenClaw agent into a PM/QA managing Claude Code as a background engineer. The agent reviews plans, gates approval, monitors progress, validates tests, and reports results — all without blocking the chat session.

```
User (IM) → coding-pm (PM/QA, OpenClaw agent) → coding-agent (Engineer, background)
```

## Architecture

**Pure SKILL.md** — All PM/QA logic lives in natural language instructions. No scripts, no custom state management.

- `SKILL.md` — The PM brain. 5-phase workflow: preprocessing → plan review → execution monitoring → acceptance testing → merge & cleanup.
- `references/supervisor-prompt.md` — Appended to coding-agent's system prompt via `--append-system-prompt-file`. The contract between PM and coding-agent: marker protocol + engineering practices.

**PM manages people, not tech** — coding-pm ensures requirements coverage, process compliance, and result quality. coding-agent makes all technical decisions.

**Platform tools as hands** — Uses OpenClaw's built-in `bash` (pty/background/workdir) and `process` (poll/log/kill/list/write) tools instead of custom shell scripts.

**Worktree isolation** — Each task gets a git worktree at `~/.worktrees/<task-name>/` with a feature branch. Multiple tasks run concurrently.

**Flow**: `/dev <request>` → explore project → structured prompt → coding-agent plans → PM reviews (requirements, tests, risks) → user approves → coding-agent executes → PM monitors (active loop) → acceptance testing (automated + functional + visual) → user confirms → merge & cleanup.

## Development Commands

```bash
# Publish
clawdhub publish . --slug coding-pm --name "Coding PM" --version X.Y.Z --changelog "..."
```

## Conventions

- **Commit format**: `type(scope): description` (feat/fix/refactor/test/docs/chore)
- **Branch strategy**: Direct to main for v0.x. Feature branches when needed.
- **Language**: Source files in English. coding-pm adapts IM output to user's language automatically.
- **Version tags**: SemVer `v0.1.0`, `v0.2.0`, etc. Every tag = GitHub + ClawdHub release.

## Requirements

- OpenClaw 2026.2.19+, git 2.20+
- Claude Code 2.1.0+
- `tools.fs.workspaceOnly = false` in OpenClaw config (worktree paths are outside workspace)
