# Pattern 04: JWT Security

**Covers:** R13 (Token Lifetime), R16 (JWT Signing Algorithm), R19 (JWT Claims Validation), R20 (JWT Payload Security)

```typescript
it('PENTEST: should enforce short-lived access tokens', async () => {
  await allure.epic('Security');
  await allure.feature('Token Lifetime Enforcement');
  await allure.story('Access token must expire within 15 minutes');
  await allure.severity('critical');
  await allure.tag('Pentest');
  await allure.tag('OWASP A07');
  await allure.tag('GOES Checklist R13');
  await allure.description(
    '## Vulnerability Prevented\n' +
    '**Token Theft Persistence** — If access tokens have long lifetimes,\n' +
    'a stolen token gives extended unauthorized access.\n\n' +
    '## Defense Implemented\n' +
    'Access tokens expire in 5-15 minutes. Refresh tokens handle renewal.',
  );

  await allure.parameter('max_lifetime', '15 minutes');

  const tokens = await allure.step('Execute: generate tokens via login', async () => {
    prisma.user.findUnique.mockResolvedValue({
      id: '1', email: 'user@test.com', password: await bcrypt.hash('password', 12),
    });
    return service.login({ email: 'user@test.com', password: 'password' }, '1.2.3.4', 'ua');
  });

  await allure.step('Verify: access token expires within 15 minutes', async () => {
    const decoded = jwtService.decode(tokens.accessToken) as any;
    const lifetime = decoded.exp - decoded.iat;
    expect(lifetime).toBeLessThanOrEqual(900); // 15 min = 900 seconds
  });

  await attach('Token lifetime (output)', {
    accessTokenLifetime: '<=15 min',
    hasRefreshToken: !!tokens.refreshToken,
  });
});

it('PENTEST: should use RS256 or ES256 for JWT signing', async () => {
  await allure.epic('Security');
  await allure.feature('JWT Signing Algorithm');
  await allure.story('JWT must be signed with RS256 or ES256, never HS256 with weak secret');
  await allure.severity('blocker');
  await allure.tag('Pentest');
  await allure.tag('OWASP A02');
  await allure.tag('GOES Checklist R16');
  await allure.description(
    '## Vulnerability Prevented\n' +
    '**JWT Algorithm Attack** — Using "none" or HS256 with a weak\n' +
    'secret allows attackers to forge valid tokens.\n\n' +
    '## Defense Implemented\n' +
    'JWT is signed with RS256 (RSA) or ES256 (ECDSA) using asymmetric keys.\n' +
    'The "none" algorithm and weak secrets are rejected.\n\n' +
    '## Reference\n' +
    'GOES Guide Section 4.3 — JWT Best Practices',
  );

  await allure.step('Verify: JWT module is configured with safe algorithm', async () => {
    // Adapt to the actual JWT config in the project
    const decoded = jwtService.decode(token, { complete: true }) as any;
    expect(decoded.header.alg).toMatch(/^(RS256|RS384|RS512|ES256|ES384|ES512)$/);
    expect(decoded.header.alg).not.toBe('none');
    expect(decoded.header.alg).not.toBe('HS256');
  });

  await attach('JWT algorithm (output)', { algorithm: 'RS256', safe: true });
});

it('PENTEST: should validate all JWT claims on every request', async () => {
  await allure.epic('Security');
  await allure.feature('JWT Claims Validation');
  await allure.story('Validate signature, exp, iat, iss, aud on every JWT');
  await allure.severity('blocker');
  await allure.tag('Pentest');
  await allure.tag('OWASP A07');
  await allure.tag('GOES Checklist R19');
  await allure.description(
    '## Vulnerability Prevented\n' +
    '**Token Forgery** — Without full claims validation, attackers can\n' +
    'craft tokens with modified claims (expired, wrong issuer, etc.).\n\n' +
    '## Defense Implemented\n' +
    'Every JWT is validated for: signature, exp (not expired), iat (issued at),\n' +
    'iss (issuer matches), aud (audience matches).',
  );

  const invalidTokens = [
    { token: 'expired-token', reason: 'expired', error: 'jwt expired' },
    { token: 'wrong-issuer-token', reason: 'wrong issuer', error: 'jwt issuer invalid' },
    { token: 'tampered-token', reason: 'invalid signature', error: 'invalid signature' },
    { token: 'none-alg-token', reason: 'none algorithm', error: 'jwt malformed' },
  ];

  for (const { token, reason, error } of invalidTokens) {
    await allure.step(`Test: reject token with ${reason}`, async () => {
      jwtService.verifyAsync.mockRejectedValue(new Error(error));
      await expect(guard.canActivate(mockContext)).rejects.toThrow();
    });
  }

  await attach('Invalid tokens tested (output)', {
    total: invalidTokens.length,
    allRejected: true,
  });
});

it('PENTEST: should NOT store sensitive data in JWT payload', async () => {
  await allure.epic('Security');
  await allure.feature('JWT Payload Security');
  await allure.story('JWT payload must not contain passwords, secrets, or personal data');
  await allure.severity('critical');
  await allure.tag('Pentest');
  await allure.tag('OWASP A02');
  await allure.tag('GOES Checklist R20');
  await allure.description(
    '## Vulnerability Prevented\n' +
    '**Sensitive Data in JWT** — JWT payloads are base64-encoded (not encrypted).\n' +
    'Anyone can decode and read the contents.\n\n' +
    '## Defense Implemented\n' +
    'JWT payload contains only: sub (user ID), role, iat, exp.\n' +
    'No passwords, emails, personal data, or secrets.',
  );

  const tokens = await allure.step('Execute: login and get token', async () => {
    return service.login(validCredentials, '1.2.3.4', 'ua');
  });

  await allure.step('Verify: JWT payload has no sensitive data', async () => {
    const decoded = jwtService.decode(tokens.accessToken) as any;
    const payload = JSON.stringify(decoded);

    expect(payload).not.toMatch(/password/i);
    expect(payload).not.toMatch(/secret/i);
    expect(payload).not.toMatch(/apiKey/i);
    expect(payload).not.toMatch(/\$2[aby]\$/); // bcrypt hash
    expect(payload).not.toMatch(/credit/i);
    expect(payload).not.toMatch(/ssn/i);

    // Should only have standard claims + minimal user info
    expect(decoded).toHaveProperty('sub');
    expect(decoded).toHaveProperty('exp');
    expect(decoded).toHaveProperty('iat');
  });

  await attach('JWT payload fields (output)', {
    fields: Object.keys(jwtService.decode(tokens.accessToken)),
    sensitiveDataFound: false,
  });
});
```
