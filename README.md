# goes-security-skill

Claude skill for generating automated security tests with a **Custom HTML Reporter** for **NestJS + Jest** projects.

No Java, no Allure — pure Node.js reporter that produces a single self-contained HTML file with sidebar navigation (Epic → Feature → Story), severity / OWASP charts, JSON evidence with syntax highlighting, dark theme, real-time search and PDF export.

Covers the **GOES Cybersecurity Checklist** (60 items), **OWASP Top 10**, and **OWASP API Security Top 10**.

---

## Installation

### 1. Clone this repo

```bash
git clone https://github.com/goes-tools/goes-security-skill.git
```

### 2. Copy the `.claude/` folder to your NestJS project

```bash
cp -r goes-security-skill/.claude/ /path/to/your-project/.claude/
```

Your project structure will look like:

```
your-nestjs-project/
  .claude/
    skills/
      goes-security-testing/
        SKILL.md                          <-- instructions for Claude
        reporter/
          html-reporter.js                <-- custom HTML reporter (bundled, JS)
          metadata.ts                     <-- metadata collector + AllureCompat
        references/
          goes-checklist.md               <-- 60 items GOES checklist
          test-patterns/                  <-- 21 code patterns + support files
            _setup.md
            _e2e-setup.md
            _html-reporter-customization.md
            _severity-guide.md
            01-crud-validation.md
            ...
            21-audit-log.md
  src/
  package.json
  ...
```

### 3. Open Claude in your project

Open **Claude Code** or **Claude Cowork** in your project folder.

### 4. Ask Claude to generate the tests

```
Generate security tests for all services using the goes-security-testing skill.
Cover all GOES checklist items with OWASP traceability.
```

Claude reads the skill, analyses your actual code, configures Jest with the bundled reporter, generates the tests, runs them, and writes the HTML report automatically.

---

## What Claude does

When you activate the skill, Claude:

1. Analyses your services, controllers, guards, and middleware.
2. Installs missing dependencies (`ts-jest`, `@types/jest`) — no Java required.
3. Configures Jest to use the bundled reporter directly from `.claude/` (no file duplication).
4. Creates `test/security/` with `.security-html.spec.ts` files, each containing:
   - Metadata (epic, feature, story, severity, tags) via `AllureCompat`.
   - Triple traceability: `GOES Checklist Rxx` + `OWASP Axx` + `OWASP APIxx`.
   - Visible steps (Prepare / Execute / Verify).
   - JSON evidence (attacker payload + defense response).
5. Runs the security tests automatically.
6. Generates `reports/security/security-report.html` — single self-contained HTML file with charts, severity badges, OWASP buckets and full per-test detail.

---

## Usage examples

```
> Generate security tests for the AuthService
> Generate security tests for the UsersService
> Generate security tests for all services
> Generate security tests for the guards and middleware
```

### Run the report

```bash
npm run test:security:html
# Output: reports/security/security-report.html
# Open it directly in any browser — no server needed.
```

The HTML is regenerated on every run. It is fully self-contained (CSS and JS embedded) so you can email it, attach it to a Jira ticket, or zip it as a compliance annex without external dependencies.

---

## Coverage

### GOES Cybersecurity Checklist (60 items)

| Category | Items | Covers |
|----------|-------|--------|
| Web Content | R3-R6 | Sensitive data, XSS, sanitization |
| Server Input/Output | R8-R11 | Generic errors, RBAC, DTO validation |
| Auth & Sessions | R13-R35 | JWT, bcrypt, RS256, brute force, IDOR, token rotation, RBAC |
| Configuration | R37-R55 | CORS, HTTP headers, cookies, rate limit, ORM |
| File Handling | R57-R60 | Magic bytes, whitelist, size limit, UUID rename |

### OWASP Top 10

A01 (Broken Access Control), A02 (Crypto Failures), A03 (Injection), A04 (Insecure Design), A05 (Security Misconfiguration), A07 (Auth Failures), A09 (Logging Failures).

A06 (Vulnerable Components) and A08 (Data Integrity Failures) are out of scope for unit tests; A10 (SSRF) is on the roadmap.

### OWASP API Security Top 10

API1 (Object Level Auth), API2 (Broken Auth), API3 (Property Auth — partial), API4 (Resource Consumption), API5 (Function Level Auth), API8 (Security Misconfiguration).

API6 / API9 / API10 are on the roadmap.

### Test Patterns — 21 patterns, 100% checklist coverage

1. CRUD + DTO Validation (R11)
2. XSS / Input Sanitization (R3, R4, R5)
3. Error Handling (R6, R8, R22)
4. JWT Security (R13, R16, R19, R20)
5. Password Security (R15, R29, R30, R31)
6. Brute Force Protection (R27)
7. Timing Attack Prevention (R14)
8. Replay Attack Detection (R32)
9. RBAC / Privilege Escalation (R9, R24, R34)
10. IDOR Prevention (R23)
11. Session Management (R17, R18, R35)
12. Forced Browsing / Token Per Request (R21, R33)
13. Registration Security (R25, R26, R28)
14. CORS Configuration (R38, R39, R40, R41)
15. Cookie Security (R42, R51)
16. HTTP Security Headers (R44-R50)
17. Debug Mode & HTTP Methods (R43, R52, R53, R54)
18. SQL Injection / ORM (R37)
19. Rate Limiting (R55)
20. File Upload Security (R57, R58, R59, R60)
21. Audit Log (R10)

---

## Reporter customization (v1.1)

The bundled HTML reporter accepts options in the Jest config:

```typescript
[
  path.join(reporterPath, 'html-reporter.js'),
  {
    outputPath: './reports/security/security-report.html',
    projectName: 'GOES — Sistema de Trámites',  // optional, falls back to package.json#name
    reportTitle: 'Reporte de Seguridad GOES',   // optional
  },
],
```

Two environment variables help in CI / parallel runs:

| Variable | Purpose |
|----------|---------|
| `SECURITY_REPORTER_RUN_ID` | Subdir per run under `$TMPDIR/security-html-reporter/<runId>` — required when running Jest with `--maxWorkers > 1` or when several CI jobs share a runner. |
| `SECURITY_REPORTER_TEMP_DIR` | Full override of the metadata tempdir (use it when you need the metadata under your CI workspace). |

See `.claude/skills/goes-security-testing/references/test-patterns/_html-reporter-customization.md` for branding, badge classification, GitHub Actions / GitLab CI snippets, and troubleshooting.

---

## Requirements

- **NestJS** project with Jest configured.
- **Node.js 18+**.
- **No Java required** — uses a custom pure Node.js HTML reporter.

---

## Changelog

### v1.1 (2026-04)

- Reporter project name and report title are now parametrizable; the legacy hard-coded `'Portafolio IT Backend'` placeholder was removed.
- New `SECURITY_REPORTER_RUN_ID` env var scopes metadata per process so parallel Jest workers and concurrent CI jobs no longer mix runs.
- `createStatusChart()` now renders a "No tests" placeholder when zero tests run, instead of producing `NaN` paths in the SVG.
- The 21 reference patterns were migrated from `allure-js-commons` to `AllureCompat` (bundled). Existing `await allure.epic(...)`, `await allure.step(...)` etc. work unchanged; the only mandatory addition is `await allure.flush()` at the end of each `it(...)`.
- New `attachFor(allure)` helper exported from `metadata.ts` so legacy `attach('name', data)` calls keep their short shape.
- New customization guide in `references/test-patterns/_html-reporter-customization.md` (the old `_allure-customization.md` is deprecated and kept as a stub).
- Path typo fixed: `.claude/skills/goes-security-test/` → `.claude/skills/goes-security-testing/`.

### v1.0

- Initial release with HTML reporter, 21 patterns, 60 GOES checklist items.

---

## Contributing

PRs are welcome for:

- Adding new test patterns in `references/test-patterns/` (especially the gaps: A10 SSRF, API3 mass assignment, API6 business flows, R12 payload size limit).
- Updating the checklist in `references/goes-checklist.md`.
- Improving skill instructions in `SKILL.md`.
- Adding support for other frameworks (Express, Fastify, etc.).

### How to contribute

1. Fork the repo.
2. Create a branch: `git checkout -b feature/new-pattern`.
3. Make changes.
4. Open a PR with a description of what was added or changed.

---

## License

Internal use — Government of El Salvador (GOES).
