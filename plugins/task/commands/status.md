---
description: Check ClickUp task status
argument-hint: [task-id]
---

# /task:status $ARGUMENTS

Check status of a ClickUp task.

## Workflow

1. **Detect task ID**:
   - If argument provided: Use it (CU-xxx, URL, or bare ID)
   - If no argument: Extract from current branch `feat/CU-xxx-*` or `fix/CU-xxx-*`
   - If neither: Ask user for task ID

2. **Parse task ID**:
   - `CU-xxx` → use directly
   - `https://app.clickup.com/t/xxx` → extract ID
   - `#xxx` or just `xxx` → prepend CU-

3. **Fetch status via clickup-task-agent**:
   ```
   Use Task tool with subagent_type="task:clickup-task-agent":

   "Check status of CU-{id}:
   Return: name, status, assignee, due date, last updated"
   ```

4. **Display status to user**

## Notes

- Quick lookup, no modifications made
- Useful for checking task state before starting work
