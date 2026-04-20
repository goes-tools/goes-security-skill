# Basic Setup — Use in Every Spec File

```typescript
import { Test, TestingModule } from '@nestjs/testing';
import * as allure from 'allure-js-commons';
import { MyService } from '../../src/modules/my-module/my-service.service';
import { PrismaService } from '../../src/prisma/prisma.service';

// Evidence helper — define ONCE per file
async function attach(name: string, data: unknown) {
  await allure.attachment(name, JSON.stringify(data, null, 2), {
    contentType: 'application/json',
  });
}

// Mock factory — adapt to the project's ORM (Prisma, TypeORM, etc.)
function createMockPrisma() {
  return {
    user: {
      findUnique: jest.fn(),
      findMany: jest.fn(),
      create: jest.fn(),
      update: jest.fn(),
      delete: jest.fn(),
      count: jest.fn(),
    },
    // Add more models as needed
  };
}

describe('MyService — Security Tests', () => {
  let service: MyService;
  let prisma: any;

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        MyService,
        { provide: PrismaService, useValue: createMockPrisma() },
      ],
    }).compile();

    service = module.get<MyService>(MyService);
    prisma = module.get(PrismaService);
  });

  // ... tests from patterns go here ...
});
```

## Import paths

All test files live in `test/security/`, so imports to `src/` use `../../src/...`:

```typescript
// From test/security/auth-service.security.spec.ts
import { AuthService } from '../../src/modules/auth/auth.service';
import { PrismaService } from '../../src/prisma/prisma.service';
import { JwtService } from '@nestjs/jwt';
```

## IMPORTANT: Adapt everything to the real project

All patterns in this folder are **reference examples** — they use placeholder values like `/api/users/1`, `service.findOne()`, `prisma.user`, etc. You MUST adapt every pattern to the actual project:

- **Endpoints**: Read the real controllers (`@Get`, `@Post`, `@Patch`, `@Delete` decorators) and use the actual route paths.
- **Service methods**: Read the real `.service.ts` files and use the actual method names and signatures.
- **ORM models**: Read the real Prisma schema or TypeORM entities and use the actual model names.
- **DTOs**: Read the real `.dto.ts` files and use the actual field names and validation rules.
- **Guards/Middleware**: Read the real guard and middleware files and use the actual class names.
- **Error messages**: Read the real error messages thrown by the service and match against those.

Never copy a pattern verbatim — always read the source code first and adapt.
