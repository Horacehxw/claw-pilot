# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.4.0] - 2026-02-25

### Fixed
- **BUG-1**: `--tools` → `--allowedTools` (Claude Code CLI flag name fix)
- **BUG-2**: Step numbering in Phase 1 (5 → 4)
- **FIX-1**: Skill path hardcoding — added dynamic discovery with `find ~/.openclaw -path "*/coding-pm/references/supervisor-prompt.md"`
- **FIX-2**: `[DECISION_NEEDED]` flow now uses `--resume` instead of `process action:write`
- **FIX-3**: Mixed output format — execution phases use text mode (no `--output-format json`) for real-time monitoring

### Added
- **IMP-1**: Tightened git permissions in plan phase (`Bash(git log *,git diff *,git show *,git status,git branch --list *)`)
- **IMP-2**: Wake event fallback — create `.supervisor/wake-marker` if `openclaw system event` fails
- **IMP-3**: Security warnings in README.md and README_zh.md
- **IMP-4**: Auto-create `.supervisor/` directory in worktree for marker file fallback

### Changed
- All execution/resume commands now use plain text output (removed `--output-format json`)
- Supervisor protocol updated: CC exits on `[DECISION_NEEDED]`, PM resumes with answer

## [0.3.1] - 2026-02-25

### Fixed
- **Plan mode hangs**: plan command now uses `--dangerously-skip-permissions --tools "Read,Glob,Grep,LS,WebSearch,WebFetch,Bash(git *)"` for safe read-only exploration
- **CLAUDE.md file pollution**: replaced file injection with `--append-system-prompt-file` CLI flag
- **Monitoring loop can't execute**: replaced fictional polling loop with event-driven monitoring via `openclaw system event --mode now`
- **Concurrent state lost on context loss**: `/task list` now reconstructs state from `process action:list` + `git worktree list`
- **TDD too dogmatic**: TDD is now conditional on existing test framework
- **.gitignore outdated comment**: updated to reflect current architecture

### Added
- Wake notification protocol in supervisor-prompt.md (coding-agent notifies PM on key markers)
- Worktree isolation rules in supervisor-prompt.md (stay inside worktree, don't touch ~/.openclaw/)

### Changed
- **Claude Code only**: removed multi-agent detection table, all commands use `claude` CLI directly
- All `claude -p` commands include `--append-system-prompt-file` for supervisor protocol injection

## [0.3.0] - 2026-02-25

### Added
- **5-phase workflow**: preprocessing, plan review, execution monitoring, acceptance testing, merge & cleanup
- **Phase 1 Preprocessing**: PM explores project context, composes structured prompt for coding-agent
- **Active monitoring loop**: polls every 30-60s, parses markers, pushes progress to user
- **Structured markers**: `[CHECKPOINT]`, `[DECISION_NEEDED]`, `[ERROR]` (in addition to existing `[PLAN_START/END]`, `[DONE]`)
- **PM-to-agent communication**: `process action:write` to send corrections/answers mid-execution
- **3-layer acceptance testing**: automated tests, functional integration tests, screenshot analysis
- **Error retry protocol**: auto-retry up to 3 rounds before escalating to user
- **Concurrency management**: multiple independent tasks with per-task worktree, session, and phase tracking
- **Task commands**: `/task pause`, `/task resume`, `/task progress`, `/task plan`
- **Engineering Practices** in supervisor-prompt.md: Design First, TDD, Systematic Debugging, Verification Before Completion, Planning Discipline

### Changed
- **PM role deepened**: from dispatcher to real project manager (requirements coverage, process compliance, result quality)
- **Plan review**: PM checks requirements coverage, test plan, risks, format — no technical opinions
- **User feedback**: relayed verbatim to coding-agent (PM does not rewrite)
- **Terminology**: unified to `coding-agent` throughout
- **Language rule**: source files in English, IM output adapts to user's language

## [0.2.0] - 2026-02-25

### Changed
- **Renamed**: claw-pilot → coding-pm (complements coding-agent: agent executes, PM manages)
- **Rewritten SKILL.md**: pure platform tools (bash/process), no custom scripts
- **Multi-agent support**: Claude Code, Codex, OpenCode, Pi (was CC-only)
- **Simplified supervisor prompt**: removed progress.json protocol, kept plan markers + safety rules

### Removed
- All 6 shell scripts (init-task, start-cc, check-cc, merge-task, cleanup-task, list-tasks)
- templates/ directory (supervisor prompt moved to references/)
- docs/design.md (archived in git history)
- tests/test-scripts.sh (no scripts to test)
- task.json / progress.json / status / session_id / cc.pid file-based IPC
- ~/.openclaw/supervisor/tasks/ runtime directory
- jq dependency

### Added
- references/supervisor-prompt.md (simplified supervisor contract)
- Multi-agent detection and command table
- Auto-notify on completion
- .clawhubignore

## [0.1.0] - 2026-02-25

### Added
- Core workflow: plan → approve → execute → report
- SKILL.md agent instructions for PM/QA role
- 5 shell scripts: init-task, start-cc, check-cc, merge-task, cleanup-task
- Supervisor Protocol v1.0 (CLAUDE.md.tpl)
- Dual progress tracking: git commits + progress.json
- Git worktree isolation per task
- Background CC execution via setsid
- JSON output mode (~1KB per invocation)
- README (English + Chinese)
