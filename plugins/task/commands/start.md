---
description: Start work on a ClickUp task
argument-hint: <task-id-or-url>
---

# /task:start $ARGUMENTS

Start work on ClickUp task with superpowers-driven planning.

## Workflow

### 1. Parse task ID

Parse from argument:
- `CU-xxx` → use directly
- `https://app.clickup.com/t/xxx` → extract ID
- `#xxx` or just `xxx` → prepend CU-

If no argument provided, ask user for task ID.

### 2. Fetch task via clickup-task-agent

```
Use Task tool with subagent_type="task:clickup-task-agent":

"Fetch ClickUp task CU-{id}:
1. Get task details (name, description, acceptance criteria)
2. Capture current status (for potential rollback)
3. Update status to 'in progress'
4. Assign to me

Return: task summary, suggested branch name, original_status"
```

Store `original_status` in conversation context for rollback on abort.

### 3. Enter Plan Mode + Brainstorming

First, use **EnterPlanMode tool** to enter plan mode (creates plan file at `.claude/plans/<name>.md`).

**If already in plan mode:** Skip EnterPlanMode, use existing plan file.

Then invoke brainstorming (which writes to the active plan file):
```
Skill tool with skill="superpowers:brainstorming", args:

"Implement ClickUp task CU-{id}: {task-name}

## ClickUp Context
{description from subagent}

## Acceptance Criteria
{criteria from subagent}

## Branch Convention
Use branch: feat/CU-{id}-{slug} (or fix/ for bugs)"
```

Brainstorming will explore context, ask questions, and output design doc to the plan file.

### 4. Review Loop

Track `iteration_count = 0`.

**REPEAT** (max 5 iterations):

1. Increment `iteration_count`
2. Invoke `Skill("kas:review-plan")`
3. If **APPROVED**: call **ExitPlanMode**, proceed to step 5
4. If `iteration_count >= 5`: force user to choose override or abort (no more revisions)
5. If **NEEDS REVISION** or **BLOCKED**:
   - Present findings to user (quote critical/high issues verbatim)
   - State overall verdict
   - Use AskUserQuestion tool with options:
     - **Revise** - refine plan with feedback
     - **Override** - proceed despite issues (user accepts risk)
     - **Abort** - stop workflow
   - If Revise: re-invoke brainstorming with feedback context, continue loop
   - If Override: call **ExitPlanMode**, proceed to step 5
   - If Abort: revert ClickUp status to `original_status`, stop workflow

**On abort:** Use clickup-task-agent: "Update task CU-{id} status to {original_status}"

**On ExitPlanMode:** User sees plan + review findings together and confirms readiness.

### 5. Implementation Choice

After plan approval, present options:

```
Plan approved. Choose implementation approach:
1. Subagent-Driven (this session) - Fresh subagent per task
2. Parallel Session - Create worktree, implement in new session
```

**Option 1:** Invoke `Skill("superpowers:subagent-driven-development")`

**Option 2:**
1. Invoke `Skill("superpowers:using-git-worktrees")`
2. Save session context to `.claude/task-session.json`:
   ```json
   {
     "taskId": "CU-xxx",
     "branch": "feat/CU-xxx-slug",
     "worktree": ".worktrees/CU-xxx-slug/",
     "planFile": ".claude/plans/xxx.md",
     "status": "awaiting-implementation"
   }
   ```
3. Output cd command and next session prompt
4. STOP (user runs /clear, uses executing-plans in new session)

## Error Handling

| Error | Action |
|-------|--------|
| No argument | Ask for task ID |
| Task fetch fails | Show error, suggest retry |
| Task not found | Show error with valid URL format |
| EnterPlanMode fails | Show error, suggest manual plan creation |
| Already in plan mode | Use existing plan file, continue workflow |
| Brainstorming unavailable | Fall back to `/kas:start` with ClickUp context |
| Brainstorming crashes | Save partial plan, offer retry or manual edit |
| Review agent fails | Ask user: proceed without review (warn) or retry |
| User aborts | Revert ClickUp status, stop workflow |
| Max iterations (5) | Force choice: override or abort |

## Notes

- Worktree creation happens AFTER plan approval
- Task ID stored in session context for later commands
- Plan file path provided by plan mode context
