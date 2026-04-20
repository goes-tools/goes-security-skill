# goes-security-skill

Claude skill for generating automated security tests with a **Custom HTML Reporter** for **NestJS + Jest** projects.

No Java, no Allure — pure Node.js reporter that generates a self-contained HTML file with sidebar navigation, charts, and evidence highlighting.

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
      goes-security-test/
        SKILL.md                          <-- instructions for Claude
        reporter/
          html-reporter.js                <-- custom HTML reporter (bundled)
          metadata.ts                     <-- metadata collector (bundled)
        references/
          goes-checklist.md               <-- 60 items GOES checklist
          test-patterns/                  <-- 21 code patterns (100% coverage)
            _setup.md
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
Generate security tests for all services using the goes-security-test skill.
Cover all GOES checklist items with OWASP traceability.
```

Claude reads the skill, analyzes your actual code, configures Jest with the bundled reporter, generates the tests, runs them, and generates the HTML report automatically.

---

## What Claude does

When you activate the skill, Claude:

1. Analyzes your services, controllers, guards, and middleware
2. Installs dependencies if missing (`ts-jest`, `@types/jest`) — no Java required
3. Configures Jest to use the bundled reporter directly from `.claude/` (no file duplication)
4. Creates `test/security/` folder with `.security-html.spec.ts` files containing:
   - Metadata (epic, feature, story, severity, tags) via `AllureCompat`
   - Triple traceability: `GOES Checklist Rxx` + `OWASP Axx` + `OWASP APIxx`
   - Visible steps (Prepare / Execute / Verify)
   - JSON evidence (attacker payload + defense response)
5. Runs all security tests automatically
6. Generates the HTML report at `reports/security/security-report.html`

---

## Usage examples

```
> Generate security tests for the AuthService
> Generate security tests for the UsersService
> Generate security tests for all services
> Generate security tests for the guards and middleware
```

### View the report

```bash
npm run test:security
# Report generated at: reports/security/security-report.html
# Open it in your browser — no extra commands needed
```

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

A01 (Broken Access Control), A02 (Crypto Failures), A03 (Injection), A05 (Security Misconfiguration), A07 (Auth Failures), A09 (Logging Failures)

### OWASP API Security Top 10

API1 (Object Level Auth), API2 (Broken Auth), API3 (Property Auth), API4 (Resource Consumption), API5 (Function Level Auth), API8 (Security Misconfiguration)

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

## Requirements

- **NestJS** project with Jest configured
- **Node.js 18+**
- **No Java required** — uses a custom pure Node.js HTML reporter

---

## Contributing

PRs are welcome for:

- Adding new test patterns in `references/test-patterns/`
- Updating the checklist in `references/goes-checklist.md`
- Improving skill instructions in `SKILL.md`
- Adding support for other frameworks (Express, Fastify, etc.)

### How to contribute

1. Fork the repo
2. Create a branch: `git checkout -b feature/new-pattern`
3. Make changes
4. PR with description of what was added/changed

---

## License

Internal use — Government of El Salvador (GOES)
