---
description: Tiered code verification with early-exit on issues
---

# /kas:verify - Verify Implementation

Tiered verification with early-exit. Static analysis first, dynamic checks only if clean.

## Workflow

### 1. Check scope

```bash
git status
git diff --stat
```

If no changes, report "Nothing to verify" and stop.

### 2. Tier 1: Static Analysis (parallel)

Launch **five agents in parallel** using the Task tool:

**Agent 1 - Code Review (kas):**
```
"Run code review on the current git changes.
Read agent instructions at /home/kas/dev/kas-plugins/agents/code-reviewer.md
Return structured findings with severity levels and overall assessment (APPROVED/NEEDS CHANGES/REJECTED)."
```

**Agent 2 - Silent Failure Hunter (pr-review-toolkit):**
```
"Check for silent failures in error handling.
Look for: empty catch blocks, swallowed exceptions, missing error propagation, try/catch without proper handling.
Return findings with severity levels."
```

**Agent 3 - Type Design Analyzer (pr-review-toolkit):**
```
"Review type definitions, interfaces, and schemas in the changes.
Check for: overly permissive types, missing null checks, inconsistent naming, type safety issues.
Return findings with severity levels."
```

**Agent 4 - Comment Analyzer (pr-review-toolkit):**
```
"Analyze comments, docstrings, and documentation in the changes.
Check for: outdated comments, missing docs on public APIs, misleading comments, TODO/FIXME items.
Return findings with severity levels."
```

**Agent 5 - Test Analyzer (pr-review-toolkit):**
```
"Analyze test coverage for the changes.
Check for: missing tests, inadequate assertions, untested edge cases, test quality.
Return findings with coverage assessment."
```

**Tier 1 Exit Conditions:**

| Condition | Verdict | Action |
|-----------|---------|--------|
| Any critical/high issues | BLOCKED | Stop, present findings |
| Any medium issues | NEEDS CHANGES | Stop, present findings |
| All clean | Continue | Proceed to Tier 2 |

If Tier 1 fails, do NOT proceed to Tier 2. Present combined findings and ask for direction.

### 3. Tier 2: Dynamic Assessment (only if Tier 1 clean)

Launch **one agent**:

**Agent 6 - Reality Assessment (kas):**
```
"Run reality assessment on the current git changes.
Read agent instructions at /home/kas/dev/kas-plugins/agents/project-reality-manager.md
Return gap analysis and functional state assessment."
```

**Tier 2 Exit Conditions:**

| Condition | Verdict | Action |
|-----------|---------|--------|
| Gaps or issues found | NEEDS CHANGES | Stop, present findings |
| Severe gaps | BLOCKED | Stop, present findings |
| Clean | VERIFIED | Proceed to Tier 3 |

### 4. Tier 3: Polish (only if VERIFIED)

Launch **one agent**:

**Agent 7 - Code Simplifier (pr-review-toolkit):**
```
"Review the changes for simplification opportunities.
Look for: over-engineering, unnecessary abstractions, code that could be simpler.
Return suggestions (these are optional improvements, not blockers)."
```

Simplification suggestions are optional - they don't change the VERIFIED verdict.

### 5. Present and wait

Summarize findings based on exit tier:

- **VERIFIED** (reached Tier 3): "All checks pass. [Simplification suggestions if any]. Proceed?"
- **NEEDS CHANGES** (Tier 1 or 2): "Issues found at [tier]. Approve fixes?"
- **BLOCKED** (Tier 1 or 2): "Critical blockers at [tier]. How to proceed?"

Do NOT apply fixes automatically.

## Rules

- Tiers execute sequentially with early-exit
- Tier 1 agents run in parallel for speed
- Tier 2 only runs if Tier 1 has zero issues
- Tier 3 only runs if Tier 2 returns VERIFIED
- Never apply fixes without user approval
- Empty diff = exit early, don't launch agents
