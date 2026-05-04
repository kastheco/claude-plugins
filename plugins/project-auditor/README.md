# project-auditor

Production-readiness audit for projects — especially vibe-coded / AI-generated codebases. Scores security, code quality, infrastructure, and testing, then surfaces severity-tagged findings and remediation effort estimates.

## What it does

Spawns parallel subagents to assess:

- **Security** — auth, input validation, secrets, CORS, OWASP Top 10
- **Code quality** — type safety, error handling, organization, vibe-code red flags
- **Infrastructure** — CI/CD, containerization, deployment, observability, DB
- **Testing** — coverage, framework, unit/integration presence

Produces a scored report (0-100) with verdict tiers: **READY**, **READY WITH FOLLOWUPS**, **NEEDS WORK**, **NOT RECOMMENDED**.

## Installation

Part of the [kas-claude-plugins](https://github.com/brkastner/kas-claude-plugins) marketplace.

```json
{
  "enabledPlugins": {
    "project-auditor@kas-claude-plugins": true
  }
}
```

## Usage

```bash
# Audit current directory
/project-auditor:audit

# Audit a specific path
/project-auditor:audit /path/to/project

# Detailed audit with file-level findings
/project-auditor:audit /path/to/project --detailed
```

The skill is also auto-invocable — saying "audit this project" or "run a production-readiness check" will trigger it.

## Scoring

Score starts at 100. Per-finding deductions are capped per severity class so a pile of polish nits cannot drag down an otherwise-clean project.

| Severity | Per finding | Class cap |
|---|---|---|
| CRITICAL | -20 | -60 |
| HIGH | -8 | -24 |
| MEDIUM | -3 | -15 |
| LOW | -1 | -5 |

| Score | Verdict |
|---|---|
| 85-100 | READY |
| 70-84 | READY WITH FOLLOWUPS |
| 50-69 | NEEDS WORK |
| 0-49 | NOT RECOMMENDED |

See `skills/project-auditor/SKILL.md` for full audit phases and `references/` for the detailed checklists used by the subagents.
