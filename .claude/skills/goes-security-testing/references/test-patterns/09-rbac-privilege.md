# Pattern 09: RBAC / Privilege Escalation Prevention

> **Migration note (skill v1.1):** these patterns were originally written for
> `allure-js-commons`. They now run against the bundled custom HTML reporter
> via `AllureCompat`, which mirrors the same API. The `_setup.md` snippet
> shows the new top-level imports. Existing `await allure.epic(...)`,
> `await allure.step(...)` etc. work identically. Two extras:
>
> - Each `it(...)` block must end with `await allure.flush();` so the metadata
>   reaches the reporter.
> - `attach(name, data)` is now async — call it as `await attach(...)`.

**Covers:** R9 (RBAC Enforcement), R24 (Privilege Escalation Prevention), R34 (Role-Based Access Control)

```typescript
it('PENTEST: should enforce role-based access on every endpoint', async () => {
  const allure = new AllureCompat();
  const attach = attachFor(allure);
  await allure.epic('Authentication');
  await allure.feature('RBAC Enforcement');
  await allure.story('Server validates role/permission on every request');
  await allure.severity('blocker');
  await allure.tag('Pentest');
  await allure.tag('OWASP A01');
  await allure.tag('OWASP API5');
  await allure.tag('GOES Checklist R9');
  await allure.tag('GOES Checklist R34');
  await allure.description(
    '## Vulnerability Prevented\n' +
    '**Missing RBAC** — Endpoints that do not check the user\'s role\n' +
    'allow any authenticated user to perform any action.\n\n' +
    '## Defense Implemented\n' +
    'Every endpoint is protected by a RolesGuard that checks the user\'s\n' +
    'role against the required role defined in route metadata.',
  );

  const testCases = [
    { role: 'USER', endpoint: 'admin/users', action: 'listAllUsers', shouldAllow: false },
    { role: 'USER', endpoint: 'admin/roles', action: 'manageRoles', shouldAllow: false },
    { role: 'ADMIN', endpoint: 'admin/users', action: 'listAllUsers', shouldAllow: true },
    { role: 'MODERATOR', endpoint: 'admin/settings', action: 'changeSettings', shouldAllow: false },
  ];

  for (const { role, endpoint, action, shouldAllow } of testCases) {
    await allure.step(`Test: ${role} accessing ${endpoint}`, async () => {
      const mockUser = { id: '1', role };

      if (shouldAllow) {
        await expect(guard.canActivate(createMockContext(mockUser, endpoint))).resolves.toBe(true);
      } else {
        await expect(guard.canActivate(createMockContext(mockUser, endpoint))).rejects.toThrow();
      }
    });
  }

  await attach('RBAC test results (output)', {
    scenarios: testCases.length,
    allCorrect: true,
  });
  await allure.flush();
});

it('PENTEST: should prevent privilege escalation via role manipulation', async () => {
  const allure = new AllureCompat();
  const attach = attachFor(allure);
  await allure.epic('Authentication');
  await allure.feature('Privilege Escalation Prevention');
  await allure.story('Low-privilege user cannot elevate their own role');
  await allure.severity('blocker');
  await allure.tag('Pentest');
  await allure.tag('OWASP A01');
  await allure.tag('GOES Checklist R24');
  await allure.description(
    '## Vulnerability Prevented\n' +
    '**Privilege Escalation** — A regular user sends a request to\n' +
    'change their own role to ADMIN, or accesses admin-only actions.\n\n' +
    '## Defense Implemented\n' +
    'Role changes are restricted to ADMIN users only.\n' +
    'The role field is ignored in self-update requests.',
  );

  await allure.parameter('attacker_role', 'USER');
  await allure.parameter('attempted_role', 'ADMIN');

  await allure.step('Prepare: regular user tries to update their own role', async () => {
    prisma.user.findUnique.mockResolvedValue({ id: '1', role: 'USER' });
  });

  await allure.step('Execute: user sends update with role=ADMIN', async () => {
    await expect(
      service.update('1', { role: 'ADMIN' }, { userId: '1', role: 'USER' }),
    ).rejects.toThrow();
  });

  await allure.step('Verify: role was NOT changed', async () => {
    // If the service allows partial updates, verify role field was stripped
    if (prisma.user.update.mock.calls.length > 0) {
      const updateData = prisma.user.update.mock.calls[0][0].data;
      expect(updateData).not.toHaveProperty('role');
    }
  });

  await attach('Privilege escalation attempt (output)', {
    escalationBlocked: true,
    reason: 'Only ADMIN can change roles',
  });
  await allure.flush();
});
```
