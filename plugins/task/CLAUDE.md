# task Plugin Instructions

## Core Principle

**NEVER call ClickUp MCP tools directly in main context.**

All ClickUp API operations must be delegated to the `clickup-task-agent` subagent. This keeps the main conversation clean - ClickUp API responses are verbose and would pollute context.

## Commands

| Command | Purpose | Wraps |
|---------|---------|-------|
| `/task:start <id>` | Start work on ClickUp task | superpowers:brainstorming → kas:review-plan → implementation |
| `/task:done` | Complete work, create PR, update ClickUp | kas:verify → kas:done |
| `/task:merge` | Merge PR, close ClickUp task | kas:merge |
| `/task:status [id]` | Check task status | (ClickUp only) |
| `/task:new` | Create new task via interview | (ClickUp only) |

## Subagent Delegation Pattern

When you need ClickUp data, spawn the agent:

```
Task tool with subagent_type="task:clickup-task-agent":
"[Operation]: [specific instructions]
Return ONLY: [concise format]"
```

See `skills/task-workflow/references/subagent-prompts.md` for copy-paste prompts.

## Git Conventions

- **Branches**: `feat/CU-<id>-<slug>` or `fix/CU-<id>-<slug>`
- **Commits**: Include `CU-<task-id>` for ClickUp linking
- **PR titles**: Include `[CU-xxx]` prefix

## Configuration

ClickUp MCP server uses OAuth - authenticate when prompted on first use.

For manual API token setup, see `skills/task-workflow/references/setup.md`.

## Superpowers Integration

This plugin uses superpowers skills for planning and implementation.

### Planning Phase

| Skill | Purpose |
|-------|---------|
| `superpowers:brainstorming` | Explore context, ideate, create design doc |
| `kas:review-plan` | Review design for gaps (loops until approved) |

### Implementation Phase

| Skill | When to Use |
|-------|-------------|
| `superpowers:subagent-driven-development` | Same session, small-medium tasks |
| `superpowers:using-git-worktrees` | Creates worktree for separate session |
| `superpowers:executing-plans` | Used in NEW session after worktree creation |

**Note:** For separate session (Option 2), workflow is: `using-git-worktrees` → save session file → user runs `/clear` → user invokes `executing-plans` in new session.

### Skill Invocation

```
Skill tool with skill="superpowers:brainstorming"
```

## Dependencies

This plugin requires:
- `kas` plugin (review-plan, verify, done, merge)
- `superpowers` plugin (brainstorming, subagent-driven-development, etc.)
- ClickUp MCP server (configured in `.mcp.json`)
