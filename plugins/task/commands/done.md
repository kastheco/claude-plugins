---
description: Complete work and create PR for ClickUp task
---

# /task:done

Complete work on current ClickUp task: verify, commit, push, create PR, update ClickUp.

## Integration with Superpowers

If implementation used `superpowers:subagent-driven-development`, that skill ends with `superpowers:finishing-a-development-branch`. When user selects PR option, this command handles ClickUp integration.

## Workflow

1. **Detect current task**:
   - Extract from branch name: `feat/CU-xxx-*` or `fix/CU-xxx-*`
   - If not found: Ask user for task ID

2. **Run verification first**:
   - Delegate to `/kas:verify`
   - If BLOCKED or NEEDS CHANGES: Stop, present findings, do NOT proceed
   - If VERIFIED: Continue

3. **Delegate to /kas:done**:
   - Handles: commit, push, verify clean state

4. **Create PR (if none exists)**:
   ```bash
   # Check for existing PR
   gh pr view --json url -q .url 2>/dev/null

   # If no PR, create one
   gh pr create --title "[CU-xxx] {task-title}" --body "..."
   ```

5. **Update ClickUp via clickup-task-agent**:
   ```
   Use Task tool with subagent_type="task:clickup-task-agent":

   "Mark task CU-{id} ready for review:
   1. Update status to 'ready for review'
   2. Add comment with PR URL: {pr_url}
   3. Include brief work summary"
   ```

## Error Handling

- If verification fails: Stop, do NOT update ClickUp
- If kas:done fails: Stop, do NOT update ClickUp
- If PR creation fails: Stop, do NOT update ClickUp
- If ClickUp update fails after success: Warn user, show manual command

6. **Cleanup session file**:
   - Check for `.claude/task-session.json` and delete if exists
   - Only cleaned up AFTER successful completion (preserves recovery context on failure)

## Notes

- This command ensures all quality gates pass before marking ready for review
- ClickUp is only updated if ALL prior steps succeed
- Session file cleanup happens last to preserve recovery context on partial failures
