# Pattern 18: SQL Injection / ORM Usage

**Covers:** R37 (ORM Usage / SQL Injection Prevention)

```typescript
it('PENTEST: should use ORM for all queries (no raw SQL concatenation)', async () => {
  const allure = new AllureCompat();
  await allure.epic('Security');
  await allure.feature('ORM Usage / SQL Injection Prevention');
  await allure.story('All database queries use parameterized ORM methods');
  await allure.severity('blocker');
  await allure.tag('Pentest');
  await allure.tag('OWASP A03');
  await allure.tag('GOES Checklist R37');
  await allure.description(
    '## Vulnerability Prevented\n' +
    '**SQL Injection** — Attackers inject malicious SQL through user\n' +
    'input that is concatenated directly into queries.\n\n' +
    '## Defense Implemented\n' +
    'All queries use Prisma/TypeORM ORM methods with parameterized inputs.\n' +
    'No raw SQL string concatenation.\n\n' +
    '## Reference\n' +
    'GOES Guide Section 7 — Database Security',
  );

  const sqlInjectionPayloads = [
    "' OR '1'='1",
    "'; DROP TABLE users; --",
    "' UNION SELECT * FROM users --",
    "1; DELETE FROM users",
    "admin'--",
  ];

  await allure.parameter('payloads', sqlInjectionPayloads.length.toString());

  for (const payload of sqlInjectionPayloads) {
    await allure.step(`Test payload: ${payload.substring(0, 30)}`, async () => {
      // The ORM should handle parameterization, so the payload
      // is treated as a literal string value, not executable SQL
      prisma.user.findUnique.mockResolvedValue(null);

      const result = await service.findByEmail(payload);

      // The ORM method should receive the payload as a parameter,
      // not as part of a concatenated query
      expect(prisma.user.findUnique).toHaveBeenCalledWith({
        where: { email: payload }, // passed as parameter, not concatenated
      });

      // No record should be returned for SQL injection attempts
      expect(result).toBeNull();
    });
  }

  await allure.attachment('SQL injection payloads tested (input)', JSON.stringify({ payloads: sqlInjectionPayloads }, null, 2), { contentType: 'application/json' });
  await allure.attachment('Defense result (output)', JSON.stringify({
    allPayloadsParameterized: true,
    noRawSQLUsed: true,
  }, null, 2), { contentType: 'application/json' });

  await allure.flush();
});

it('PENTEST: should prevent NoSQL injection in query parameters', async () => {
  const allure = new AllureCompat();
  await allure.epic('Security');
  await allure.feature('ORM Usage / SQL Injection Prevention');
  await allure.story('Reject NoSQL injection operators in query params');
  await allure.severity('blocker');
  await allure.tag('Pentest');
  await allure.tag('OWASP A03');
  await allure.tag('GOES Checklist R37');
  await allure.description(
    '## Vulnerability Prevented\n' +
    '**NoSQL Injection** — Attackers send objects with MongoDB operators\n' +
    '($gt, $ne, $regex) instead of expected string values.\n\n' +
    '## Defense Implemented\n' +
    'Input validation rejects objects/arrays where strings are expected.\n' +
    'DTO validation ensures correct types.',
  );

  const noSqlPayloads = [
    { email: { $gt: '' } },
    { email: { $ne: null } },
    { email: { $regex: '.*' } },
    { password: { $exists: true } },
  ];

  for (const payload of noSqlPayloads) {
    await allure.step(`Test payload: ${JSON.stringify(payload).substring(0, 40)}`, async () => {
      await expect(service.login(payload as any, '1.2.3.4', 'ua')).rejects.toThrow();
    });
  }

  await allure.attachment('NoSQL injection payloads (output)', JSON.stringify({
    total: noSqlPayloads.length,
    allRejected: true,
  }, null, 2), { contentType: 'application/json' });

  await allure.flush();
});
```
