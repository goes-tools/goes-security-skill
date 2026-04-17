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
  version: "1.0.0"
  coverage: "60 items Checklist GOES + OWASP Top 10 + API Security Top 10"
---

# GOES Security Test Generator

Generate professional security tests with Allure Report for NestJS + Jest projects.
Covers the full GOES Cybersecurity Checklist (60 items), OWASP Top 10, and OWASP API Security Top 10.

## Execution Flow

Follow these 6 steps in order. Do NOT skip any step.

### STEP 1: Analyze the project

Read these files to understand the project structure:

1. `package.json` — framework, ORM, Jest config, auth dependencies
2. Glob `src/modules/**/*.service.ts` — list all services
3. Glob `src/modules/**/*.controller.ts` — list all controllers
4. Glob `src/**/*.guard.ts` — security guards
5. Glob `src/**/*.middleware.ts` — middleware (rate limit, helmet, cors, etc.)
6. Check if `.spec.ts` files already exist

### STEP 2: Install dependencies (if missing)

Check `package.json` devDependencies. Only install what is missing:

```bash
npm install --save-dev allure-jest allure-js-commons allure-commandline jest-html-reporters
```

### STEP 3: Configure Jest + Allure

Modify `package.json` jest section:

- Set `"testEnvironment": "allure-jest/node"`
- Add `"testEnvironmentOptions": { "resultsDir": "allure-results" }`
- Add `"setupFiles": ["../test/allure-setup.ts"]`
- Add jest-html-reporters to reporters array

Add npm scripts: `test:allure`, `allure:generate`, `allure:open`.

Create support files:
- `test/allure-setup.ts` — copies categories to allure-results
- `allure-categories.json` — custom failure categories (Pentest, Validation, Config, Product)
- Add `/allure-results`, `/allure-report`, `/reports` to `.gitignore`
- Add `'@typescript-eslint/require-await': 'off'` to test file ESLint overrides

### STEP 4: Read the service code

Before writing ANY test, read the actual `.service.ts` file. Understand:
- Every public method and what it does
- Validations, guards, error handling
- ORM calls (Prisma, TypeORM, etc.)
- Auth logic, token handling, password hashing

### STEP 5: Generate tests

For each service/controller, create a `.spec.ts` file following the test structure below.
Map every applicable item from the GOES Checklist (see `references/goes-checklist.md`).

Read `references/goes-checklist.md` for the complete 60-item checklist with IDs, epics, features and severities.
Read `references/test-patterns.md` for the exact code patterns to use.

#### Test structure (mandatory for every test)

```typescript
import * as allure from 'allure-js-commons';

// Helper — define ONCE per file
async function attach(name: string, data: unknown) {
  await allure.attachment(name, JSON.stringify(data, null, 2), {
    contentType: 'application/json',
  });
}

it('descriptive test name', async () => {
  // 1. METADATA (all required)
  await allure.epic('...');        // Area: Seguridad | Autenticacion | Dominio | Configuracion | Auditoria | Infraestructura | Archivos
  await allure.feature('...');     // Specific feature
  await allure.story('...');       // Concrete scenario
  await allure.severity('...');    // blocker | critical | normal | minor
  await allure.tag('...');         // Category: Pentest, CRUD, Auth, Config
  await allure.tag('...');         // Traceability: OWASP A07, GOES Checklist R27
  await allure.description('...'); // Markdown: ## Objetivo / ## Vulnerabilidad / ## Defensa

  // 2. PARAMETERS (visible inputs)
  await allure.parameter('key', 'value');

  // 3. STEPS (Preparar → Ejecutar → Verificar)
  await allure.step('Preparar: ...', async () => { /* mocks */ });
  await attach('Input (input)', data);
  const result = await allure.step('Ejecutar: ...', async () => { return service.method(); });
  await allure.step('Verificar: ...', async () => { expect(result).toBe(...); });
  await attach('Resultado (output)', result);
});
```

#### Rules for PENTEST tests

- Prefix: `'PENTEST: ...'` in the it() name
- Epic: `'Seguridad'`
- Tags: `'Pentest'` + `'OWASP Axx'` + `'GOES Checklist Rxx'`
- Description: must include `## Vulnerabilidad que previene` and `## Defensa implementada`
- Attachments: attacker payload (input) + defense response (output)

### STEP 6: Verify

1. Run `npm test` — all tests must pass
2. Fix ESLint errors if any (especially `require-await` in test files)
3. Show summary: total tests, checklist items covered, OWASP items covered

## Important Rules

- NEVER generate empty or placeholder tests — every test must have real assertions against real code
- Read the actual service code BEFORE writing tests
- Comments in code: NO accents (ASCII only). Allure strings (descriptions, steps): YES accents
- Respect the project's tsconfig.json — if `ignoreDeprecations` is `"5.0"`, do NOT change it
- Each test must be independent — no shared state, no execution order dependency
- Use ORM mocks (Prisma, TypeORM, etc.) — never connect to a real DB in unit tests
