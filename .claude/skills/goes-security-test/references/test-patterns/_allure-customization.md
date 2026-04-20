# Allure Report Customization — Professional Setup

These customizations make the Allure report look professional with environment info,
OWASP links, test ownership, rich descriptions, and trend tracking.

## 1. Environment Info Widget

Update `test/allure-setup.ts` to include project info in the report's Environment widget:

```typescript
import * as fs from 'fs';
import * as path from 'path';

const resultsDir = path.resolve(__dirname, '..', 'allure-results');
const categoriesSource = path.resolve(__dirname, '..', 'allure-categories.json');
const categoriesDest = path.resolve(resultsDir, 'categories.json');

// Ensure allure-results directory exists
if (!fs.existsSync(resultsDir)) {
  fs.mkdirSync(resultsDir, { recursive: true });
}

// Copy categories
if (fs.existsSync(categoriesSource)) {
  fs.copyFileSync(categoriesSource, categoriesDest);
}

// Write environment info — appears in the "Environment" widget on the dashboard
const packageJson = JSON.parse(
  fs.readFileSync(path.resolve(__dirname, '..', 'package.json'), 'utf-8'),
);
const envContent = [
  `Project.Name=${packageJson.name || 'Unknown'}`,
  `Project.Version=${packageJson.version || '0.0.0'}`,
  `Test.Date=${new Date().toISOString().split('T')[0]}`,
  `Framework=NestJS + Jest`,
  `Node.Version=${process.version}`,
  `Environment=${process.env.NODE_ENV || 'test'}`,
  `Coverage=GOES Checklist + OWASP Top 10 + API Security Top 10`,
].join('\n');
fs.writeFileSync(path.join(resultsDir, 'environment.properties'), envContent);

// Write executor info — shows who/what ran the tests
const executor = {
  name: process.env.CI ? 'CI Pipeline' : 'Local Developer',
  type: process.env.CI ? 'github' : 'local',
  buildName: `Security Tests — ${new Date().toISOString().split('T')[0]}`,
};
fs.writeFileSync(
  path.join(resultsDir, 'executor.json'),
  JSON.stringify(executor, null, 2),
);
```

## 2. OWASP Links in Tests

Add clickable links to OWASP vulnerability pages in each test:

```typescript
import * as allure from 'allure-js-commons';

it('PENTEST: should prevent IDOR access', async () => {
  await allure.epic('Security');
  await allure.feature('IDOR Prevention');
  await allure.severity('blocker');

  // Clickable links — appear in the "Links" section of each test
  await allure.link('https://owasp.org/Top10/A01_2021-Broken_Access_Control/', 'OWASP A01:2021');
  await allure.link('https://owasp.org/API-Security/editions/2023/en/0xa1-broken-object-level-authorization/', 'OWASP API1:2023');

  // ... test code
});
```

### OWASP Link Reference

Use these URLs in the `allure.link()` calls:

| Tag | URL |
|-----|-----|
| OWASP A01 | `https://owasp.org/Top10/A01_2021-Broken_Access_Control/` |
| OWASP A02 | `https://owasp.org/Top10/A02_2021-Cryptographic_Failures/` |
| OWASP A03 | `https://owasp.org/Top10/A03_2021-Injection/` |
| OWASP A04 | `https://owasp.org/Top10/A04_2021-Insecure_Design/` |
| OWASP A05 | `https://owasp.org/Top10/A05_2021-Security_Misconfiguration/` |
| OWASP A07 | `https://owasp.org/Top10/A07_2021-Identification_and_Authentication_Failures/` |
| OWASP A09 | `https://owasp.org/Top10/A09_2021-Security_Logging_and_Monitoring_Failures/` |
| OWASP A10 | `https://owasp.org/Top10/A10_2021-Server-Side_Request_Forgery_%28SSRF%29/` |
| OWASP API1 | `https://owasp.org/API-Security/editions/2023/en/0xa1-broken-object-level-authorization/` |
| OWASP API2 | `https://owasp.org/API-Security/editions/2023/en/0xa2-broken-authentication/` |
| OWASP API3 | `https://owasp.org/API-Security/editions/2023/en/0xa3-broken-object-property-level-authorization/` |
| OWASP API4 | `https://owasp.org/API-Security/editions/2023/en/0xa4-unrestricted-resource-consumption/` |
| OWASP API5 | `https://owasp.org/API-Security/editions/2023/en/0xa5-broken-function-level-authorization/` |

## 3. Test Ownership

Assign ownership to each test so the report shows who is responsible:

```typescript
it('PENTEST: should hash passwords with bcrypt', async () => {
  await allure.owner('security-team');
  await allure.epic('Security');
  // ... test code
});
```

For a project team, set a default owner in the describe block:

```typescript
describe('AuthService — Security Tests', () => {
  beforeEach(async () => {
    // Default owner for all tests in this file
    await allure.owner('security-team');
  });

  // Tests inherit the owner...
});
```

## 4. Rich HTML Descriptions

Use `allure.descriptionHtml()` for formatted descriptions with tables:

```typescript
await allure.descriptionHtml(`
  <h2>IDOR Prevention — R23</h2>

  <h3>Vulnerability</h3>
  <p><strong>Insecure Direct Object Reference</strong> — An attacker modifies
  the ID in the URL to access another user's resources.</p>

  <h3>GOES Checklist Mapping</h3>
  <table style="border-collapse: collapse; width: 100%;">
    <tr style="background: #f0f0f0;">
      <th style="padding: 8px; border: 1px solid #ddd; text-align: left;">ID</th>
      <th style="padding: 8px; border: 1px solid #ddd; text-align: left;">Item</th>
      <th style="padding: 8px; border: 1px solid #ddd; text-align: left;">Severity</th>
    </tr>
    <tr>
      <td style="padding: 8px; border: 1px solid #ddd;">R23</td>
      <td style="padding: 8px; border: 1px solid #ddd;">IDOR Prevention</td>
      <td style="padding: 8px; border: 1px solid #ddd;">🔴 Blocker</td>
    </tr>
  </table>

  <h3>Defense</h3>
  <p>Verify resource ownership on every request. Never trust URL IDs.</p>
`);
```

## 5. Additional Labels

Add extra metadata visible in the Allure dashboard:

```typescript
// Standard labels
await allure.label('compliance', 'GOES');
await allure.label('layer', 'api');
await allure.label('component', 'authentication');

// These affect how tests are grouped in the report:
await allure.suite('Security Tests');
await allure.parentSuite('GOES Compliance');
await allure.subSuite('Authentication');
```

## 6. Trend Tracking (History)

To enable the "Trend" graph that shows test results over time, preserve the
history folder between test runs. Add this to `test/allure-setup.ts`:

```typescript
// Preserve history from previous report (if it exists)
const reportHistoryDir = path.resolve(__dirname, '..', 'allure-report', 'history');
const resultsHistoryDir = path.resolve(resultsDir, 'history');

if (fs.existsSync(reportHistoryDir)) {
  if (!fs.existsSync(resultsHistoryDir)) {
    fs.mkdirSync(resultsHistoryDir, { recursive: true });
  }
  const historyFiles = fs.readdirSync(reportHistoryDir);
  for (const file of historyFiles) {
    fs.copyFileSync(
      path.join(reportHistoryDir, file),
      path.join(resultsHistoryDir, file),
    );
  }
}
```

And add this npm script for running with history:

```json
{
  "test:allure": "npm run test:security && npx allure generate allure-results --clean -o allure-report && npx allure open allure-report"
}
```

After the first run, subsequent runs will show the trend graph on the dashboard.

## 7. Complete Test Template with All Customizations

```typescript
it('PENTEST: should prevent access to another user resource (IDOR)', async () => {
  // Metadata
  await allure.epic('Security');
  await allure.feature('IDOR Prevention');
  await allure.story('User cannot access resources belonging to another user');
  await allure.severity('blocker');
  await allure.owner('security-team');

  // Labels
  await allure.label('compliance', 'GOES');
  await allure.label('layer', 'api');
  await allure.suite('Security Tests');
  await allure.parentSuite('GOES Compliance');
  await allure.subSuite('Access Control');

  // Tags (visible in dashboard filters)
  await allure.tag('Pentest');
  await allure.tag('OWASP A01');
  await allure.tag('OWASP API1');
  await allure.tag('GOES Checklist R23');

  // Links (clickable in test details)
  await allure.link('https://owasp.org/Top10/A01_2021-Broken_Access_Control/', 'OWASP A01:2021');

  // Rich HTML description
  await allure.descriptionHtml(`
    <h2>Vulnerability Prevented</h2>
    <p><strong>IDOR (Insecure Direct Object Reference)</strong> — An attacker
    modifies the ID in the URL to access another user's resources.</p>
    <h2>Defense Implemented</h2>
    <p>Verify that the requested resource belongs to the authenticated user
    before returning it.</p>
    <h2>Traceability</h2>
    <table style="border-collapse: collapse;">
      <tr style="background: #f0f0f0;">
        <th style="padding: 6px; border: 1px solid #ddd;">Standard</th>
        <th style="padding: 6px; border: 1px solid #ddd;">ID</th>
      </tr>
      <tr>
        <td style="padding: 6px; border: 1px solid #ddd;">GOES Checklist</td>
        <td style="padding: 6px; border: 1px solid #ddd;">R23</td>
      </tr>
      <tr>
        <td style="padding: 6px; border: 1px solid #ddd;">OWASP Top 10</td>
        <td style="padding: 6px; border: 1px solid #ddd;">A01:2021</td>
      </tr>
      <tr>
        <td style="padding: 6px; border: 1px solid #ddd;">OWASP API Security</td>
        <td style="padding: 6px; border: 1px solid #ddd;">API1:2023</td>
      </tr>
    </table>
  `);

  // Parameters
  await allure.parameter('attacker_userId', 'user-1');
  await allure.parameter('victim_userId', 'user-2');
  await allure.parameter('resourceId', 'resource-1');

  // Steps + Assertions
  await allure.step('Prepare: resource belongs to user-2', async () => {
    prisma.resource.findUnique.mockResolvedValue({ id: 'resource-1', userId: 'user-2' });
  });

  await attach('Attacker attempt (input)', {
    attackerUserId: 'user-1', resourceId: 'resource-1', ownerUserId: 'user-2',
  });

  await allure.step('Execute: attacker requests another user\'s resource', async () => {
    await expect(service.findOne('resource-1', 'user-1')).rejects.toThrow('Unauthorized');
  });

  await attach('Defense executed (output)', {
    accessBlocked: true, reason: 'Resource does not belong to the authenticated user',
  });
});
```
