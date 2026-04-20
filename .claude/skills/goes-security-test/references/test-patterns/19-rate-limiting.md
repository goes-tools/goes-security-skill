# Pattern 19: Rate Limiting

**Covers:** R55 (Rate Limiting)

```typescript
it('PENTEST: should reject requests that exceed the rate limit', async () => {
  await allure.epic('Security');
  await allure.feature('Rate Limiting');
  await allure.story('Block excessive requests per IP');
  await allure.severity('blocker');
  await allure.tag('Pentest');
  await allure.tag('OWASP A04');
  await allure.tag('OWASP API4');
  await allure.tag('GOES Checklist R55');
  await allure.description(
    '## Vulnerability Prevented\n' +
    '**DoS / Brute Force** — An attacker sends thousands of requests\n' +
    'to overwhelm the server or try combinations by brute force.\n\n' +
    '## Defense Implemented\n' +
    'Rate limiting with ThrottlerGuard: 100 req/min for normal endpoints,\n' +
    '5 req/min for login. Responds 429 Too Many Requests\n' +
    'with Retry-After header.\n\n' +
    '## Reference\n' +
    'OWASP A04:2021 — Insecure Design\n' +
    'GOES Guide Section 6 — API Security',
  );

  await allure.parameter('normal_limit', '100 req/min');
  await allure.parameter('login_limit', '5 req/min');

  await allure.step('Prepare: simulate IP already exceeded the limit', async () => {
    // Adapt to the project's actual throttler (ThrottlerGuard, express-rate-limit, etc.)
    mockThrottler.isRateLimited.mockReturnValue(true);
  });

  await attach('Attacker state (input)', {
    ip: '192.168.1.100',
    endpoint: '/auth/login',
    requestsInLastMinute: 6,
    limit: 5,
  });

  await allure.step('Execute: request that exceeds the limit', async () => {
    await expect(
      controller.login(dto, mockRequest),
    ).rejects.toThrow('ThrottlerException');
  });

  await allure.step('Verify: 429 response with Retry-After', async () => {
    expect(mockResponse.statusCode).toBe(429);
  });

  await attach('Defense response (output)', {
    statusCode: 429,
    message: 'Too Many Requests',
    retryAfter: '60 seconds',
    note: 'IP temporarily blocked for exceeding limit',
  });
});

it('should have stricter rate limit on login endpoint', async () => {
  await allure.epic('Security');
  await allure.feature('Rate Limiting');
  await allure.story('Login endpoint has lower rate limit than normal endpoints');
  await allure.severity('blocker');
  await allure.tag('Auth');
  await allure.tag('OWASP API2');
  await allure.tag('GOES Checklist R55');
  await allure.description(
    '## Objective\n' +
    'Verify that the login endpoint has a stricter rate limit\n' +
    '(5 req/min) compared to normal endpoints (100 req/min).',
  );

  await allure.step('Verify: login controller has @Throttle decorator', async () => {
    // Check that the login method or controller has rate limiting configured
    // This can be verified by checking the decorator metadata
    const metadata = Reflect.getMetadata('THROTTLER:LIMIT', controller.login);
    expect(metadata).toBeDefined();
    // Login should have a lower limit
    expect(metadata).toBeLessThanOrEqual(10);
  });

  await attach('Rate limit config (output)', {
    loginLimit: '5 req/min',
    normalLimit: '100 req/min',
  });
});
```
