---
name: generate-security-tests
description: >
  This skill should be used when the user asks to "generate security tests",
  "create pentest specs", "generate GOES tests", "add allure tests",
  "create security specs", "checklist GOES tests", "OWASP tests",
  "security test generator", "generate tests de seguridad",
  "crear tests de seguridad", "generar specs de seguridad",
  or any variation requesting automated security test generation for a NestJS project.
metadata:
  version: "2.0.0"
  coverage: "60 items GOES Checklist + OWASP Top 10 + API Security Top 10"
---

# GOES Security Test Generator

Generate professional security tests with Allure Report for NestJS + Jest projects.
Covers the full GOES Cybersecurity Checklist (60 items), OWASP Top 10, and OWASP API Security Top 10.

## Output Structure

All generated test files MUST go inside `test/security/`:

```
project-root/
  test/
    security/
      auth-service.security.spec.ts
      users-service.security.spec.ts
      roles-guard.security.spec.ts
      helmet-middleware.security.spec.ts
      ...
    allure-setup.ts
  allure-categories.json
  src/
  package.json
```

File naming convention: `<source-name>.security.spec.ts`
Examples:
- `auth.service.ts` → `auth-service.security.spec.ts`
- `roles.guard.ts` → `roles-guard.security.spec.ts`
- `helmet.middleware.ts` → `helmet-middleware.security.spec.ts`
- `users.controller.ts` → `users-controller.security.spec.ts`

## Execution Flow

Follow these 6 steps in order. Do NOT skip any step.

### STEP 1: Analyze the project

Read these files to understand the project structure:

1. `package.json` — framework, ORM (Prisma or TypeORM), Jest config, auth dependencies
2. `src/main.ts` — global middleware, pipes, guards, CORS, helmet (needed for E2E tests)
3. Glob `src/modules/**/*.service.ts` — list all services
4. Glob `src/modules/**/*.controller.ts` — list all controllers
5. Glob `src/**/*.guard.ts` — security guards
6. Glob `src/**/*.middleware.ts` — middleware (rate limit, helmet, cors, etc.)
7. Glob `src/**/*.dto.ts` — DTOs and validation rules
8. Check if `test/security/*.security.spec.ts` or `*.e2e.spec.ts` files already exist
9. Detect ORM: `@prisma/client` → Prisma, `typeorm` / `@nestjs/typeorm` → TypeORM

### STEP 2: Install dependencies (if missing)

Check `package.json` devDependencies. Only install what is missing:

```bash
npm install --save-dev allure-jest allure-js-commons allure jest-html-reporters supertest @types/supertest
```

Note: The `allure` package (v3+) is pure JavaScript — no Java required.
`supertest` is needed for E2E security tests (headers, routes, HTTP methods).

### STEP 3: Configure Jest + Allure

Modify `package.json` jest section:

- Set `"testEnvironment": "allure-jest/node"`
- Add `"testEnvironmentOptions": { "resultsDir": "allure-results" }`
- Add `"setupFiles": ["./test/allure-setup.ts"]`
- Add jest-html-reporters to reporters array
- Set `"testMatch"` to include `"**/test/security/**/*.spec.ts"`

Add npm scripts to `package.json`:
```json
{
  "test:security": "jest --testPathPattern=test/security --verbose",
  "test:allure": "npm run test:security && npx allure generate allure-results --clean -o allure-report && npx allure open allure-report",
  "allure:generate": "npx allure generate allure-results --clean -o allure-report",
  "allure:open": "npx allure open allure-report"
}
```

Create the `test/security/` directory if it does not exist.

Create support files using the exact templates from `references/test-patterns/_support-files.md`:
- `test/allure-setup.ts` — copies categories to allure-results
- `allure-categories.json` — custom failure categories (Pentest, Auth, Access Control, Config, Validation, Files)
- Add `/allure-results`, `/allure-report`, `/reports` to `.gitignore`
- Add `'@typescript-eslint/require-await': 'off'` to test file ESLint overrides

Read `references/test-patterns/_support-files.md` for the complete code of each file.

### STEP 4: Read the service code

Before writing ANY test, read the actual source files. Understand:
- Every public method and what it does
- Validations, guards, error handling
- ORM calls (Prisma or TypeORM) — read `references/test-patterns/_orm-mocks.md` for mock setup
- Auth logic, token handling, password hashing
- Middleware configuration (helmet, cors, rate limiting)
- `main.ts` bootstrap function — needed to replicate setup in E2E tests
- Actual route paths from controllers (`@Get`, `@Post`, `@Patch`, `@Delete` decorators)
- Actual error messages thrown by the service (use those in test assertions)
- Actual DTO field names and validation decorators

### STEP 5: Coverage analysis + Generate tests

#### 5a. Coverage Analysis

Before generating any test, go through ALL 60 items in `references/goes-checklist.md` and classify each one:

- **COVERED** — The project implements this feature → generate a test
- **NOT IMPLEMENTED** — The project does NOT implement this feature → flag it as a security gap
- **NOT APPLICABLE** — The item does not apply to this project type (e.g., file upload items for a project that has no file upload)

Examples:
- R35 (Session Inactivity Timeout): if the project has no idle timeout logic → **NOT IMPLEMENTED** (security gap)
- R57-R60 (File Upload): if the project has no file upload feature → **NOT APPLICABLE**
- R26 (Disposable Email Rejection): if the project has no registration → **NOT APPLICABLE**
- R42 (Cookie Security Flags): if the project does not use cookies → **NOT APPLICABLE**

**IMPORTANT:** Only generate tests for items classified as COVERED.
Items classified as NOT IMPLEMENTED are security gaps — report them to the user as recommendations.
Use `references/test-patterns/_recommendations.md` for the exact recommendation text for each item.
Items classified as NOT APPLICABLE are skipped with justification.

#### 5b. Generate tests

For each COVERED item, create a `.security.spec.ts` file inside `test/security/`.
Map every applicable item from the GOES Checklist (see `references/goes-checklist.md`).

Read these reference files:
1. `references/goes-checklist.md` — complete 60-item checklist with IDs, epics, features and severities
2. `references/test-patterns/_setup.md` — basic spec file structure and adaptation rules
3. `references/test-patterns/_severity-guide.md` — severity assignment and PENTEST rules
4. `references/test-patterns/_orm-mocks.md` — Prisma and TypeORM mock setup
5. `references/test-patterns/_support-files.md` — allure-setup.ts (with environment, executor, history), categories.json, Jest config templates
6. `references/test-patterns/_e2e-setup.md` — E2E test setup for patterns that need a running app
7. `references/test-patterns/_allure-customization.md` — OWASP links, HTML descriptions, ownership, labels, trend tracking
8. `references/test-patterns/_recommendations.md` — Recommendation text for NOT IMPLEMENTED items
9. ALL pattern files `references/test-patterns/01-*.md` through `21-*.md` — 100% coverage of all 60 GOES items

#### Unit tests vs E2E tests

Most patterns are **unit tests** (mock the ORM, test service logic). Some patterns require **E2E tests** (need a running app instance with supertest). See `_e2e-setup.md` for which patterns need E2E and how to set them up.

- Unit test files: `test/security/<name>.security.spec.ts`
- E2E test files: `test/security/<name>.e2e.spec.ts`

#### Test structure (mandatory for every test)

```typescript
import { Test, TestingModule } from '@nestjs/testing';
import * as allure from 'allure-js-commons';

// Evidence helper — define ONCE per file
async function attach(name: string, data: unknown) {
  await allure.attachment(name, JSON.stringify(data, null, 2), {
    contentType: 'application/json',
  });
}

it('descriptive test name', async () => {
  // 1. METADATA (all required)
  await allure.epic('...');        // Area: Security | Authentication | Domain | Configuration | Audit | Infrastructure | Files
  await allure.feature('...');     // Specific feature
  await allure.story('...');       // Concrete scenario
  await allure.severity('...');    // blocker | critical | normal | minor
  await allure.owner('security-team');

  // 2. TAGS — traceability
  await allure.tag('...');         // Category: Pentest, CRUD, Auth, Config
  await allure.tag('...');         // OWASP: OWASP A07
  await allure.tag('...');         // GOES: GOES Checklist R27

  // 3. LABELS — report grouping
  await allure.label('compliance', 'GOES');
  await allure.label('layer', 'api');
  await allure.suite('Security Tests');
  await allure.parentSuite('GOES Compliance');

  // 4. LINKS — clickable OWASP references (see _allure-customization.md for URL list)
  await allure.link('https://owasp.org/Top10/A07_2021-.../', 'OWASP A07:2021');

  // 5. DESCRIPTION — rich HTML with traceability table (see _allure-customization.md)
  await allure.descriptionHtml('...');

  // 6. PARAMETERS (visible inputs)
  await allure.parameter('key', 'value');

  // 7. STEPS (Prepare → Execute → Verify)
  await allure.step('Prepare: ...', async () => { /* mocks */ });
  await attach('Input payload (input)', data);
  const result = await allure.step('Execute: ...', async () => { return service.method(); });
  await allure.step('Verify: ...', async () => { expect(result).toBe(...); });
  await attach('Result (output)', result);
});
```

#### Rules for PENTEST tests

- Prefix: `'PENTEST: ...'` in the it() name
- Epic: `'Security'`
- Tags: `'Pentest'` + `'OWASP Axx'` + `'GOES Checklist Rxx'`
- Description: must include `## Vulnerability Prevented` and `## Defense Implemented`
- Attachments: attacker payload (input) + defense response (output)

### STEP 6: Run tests and generate Allure Report

After generating all test files, you MUST run the tests and generate the report automatically. Do NOT leave this as a manual step for the user.

1. Run the security tests:
   ```bash
   npm run test:security
   ```

2. If any tests fail due to import, config, or ESLint errors (especially `require-await`), fix them and re-run.

3. Generate the Allure report:
   ```bash
   npx allure generate allure-results --clean -o allure-report
   ```

4. Open the Allure report in the browser:
   ```bash
   npx allure open allure-report
   ```

5. Show the **Security Coverage Report** to the user in this exact format:

```
═══════════════════════════════════════════════════════════
  GOES SECURITY COVERAGE REPORT
═══════════════════════════════════════════════════════════

  Tests:     XX passed | XX failed | XX total
  Files:     XX spec files generated

  ✅ COVERED (XX items) — Tests generated
  ─────────────────────────────────────────
  R5  - XSS Prevention                    [OWASP A03]
  R15 - Password Hashing                  [OWASP A02]
  R23 - IDOR Prevention                   [OWASP A01]
  R27 - Brute Force Protection            [OWASP A07]
  ... (list all covered items)

  ⚠️  NOT IMPLEMENTED (XX items) — Security gaps found
  ─────────────────────────────────────────
  R35 - Session Inactivity Timeout        [OWASP A07]
        → Recommendation: Add idle timeout (15-30 min)
          that clears the session and redirects to login.
  R26 - Disposable Email Rejection        [OWASP A07]
        → Recommendation: Add email domain validation
          against a blocklist of disposable providers.
  ... (list all NOT IMPLEMENTED items with recommendations)

  ⬚  NOT APPLICABLE (XX items) — Skipped
  ─────────────────────────────────────────
  R57 - File Extension Validation         (no file upload)
  R58 - File Extension Whitelist          (no file upload)
  R59 - File Size Limit                   (no file upload)
  R60 - File UUID Rename                  (no file upload)
  ... (list all N/A items with reason)

  COVERAGE: XX/60 items tested | XX/60 gaps | XX/60 N/A
═══════════════════════════════════════════════════════════
```

**Important:**
- Steps 1-4 are mandatory and automatic. The user should see the Allure report open in their browser without running any commands manually.
- The coverage report MUST always be shown — it is the most valuable output for the team.
- NOT IMPLEMENTED items are the most important part — each one must include a concrete, actionable recommendation for the developer to fix the gap.

## Important Rules

- ALL code, comments, variable names, and strings MUST be in English
- NEVER generate empty or placeholder tests — every test must have real assertions against real code
- Read the actual service code BEFORE writing tests
- All test files go in `test/security/` with the naming convention `<name>.security.spec.ts`
- If a `.security.spec.ts` file already exists, READ IT FIRST. Compare existing tests against the GOES checklist and ONLY add tests that are missing. Never duplicate existing tests.
- Respect the project's tsconfig.json — if `ignoreDeprecations` is `"5.0"`, do NOT change it
- Each test must be independent — no shared state, no execution order dependency
- Use ORM mocks (Prisma, TypeORM, etc.) — never connect to a real DB in unit tests
- No Java required — the `allure` npm package (v3+) is pure JavaScript
- For E2E tests, replicate the exact main.ts setup (helmet, CORS, pipes, guards, prefix)
- Detect the project's ORM from package.json and use the correct mock pattern from `_orm-mocks.md`
- All patterns are reference examples — adapt endpoints, method names, DTOs, and error messages to the actual code

## Recommended Prompts

These prompts are optimized to trigger the skill correctly. The user can copy-paste any of them:

```
Generate security tests for all services using the goes-security-test skill.
Cover all GOES checklist items with OWASP traceability and Allure Report.
```

```
Generate security tests for the AuthService.
Use the goes-security-test skill and cover all applicable GOES checklist items.
```

```
Generate security tests for the guards and middleware.
Include E2E tests for security headers and HTTP methods.
```

```
Run all security tests and open the Allure report.
```
