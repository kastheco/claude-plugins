---
description: Verify code using parallel review and reality assessment
---

# /kas:verify - Verify Implementation

Run code review and reality assessment in parallel, then combine findings.

## Workflow

### 1. Check scope

```bash
git status
git diff --stat
```

If no changes, report "Nothing to verify" and stop.

### 2. Launch parallel reviews

Use the Task tool to launch **two general-purpose agents in parallel** (single message, multiple tool calls):

**Agent 1 - Code Review:**
```
"Run the /kas:review-code workflow on the current git changes.
Read the agent instructions at /home/kas/dev/kas-plugins/agents/code-reviewer.md
Return structured findings with severity levels and overall assessment."
```

**Agent 2 - Reality Assessment:**
```
"Run the /kas:review-reality workflow on the current git changes.
Read the agent instructions at /home/kas/dev/kas-plugins/agents/project-reality-manager.md
Return structured findings with gap analysis and functional state assessment."
```

### 3. Combine findings

After both agents return, merge results:

**Code Quality** (from review-code):
- Critical/High/Medium/Low issues
- Overall assessment

**Reality Assessment** (from review-reality):
- Functional state
- Gap analysis
- Action items

**Overall Verdict** using worst-wins logic:

| Code Review | Reality Assessment | Verdict |
|-------------|-------------------|---------|
| APPROVED | No critical gaps | VERIFIED |
| APPROVED | Has gaps | NEEDS CHANGES |
| NEEDS CHANGES | Any | NEEDS CHANGES |
| REJECTED | Any | BLOCKED |
| Any | Critical gaps | BLOCKED |

### 4. Present and wait

Summarize combined findings and ask for approval:
- If VERIFIED: "All checks pass. Proceed?"
- If NEEDS CHANGES: "Issues found. Approve fixes?"
- If BLOCKED: "Critical blockers. How to proceed?"

Do NOT apply fixes automatically.

## Rules

- Both agents MUST complete before providing verdict
- Use worst-wins logic (conservative approach)
- Never apply fixes without user approval
- Empty diff = exit early, don't launch agents
