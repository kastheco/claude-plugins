---
name: Kascheck
description: Use when the user asks for a production-readiness audit, project audit, kascheck, or runs /kascheck:audit. Spawns parallel subagents to assess security, code quality, infrastructure, and testing, then produces a scored report with severity-tagged findings and remediation effort estimates. Especially useful for vibe-coded/AI-generated projects.
version: 1.0.0
---

# Kascheck

Analyze projects (especially vibe-coded/AI-generated) to assess production readiness and identify required remediation work.

## Slash Command

**`/kascheck:audit [path] [--detailed]`**

- `path`: Local directory path to analyze (defaults to current directory)
- `--detailed`: Include specific file paths, code examples, and remediation steps

## Output Format

The audit produces a comprehensive report with three views:

1. **Checklist View**: Pass/Warn/Fail for each category
2. **Priority View**: Critical/High/Medium/Low issues with remediation
3. **Effort View**: Quick fix / Moderate / Significant refactor categorization

## Audit Process

### Phase 1: Discovery (Subagent)

Spawn an Explore agent to understand the project:

```
Task("Kascheck: Discover project structure

Analyze this project to understand:
1. Technology stack (languages, frameworks, databases)
2. Project structure (monorepo, single app, microservices)
3. Entry points (main files, routers, handlers)
4. Configuration files present
5. Test file locations
6. Documentation files

Return a structured summary including:
- Primary language(s) and framework(s)
- Architecture pattern detected
- Key directories and their purposes
- Configuration files found
- Test coverage presence

Path: {target_path}", subagent_type="Explore")
```

### Phase 2: Security Audit (Subagent)

```
Task("Kascheck: Security assessment

Analyze security practices in this {framework} project at {target_path}:

**Authentication & Authorization:**
- [ ] Authentication mechanism present and properly implemented
- [ ] Authorization/RBAC enforced on protected routes
- [ ] Session management secure (HttpOnly, Secure, SameSite cookies)
- [ ] JWT implementation secure (proper signing, expiration, refresh)

**Input Validation:**
- [ ] Request validation on all endpoints
- [ ] SQL injection prevention (parameterized queries/ORM)
- [ ] XSS prevention (output encoding, CSP headers)
- [ ] File upload validation (type, size, content)

**Secrets Management:**
- [ ] No hardcoded secrets in code
- [ ] Environment variables for configuration
- [ ] .env files in .gitignore
- [ ] Secrets not logged or exposed in errors

**API Security:**
- [ ] CORS properly configured (not wildcard in production)
- [ ] Rate limiting implemented
- [ ] Security headers present (HSTS, X-Frame-Options, etc.)
- [ ] HTTPS enforced

**OWASP Top 10 2025 Check:**
- [ ] A01: Broken Access Control
- [ ] A02: Security Misconfiguration
- [ ] A03: Software Supply Chain Failures
- [ ] A04: Cryptographic Failures
- [ ] A05: Injection
- [ ] A06: Insecure Design
- [ ] A07: Identification/Authentication Failures
- [ ] A08: Software/Data Integrity Failures
- [ ] A09: Security Logging/Alerting Failures
- [ ] A10: Mishandling Exceptional Conditions

Reference: references/security-checklist.md

Return findings with severity (CRITICAL/HIGH/MEDIUM/LOW) and specific file locations.", subagent_type="general-purpose")
```

### Phase 3: Code Quality Audit (Subagent)

```
Task("Kascheck: Code quality assessment

Analyze code quality in this {framework} project at {target_path}:

**Type Safety:**
- [ ] TypeScript strict mode / Python type hints
- [ ] Proper null/undefined handling
- [ ] No `any` types or untyped functions

**Error Handling:**
- [ ] Consistent error handling pattern
- [ ] No swallowed exceptions
- [ ] Proper error propagation
- [ ] User-friendly error messages (no stack traces leaked)

**Code Organization:**
- [ ] Clear separation of concerns
- [ ] No circular dependencies
- [ ] Reasonable file sizes (<500 lines)
- [ ] Consistent naming conventions

**Testing:**
- [ ] Test framework configured
- [ ] Unit tests present
- [ ] Integration tests present
- [ ] Mocking patterns used appropriately
- [ ] Test coverage configuration

**Documentation:**
- [ ] README with setup instructions
- [ ] API documentation or OpenAPI spec
- [ ] Code comments where necessary
- [ ] Environment variable documentation

**Vibe-Code Red Flags:**
- [ ] Over-abstracted code (2.4x more layers than needed)
- [ ] Inconsistent patterns across files
- [ ] Unused imports/variables
- [ ] Dead code / commented-out code
- [ ] Copy-paste duplication
- [ ] Hallucinated package imports

Reference: references/code-quality-checklist.md
Reference: references/vibe-code-patterns.md

Return findings with effort estimate (QUICK_FIX/MODERATE/SIGNIFICANT).", subagent_type="general-purpose")
```

### Phase 4: Infrastructure Audit (Subagent)

```
Task("Kascheck: Infrastructure assessment

Analyze infrastructure and deployment readiness at {target_path}:

**Developer Tooling:**
- [ ] Linting configured (ESLint, Ruff, etc.)
- [ ] Formatting configured (Prettier, Black, etc.)
- [ ] Pre-commit hooks or CI lint checks
- [ ] IDE configuration (.editorconfig, .vscode)

**CI/CD:**
- [ ] CI pipeline exists (GitHub Actions, etc.)
- [ ] Automated testing in CI
- [ ] Lint/format checks in CI
- [ ] Build verification in CI

**Containerization:**
- [ ] Dockerfile present and optimized
- [ ] Multi-stage builds (if applicable)
- [ ] .dockerignore configured
- [ ] No secrets in Dockerfile

**Deployment:**
- [ ] Deployment configuration (render.yaml, docker-compose, k8s)
- [ ] Environment-specific configs
- [ ] Health check endpoints
- [ ] Graceful shutdown handling

**Observability:**
- [ ] Logging configured (structured logs)
- [ ] Error tracking (Sentry, etc.)
- [ ] Performance monitoring
- [ ] Alerting configuration

**Database:**
- [ ] Migration system in place
- [ ] Connection pooling configured
- [ ] Backup strategy documented
- [ ] No raw credentials in config

Reference: references/infrastructure-checklist.md

Return findings with deployment-blocking issues highlighted.", subagent_type="general-purpose")
```

### Phase 5: Report Generation

After all subagents complete, compile the final report:

```markdown
# Production Readiness Audit Report

**Project:** {project_name}
**Stack:** {detected_stack}
**Audit Date:** {date}
**Overall Score:** {score}/100

## Executive Summary

{2-3 sentence overview of production readiness}

**Verdict:** READY / NEEDS WORK / NOT RECOMMENDED

---

## Checklist Summary

| Category | Status | Issues |
|----------|--------|--------|
| Security | {PASS/WARN/FAIL} | {count} |
| Code Quality | {PASS/WARN/FAIL} | {count} |
| Infrastructure | {PASS/WARN/FAIL} | {count} |
| Testing | {PASS/WARN/FAIL} | {count} |

---

## Critical Issues (Block Deployment)

{List of CRITICAL severity items that MUST be fixed}

---

## High Priority Issues

{List of HIGH severity items}

---

## Medium Priority Issues

{List of MEDIUM severity items}

---

## Low Priority / Recommendations

{List of LOW severity items and nice-to-haves}

---

## Effort Breakdown

### Quick Fixes (< 1 hour each)
{Items that can be resolved quickly}

### Moderate Work (1-4 hours each)
{Items requiring moderate effort}

### Significant Refactoring (> 4 hours)
{Items requiring substantial changes}

---

## Detailed Findings

{Only included if --detailed flag used}

### Security Details
{File paths, code snippets, specific remediation steps}

### Code Quality Details
{Specific files, patterns found, refactoring suggestions}

### Infrastructure Details
{Missing configs, setup steps needed}
```

## Scoring Methodology

Score starts at 100 and deducts per-finding, with a per-severity class cap so no single severity can compound beyond its share. LOW findings are polish/nitpicks and must not dominate an otherwise-green project.

**Deductions (per finding, capped by severity class):**

| Severity | Per-finding | Class cap | Rationale |
|---|---|---|---|
| CRITICAL | -20 | -60 | Each blocks deploy. 3+ is terminal. |
| HIGH | -8 | -24 | Serious but not deploy-blocking. |
| MEDIUM | -3 | -15 | Real debt, slow burn. |
| LOW | -1 | -5 | Polish / nitpicks. Shouldn't compound. |

Floor the final score at 0. No score ceiling adjustments — within each class cap, every finding counts equally.

**Verdicts:**
- **85-100: READY** (ship with monitoring)
- **70-84: READY WITH FOLLOWUPS** (ship but schedule a hardening pass)
- **50-69: NEEDS WORK** (address HIGH+ before deploy)
- **0-49: NOT RECOMMENDED** (significant remediation needed)

**Calibration principle:** The score should reflect ship-readiness, not a checklist tally. 0 CRITICAL + 0 HIGH + green tests + CI gate = at least READY WITH FOLLOWUPS (70+); an otherwise-green project shouldn't drop into NEEDS WORK purely from MED/LOW polish. The per-class caps enforce this.

**Sanity-check examples:**
- 0C / 0H / 0M / 0L → **100** (READY)
- 0C / 0H / 2M / 8L → 100 − 6 − 5 = **89** (READY)
- 0C / 2H / 5M / 5L → 100 − 16 − 15 − 5 = **64** (NEEDS WORK)
- 3C / 6H / 5M / 5L → 100 − 60 − 24 − 15 − 5 = **0** (NOT RECOMMENDED)

## Framework-Specific Checks

The audit adapts based on detected framework:

### Next.js / React
- Server component security
- API route protection
- Environment variable exposure (NEXT_PUBLIC_*)
- Middleware authentication
- Image optimization

### FastAPI / Python
- Dependency injection patterns
- Pydantic validation
- Async/await correctness
- SQLAlchemy/SQLModel usage
- ASGI middleware

### Express / Node.js
- Helmet.js usage
- Express middleware order
- Body parser limits
- Session configuration
- PM2 / cluster setup

### Django
- CSRF protection
- Admin security
- ORM usage
- Settings module structure
- Celery configuration

## Vibe-Code Specific Patterns

Things to specifically watch for in AI-generated code:

1. **Hallucinated Dependencies**: Check if all imports exist in package.json/requirements.txt
2. **Placeholder Code**: Look for TODO, FIXME, "implement this" comments
3. **Inconsistent Patterns**: Different auth patterns in different files
4. **Over-Engineering**: Abstract factories for simple CRUD operations
5. **Missing Error Paths**: Happy path only, no error handling
6. **Hardcoded Values**: Magic strings, hardcoded URLs, inline credentials
7. **No Input Validation**: Direct use of user input without sanitization
8. **Exposed Internals**: Stack traces, debug info in production responses

## Reference Files

- `references/security-checklist.md` - Detailed security checks
- `references/code-quality-checklist.md` - Code quality standards
- `references/infrastructure-checklist.md` - DevOps requirements
- `references/vibe-code-patterns.md` - AI-generated code anti-patterns

## Example Usage

```bash
# Basic audit of current directory
/audit

# Audit specific path
/audit /path/to/project

# Detailed audit with file-level findings
/audit /path/to/project --detailed

# Audit after cloning a repo
git clone https://github.com/user/vibe-coded-app.git
cd vibe-coded-app
/audit --detailed
```
