---
description: Start work on a ClickUp task
argument-hint: <task-id-or-url>
---

# /task:start $ARGUMENTS

Start work on ClickUp task.

## Workflow

1. **Parse task ID** from argument:
   - `CU-xxx` → use directly
   - `https://app.clickup.com/t/xxx` → extract ID
   - `#xxx` or just `xxx` → prepend CU-

2. **Fetch task via clickup-task-agent**:
   ```
   Use Task tool with subagent_type="clickup-tasks:clickup-task-agent":

   "Fetch ClickUp task CU-{id}:
   1. Get task details (name, description, acceptance criteria)
   2. Update status to 'in progress'
   3. Assign to me

   Return concise summary with suggested branch name."
   ```

3. **Delegate to /kas:start** with context:
   ```
   Implement ClickUp task CU-{id}: {task-name}

   ## ClickUp Context
   {description from subagent}

   ## Acceptance Criteria
   {criteria from subagent}

   ## Branch Convention
   Use branch: feat/CU-{id}-{slug} (or fix/ for bugs)
   Worktree path: .worktrees/CU-{id}-{slug}/
   ```

## Error Handling

- If no argument provided: Ask user for task ID
- If task fetch fails: Show error, suggest retry
- If task not found: Show error with task URL format

## Notes

- Worktree creation happens AFTER plan approval (handled by kas:start)
- Task ID stored in session context for later commands
