# Pattern 10: IDOR Prevention

> **Migration note (skill v1.1):** these patterns were originally written for
> `allure-js-commons`. They now run against the bundled custom HTML reporter
> via `AllureCompat`, which mirrors the same API. The `_setup.md` snippet
> shows the new top-level imports. Existing `await allure.epic(...)`,
> `await allure.step(...)` etc. work identically. Two extras:
>
> - Each `it(...)` block must end with `await allure.flush();` so the metadata
>   reaches the reporter.
> - `attach(name, data)` is now async — call it as `await attach(...)`.

**Covers:** R23 (IDOR Prevention)

```typescript
it('PENTEST: should prevent access to another user resource (IDOR)', async () => {
  const allure = new AllureCompat();
  const attach = attachFor(allure);
  await allure.epic('Security');
  await allure.feature('IDOR Prevention');
  await allure.story('User cannot access resources belonging to another user by ID');
  await allure.severity('blocker');
  await allure.tag('Pentest');
  await allure.tag('OWASP A01');
  await allure.tag('OWASP API1');
  await allure.tag('GOES Checklist R23');
  await allure.description(
    '## Vulnerability Prevented\n' +
    '**IDOR (Insecure Direct Object Reference)** — An attacker modifies\n' +
    'the ID in the URL to access another user\'s resources.\n\n' +
    '## Defense Implemented\n' +
    'Verify that the requested resource belongs to the authenticated user\n' +
    'before returning it. Never trust URL IDs.',
  );

  await allure.parameter('resourceId', 'other-users-resource');
  await allure.parameter('attacker_userId', 'user-1');
  await allure.parameter('victim_userId', 'user-2');

  await allure.step('Prepare: resource belongs to user-2, attacker is user-1', async () => {
    prisma.resource.findUnique.mockResolvedValue({
      id: 'resource-1', userId: 'user-2',
    });
  });

  await attach('Attacker attempt (input)', {
    resourceId: 'resource-1', attackerUserId: 'user-1', ownerUserId: 'user-2',
  });

  await allure.step('Execute: attacker requests another user\'s resource', async () => {
    await expect(
      service.findOne('resource-1', 'user-1'),
    ).rejects.toThrow('Unauthorized');
  });

  await allure.step('Execute: attacker tries to update another user\'s resource', async () => {
    await expect(
      service.update('resource-1', { name: 'hacked' }, 'user-1'),
    ).rejects.toThrow('Unauthorized');
  });

  await allure.step('Execute: attacker tries to delete another user\'s resource', async () => {
    await expect(
      service.remove('resource-1', 'user-1'),
    ).rejects.toThrow('Unauthorized');
  });

  await attach('Defense executed (output)', {
    readBlocked: true,
    updateBlocked: true,
    deleteBlocked: true,
    reason: 'Resource does not belong to the authenticated user',
  });
  await allure.flush();
});
```
