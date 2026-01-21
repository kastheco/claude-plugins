# task

ClickUp task management with kas workflow integration.

## Overview

This plugin wraps kas workflow commands with ClickUp task synchronization. All ClickUp API calls are delegated to a subagent to avoid polluting the main conversation context.

## Prerequisites

1. **ClickUp MCP Server** - Authenticate with ClickUp when prompted (OAuth flow)
2. **kas plugin** - Required for workflow commands (`/kas:start`, `/kas:verify`, etc.)

## Installation

This plugin is part of the [kas-claude-plugins](https://github.com/brkastner/kas-claude-plugins) marketplace.

Add to your project's `.claude/settings.json`:

```json
{
  "plugins": ["kas-claude-plugins/task"]
}
```

## Quick Start

```bash
# 1. Start work on a ClickUp task (enters plan mode, creates worktree)
/task:start CU-abc123
# ... plan approved, worktree created ...
cd .worktrees/CU-abc123-feature-name/
/clear

# 2. Work on subtasks
/kas:next                    # Pick first beads issue
# ... implement ...
/kas:next                    # Pick next issue
# ... implement ...

# 3. Multi-session? Save progress and continue later
/kas:save
/clear
# ... next session, paste continuation prompt ...

# 4. All done? Complete and create PR
/task:done                   # Verifies, commits, pushes, creates PR, updates ClickUp

# 5. After PR review, merge and close
/task:merge                  # Merges PR, marks ClickUp complete, cleans up worktree
```

## Commands

### `/task:start <id-or-url>`

Start work on an existing ClickUp task.

**What it does:**
1. Fetches task details via subagent (name, description, acceptance criteria)
2. Updates ClickUp status to "in progress" and assigns to you
3. Delegates to `/kas:start` with task context injected

**Examples:**
```
/task:start CU-abc123
/task:start https://app.clickup.com/t/abc123
```

### `/task:done`

Complete work and create PR.

**What it does:**
1. Runs `/kas:verify` - stops if issues found
2. Runs `/kas:done` - commit, push
3. Creates PR with `[CU-xxx]` in title (if none exists)
4. Updates ClickUp status to "ready for review"
5. Comments PR URL on ClickUp task

### `/task:merge`

Merge PR and close task.

**What it does:**
1. Delegates to `/kas:merge` - CI check, merge, cleanup
2. Updates ClickUp status to "complete"

### `/task:status [id]`

Check task status.

**What it does:**
1. Detects task from argument or current branch (`feat/CU-xxx-*`)
2. Fetches task via subagent
3. Displays: name, status, assignee, description summary

### `/task:new [description]`

Create a new ClickUp task.

**What it does:**
1. Interviews you for: task type, project area, description, acceptance criteria
2. Creates task via subagent
3. Asks: "Start work on this task now?"

## Git Conventions

- **Branch naming**: `feat/CU-<id>-<slug>` or `fix/CU-<id>-<slug>`
- **Commit messages**: Include `CU-<task-id>` for ClickUp linking
- **PR titles**: Include `[CU-xxx]` prefix

## Architecture

```
User Command → task plugin
                    ↓
              clickup-task-agent (subagent)
                    ↓
              ClickUp MCP Server
                    ↓
              Returns summary to main context
                    ↓
              Delegates to kas commands
```

**Why subagent?** ClickUp API responses are verbose. Delegating to a subagent keeps the main conversation clean and focused.

## Error Handling

| Scenario | Action |
|----------|--------|
| ClickUp fetch fails at start | Stop, show error, suggest retry |
| Status update fails after kas success | Warn user, show manual update command |
| kas command fails | Stop, do NOT update ClickUp |

## Dependencies

- `kas` plugin (required)
- `commit-commands` plugin (recommended)
- `pr-review-toolkit` plugin (recommended)
