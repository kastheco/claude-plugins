# Changelog

All notable changes to the project-auditor plugin will be documented in this file.

## [1.0.0] - 2026-05-03

### Added

- Initial release as a standalone plugin in the `kas-claude-plugins` marketplace.
- `/project-auditor:audit [path] [--detailed]` command.
- `project-auditor` skill (auto-invocable on audit/production-readiness requests).
- 5-phase audit pipeline: discovery → security → code quality → infrastructure → report.
- Refined scoring methodology with per-severity class caps (CRITICAL -60, HIGH -24, MEDIUM -15, LOW -5) and 4-tier verdicts: READY / READY WITH FOLLOWUPS / NEEDS WORK / NOT RECOMMENDED.
- Reference checklists: security, code quality, infrastructure, vibe-code anti-patterns.
- Framework-specific checks for Next.js/React, FastAPI, Express, Django.
