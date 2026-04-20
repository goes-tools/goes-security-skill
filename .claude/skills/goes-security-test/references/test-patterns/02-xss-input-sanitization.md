# Pattern 02: XSS / Input Sanitization / Sensitive Data Exposure

**Covers:** R3 (Sensitive Data Exposure), R4 (Business Logic Exposure), R5 (XSS Prevention)

```typescript
it('PENTEST: should sanitize user input to prevent XSS injection', async () => {
  await allure.epic('Security');
  await allure.feature('XSS Prevention / Input Sanitization');
  await allure.story('Sanitize all user input before processing or storing');
  await allure.severity('blocker');
  await allure.tag('Pentest');
  await allure.tag('OWASP A03');
  await allure.tag('GOES Checklist R5');
  await allure.description(
    '## Vulnerability Prevented\n' +
    '**Cross-Site Scripting (XSS)** — An attacker injects malicious scripts\n' +
    'through user input fields that get stored or reflected.\n\n' +
    '## Defense Implemented\n' +
    'All user input is sanitized/escaped before storage. HTML tags and\n' +
    'script injections are stripped or encoded.',
  );

  const xssPayloads = [
    '<script>alert("xss")</script>',
    '<img src=x onerror=alert("xss")>',
    '<svg onload=alert("xss")>',
    'javascript:alert("xss")',
    '<iframe src="javascript:alert(1)">',
    '"><script>document.cookie</script>',
  ];

  await allure.parameter('payloads', xssPayloads.length.toString());

  for (const payload of xssPayloads) {
    await allure.step(`Test payload: ${payload.substring(0, 30)}...`, async () => {
      const dto = { name: payload, description: payload };

      // Adapt to the actual service method
      prisma.record.create.mockResolvedValue({ id: 1, ...dto });
      const result = await service.create(dto);

      // The stored value should NOT contain raw script tags
      expect(result.name).not.toContain('<script>');
      expect(result.name).not.toContain('onerror');
      expect(result.name).not.toContain('onload');
    });
  }

  await attach('XSS payloads tested (input)', { payloads: xssPayloads });
  await attach('Defense result (output)', { allPayloadsBlocked: true });
});

it('PENTEST: should not expose sensitive data in source files or responses', async () => {
  await allure.epic('Security');
  await allure.feature('Sensitive Data Exposure');
  await allure.story('No API keys, secrets, or personal data leaked in responses');
  await allure.severity('critical');
  await allure.tag('Pentest');
  await allure.tag('OWASP A02');
  await allure.tag('GOES Checklist R3');
  await allure.tag('GOES Checklist R4');
  await allure.description(
    '## Vulnerability Prevented\n' +
    '**Sensitive Data Exposure** — API responses or source files contain\n' +
    'secrets, API keys, personal IDs, or internal business logic.\n\n' +
    '## Defense Implemented\n' +
    'Responses are filtered through DTOs. Internal fields (password hash,\n' +
    'internal IDs, tokens) are never returned to the client.',
  );

  await allure.step('Prepare: mock a user record with sensitive fields', async () => {
    prisma.user.findUnique.mockResolvedValue({
      id: '1',
      email: 'user@test.com',
      password: '$2b$12$hashedpassword',
      role: 'USER',
      internalNotes: 'VIP customer',
      apiKey: 'sk-secret-key-123',
    });
  });

  const result = await allure.step('Execute: fetch user profile', async () => {
    return service.findOne('1');
  });

  await allure.step('Verify: sensitive fields are NOT in response', async () => {
    expect(result).not.toHaveProperty('password');
    expect(result).not.toHaveProperty('apiKey');
    expect(result).not.toHaveProperty('internalNotes');
    expect(JSON.stringify(result)).not.toMatch(/\$2b\$/); // no bcrypt hash
  });

  await attach('Response fields (output)', { fields: Object.keys(result) });
});
```
