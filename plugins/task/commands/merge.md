---
description: Merge PR and close ClickUp task
argument-hint: [pr-number]
---

# /task:merge $ARGUMENTS

Merge PR and mark ClickUp task complete.

## Workflow

1. **Detect current task**:
   - Extract from branch name: `feat/CU-xxx-*` or `fix/CU-xxx-*`
   - If argument provided: Use as PR number

2. **Delegate to /kas:merge**:
   - Handles: CI check, PR merge, branch deletion, worktree cleanup
   - Wait for completion

3. **On kas:merge success, update ClickUp via clickup-task-agent**:
   ```
   Use Task tool with subagent_type="task:clickup-task-agent":

   "Mark task CU-{id} complete:
   1. Update status to 'complete'"
   ```

4. **Report completion**:
   - Confirm PR merged
   - Confirm ClickUp task closed
   - Note any cleanup performed

## Error Handling

- If kas:merge fails (CI issues, conflicts): Stop, do NOT update ClickUp
- If ClickUp update fails after merge: Warn user, show manual command

## Notes

- Only run this after PR has been reviewed and approved
- CI must be green before merge proceeds
