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

## Critical Technical Decisions

### `--output-format json` usage: planning only, NOT execution

- **Phase 1-2 (planning) commands: USE `--output-format json`** — PM waits for the final result (plan text + sessionId). Structured JSON output is ideal.
- **Phase 3 (execution) and Phase 4 (fix) commands: DO NOT use `--output-format json`** — PM monitors the coding-agent in real-time via `process action:log`, parsing streaming output for markers (`[CHECKPOINT]`, `[DONE]`, `[ERROR]`, `[DECISION_NEEDED]`). With `--output-format json`, Claude buffers all output and only emits a single JSON blob at the end, making real-time monitoring impossible.


## Integration Testing

Before publishing a new version, validate the full workflow end-to-end.

### End-to-end test: Texas Holdem game

Validates that coding-pm can manage a complete development lifecycle:

1. **Setup**: Create an empty project with `git init`
2. **Invoke**: `/dev "Build a Texas Holdem poker game for 2-8 players with a web GUI. Use HTML/CSS/JS with a Node.js backend. Include game logic (dealing, betting rounds, hand evaluation), a responsive web interface showing player hands and community cards, and WebSocket-based multiplayer."`
3. **Verify each phase**:
   - Phase 1: Worktree created at `~/.worktrees/texas-holdem/`, coding-agent starts planning
   - Phase 2: Plan includes game logic, UI components, WebSocket server, test strategy — PM reviews and presents
   - Phase 3: Coding-agent executes, PM monitors checkpoints and dangerous patterns
   - Phase 4: Automated tests pass, dev server starts, UI renders correctly (screenshot if available)
   - Phase 5: Merge and cleanup succeed — no worktree/branch artifacts remain
4. **Success criteria**:
   - Game runs at `localhost:3000` (or similar)
   - 2-8 players can join and play
   - Cards deal correctly, betting rounds work, hand evaluation is accurate
   - Web UI shows player hands, community cards, pot, and game state

### Smoke test: Claude CLI flags

Quick validation that coding-pm's CLI invocations work correctly:

```bash
# Planning phase — read-only tools should work
claude -p "List files in this directory" \
  --output-format json \
  --dangerously-skip-permissions \
  --allowedTools "Read,Glob,Grep,LS,Bash(git log *,git diff *)"

# Planning phase — write tools should be blocked
claude -p "Create a file called test.txt" \
  --output-format json \
  --dangerously-skip-permissions \
  --allowedTools "Read,Glob,Grep,LS"
# Agent should not be able to create files (no Write tool)
```

### Subagent test approach

For CI or automated validation, launch a Claude Code subagent as coding-pm:
- `claude -p "<task prompt>" --append-system-prompt-file SKILL.md` to simulate PM role
- Have it manage another claude instance as the coding-agent
- Verify the full 5-phase workflow completes
- Tests prompt quality and workflow logic end-to-end