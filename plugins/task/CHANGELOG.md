# Changelog

All notable changes to the task plugin will be documented in this file.

## [2.0.0] - 2026-01-26

### Breaking Changes

- `/task:start` no longer delegates to `/kas:start`. Now uses superpowers-driven planning workflow.

### Added

- **Superpowers integration** for planning and implementation phases
- `superpowers:brainstorming` for exploration and design (replaces kas:start explore/design)
- `kas:review-plan` iteration loop with max 5 iterations
- Implementation choice: subagent-driven (same session) or executing-plans (separate session)
- ClickUp status rollback on abort (captures original status before updating)
- `ExitPlanMode` call after review approval
- Session file (`.claude/task-session.json`) for cross-session state
- Session file cleanup in `/task:done` (step 6)
- Error handling for EnterPlanMode and "already in plan mode" scenarios

### Changed

- Updated `CLAUDE.md` with Superpowers Integration section
- Updated `SKILL.md` command mapping to reflect new workflow
- Updated dependencies to require `superpowers` plugin

## [1.0.2] - Previous

- Initial stable release with kas:start delegation
