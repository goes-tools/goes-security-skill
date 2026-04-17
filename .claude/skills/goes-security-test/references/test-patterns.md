# Test Patterns — Codigo de referencia

Patrones exactos que se deben usar al generar tests. Copiar y adaptar al servicio real.

## Setup basico de cada archivo spec

```typescript
import { Test, TestingModule } from '@nestjs/testing';
import * as allure from 'allure-js-commons';
import { MiServicio } from './mi-servicio.service';

// Helper de evidencia — definir UNA VEZ por archivo
async function attach(name: string, data: unknown) {
  await allure.attachment(name, JSON.stringify(data, null, 2), {
    contentType: 'application/json',
  });
}

describe('MiServicio', () => {
  let service: MiServicio;
  let prisma: any; // mock del ORM

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        MiServicio,
        { provide: PrismaService, useValue: createMockPrisma() },
      ],
    }).compile();

    service = module.get<MiServicio>(MiServicio);
    prisma = module.get(PrismaService);
  });

  // ... tests ...
});
```

## Pattern 1: Test funcional CRUD

```typescript
it('debe crear un registro correctamente con datos validos', async () => {
  await allure.epic('Dominio');
  await allure.feature('Gestion de Registros');
  await allure.story('Crear registro nuevo');
  await allure.severity('blocker');
  await allure.tag('CRUD');
  await allure.tag('Happy Path');
  await allure.description(
    '## Objetivo\n' +
    'Verificar que el servicio crea un registro correctamente cuando recibe datos validos.\n\n' +
    '## Comportamiento esperado\n' +
    '- Retorna el objeto creado con ID generado\n' +
    '- Los campos coinciden con el DTO enviado',
  );

  const dto = { nombre: 'Ejemplo', descripcion: 'Test' };
  await allure.parameter('nombre', dto.nombre);

  await allure.step('Preparar: configurar mock de BD para insert', async () => {
    prisma.registro.create.mockResolvedValue({ id: 1, ...dto });
  });

  await attach('DTO enviado (input)', dto);

  const result = await allure.step('Ejecutar: service.create(dto)', async () => {
    return service.create(dto);
  });

  await allure.step('Verificar: retorna objeto con ID', async () => {
    expect(result).toHaveProperty('id');
    expect(result.nombre).toBe(dto.nombre);
  });

  await attach('Registro creado (output)', result);
});
```

## Pattern 2: PENTEST — Timing Attack

```typescript
it('PENTEST: debe responder en tiempo constante si el email no existe', async () => {
  await allure.epic('Seguridad');
  await allure.feature('Timing Attack Prevention');
  await allure.story('Tiempo constante independiente de existencia del usuario');
  await allure.severity('critical');
  await allure.tag('Pentest');
  await allure.tag('OWASP A07');
  await allure.tag('GOES Checklist R14');
  await allure.description(
    '## Vulnerabilidad que previene\n' +
    '**Timing Attack** — Si el servidor responde mas rapido cuando\n' +
    'el email no existe, un atacante puede medir el tiempo de\n' +
    'respuesta para descubrir emails validos.\n\n' +
    '## Defensa implementada\n' +
    'Se ejecuta `bcrypt.compare()` contra un dummy hash incluso\n' +
    'cuando el usuario no existe, para igualar el tiempo de respuesta.\n\n' +
    '## Referencia\n' +
    'OWASP A07:2021 — Identification and Authentication Failures',
  );

  await allure.parameter('email', 'no-existe@test.com');

  await allure.step('Preparar: simular que el email NO existe en BD', async () => {
    prisma.user.findUnique.mockResolvedValue(null);
  });

  await attach('Credenciales del atacante (input)', {
    email: 'no-existe@test.com',
    password: 'cualquier-password',
  });

  await allure.step('Ejecutar: intentar login con email inexistente', async () => {
    await expect(
      service.login({ email: 'no-existe@test.com', password: 'x' }, '1.2.3.4', 'ua'),
    ).rejects.toThrow('Credenciales invalidas');
  });

  await allure.step('Verificar: bcrypt.compare fue llamado (dummy hash)', async () => {
    expect(bcrypt.compare).toHaveBeenCalled();
  });

  await attach('Respuesta al atacante (output)', {
    statusCode: 401,
    message: 'Credenciales invalidas',
    nota: 'Mismo mensaje y mismo tiempo que si el email existiera',
  });
});
```

## Pattern 3: PENTEST — Replay Attack

```typescript
it('PENTEST: debe detectar REPLAY ATTACK y revocar toda la familia', async () => {
  await allure.epic('Seguridad');
  await allure.feature('Replay Attack Detection');
  await allure.story('Detectar reuso de token y revocar familia completa');
  await allure.severity('blocker');
  await allure.tag('Pentest');
  await allure.tag('OWASP A07');
  await allure.tag('GOES Checklist R32');
  await allure.description(
    '## Vulnerabilidad que previene\n' +
    '**Replay Attack** — Un atacante intercepta un refresh token y\n' +
    'lo reusa despues de que el usuario legitimo ya lo consumio.\n\n' +
    '## Defensa implementada\n' +
    'Si un token revocado se presenta de nuevo:\n' +
    '1. Se detecta como REPLAY ATTACK\n' +
    '2. Se revocan TODOS los tokens de esa familia\n' +
    '3. Se registra evento en auditoria',
  );

  await allure.parameter('token_reusado', 'reused-token');
  await allure.parameter('familyId', 'family-1');

  await allure.step('Preparar: token ya revocado en BD', async () => {
    prisma.refreshToken.findUnique.mockResolvedValue({
      id: 'token-1', token: 'reused-token', isRevoked: true, familyId: 'family-1',
    });
    prisma.refreshToken.updateMany.mockResolvedValue({ count: 3 });
  });

  await attach('Token reusado por el atacante (input)', {
    token: 'reused-token', isRevoked: true, familyId: 'family-1',
  });

  await allure.step('Ejecutar: atacante reusa token revocado', async () => {
    await expect(
      service.refreshTokens('reused-token', '1.2.3.4', 'ua'),
    ).rejects.toThrow('Sesion expirada');
  });

  await allure.step('Verificar: TODA la familia revocada', async () => {
    expect(prisma.refreshToken.updateMany).toHaveBeenCalledWith({
      where: { familyId: 'family-1' },
      data: { isRevoked: true },
    });
  });

  await attach('Acciones de defensa ejecutadas (output)', {
    replayDetectado: true, familiaRevocada: 'family-1', tokensAfectados: 3,
  });
});
```

## Pattern 4: PENTEST — Brute Force

```typescript
it('PENTEST: debe bloquear cuenta tras 5 intentos fallidos', async () => {
  await allure.epic('Seguridad');
  await allure.feature('Brute Force Protection');
  await allure.story('Bloqueo temporal tras intentos fallidos consecutivos');
  await allure.severity('blocker');
  await allure.tag('Pentest');
  await allure.tag('OWASP A07');
  await allure.tag('GOES Checklist R27');
  await allure.description(
    '## Vulnerabilidad que previene\n' +
    '**Brute Force** — Un atacante intenta miles de contrasenas\n' +
    'hasta encontrar la correcta.\n\n' +
    '## Defensa implementada\n' +
    'Tras 5 intentos fallidos consecutivos desde la misma IP,\n' +
    'la cuenta se bloquea temporalmente (15 minutos).',
  );

  await allure.parameter('max_intentos', '5');
  await allure.parameter('bloqueo_minutos', '15');

  await allure.step('Preparar: simular 5 intentos fallidos previos', async () => {
    prisma.loginAttempt.count.mockResolvedValue(5);
  });

  await attach('Estado del atacante (input)', {
    email: 'victima@test.com', intentosFallidos: 5, ip: '192.168.1.100',
  });

  await allure.step('Ejecutar: intento #6 de login', async () => {
    await expect(
      service.login(dto, '192.168.1.100', 'ua'),
    ).rejects.toThrow('Cuenta bloqueada temporalmente');
  });

  await allure.step('Verificar: se registro el bloqueo', async () => {
    expect(prisma.accountLock.create).toHaveBeenCalled();
  });

  await attach('Respuesta de defensa (output)', {
    bloqueado: true, razon: 'Demasiados intentos fallidos', desbloqueoEn: '15 minutos',
  });
});
```

## Pattern 5: PENTEST — IDOR

```typescript
it('PENTEST: debe impedir acceso a recurso de otro usuario (IDOR)', async () => {
  await allure.epic('Seguridad');
  await allure.feature('IDOR Prevention');
  await allure.story('Usuario no puede acceder a recursos de otro usuario por ID');
  await allure.severity('blocker');
  await allure.tag('Pentest');
  await allure.tag('OWASP A01');
  await allure.tag('OWASP API1');
  await allure.tag('GOES Checklist R23');
  await allure.description(
    '## Vulnerabilidad que previene\n' +
    '**IDOR (Insecure Direct Object Reference)** — Un atacante modifica\n' +
    'el ID en la URL para acceder a recursos de otro usuario.\n\n' +
    '## Defensa implementada\n' +
    'Verificar que el recurso solicitado pertenece al usuario autenticado\n' +
    'antes de retornarlo. No confiar en IDs de URL.',
  );

  await allure.parameter('resourceId', 'resource-de-otro-usuario');
  await allure.parameter('userId_atacante', 'user-1');
  await allure.parameter('userId_victima', 'user-2');

  await allure.step('Preparar: recurso pertenece a user-2, atacante es user-1', async () => {
    prisma.resource.findUnique.mockResolvedValue({
      id: 'resource-1', userId: 'user-2', // pertenece a otro
    });
  });

  await attach('Intento del atacante (input)', {
    resourceId: 'resource-1', attackerUserId: 'user-1', ownerUserId: 'user-2',
  });

  await allure.step('Ejecutar: atacante solicita recurso de otro usuario', async () => {
    await expect(
      service.findOne('resource-1', 'user-1'),
    ).rejects.toThrow('No autorizado');
  });

  await attach('Defensa ejecutada (output)', {
    accesoBloqueado: true, razon: 'Recurso no pertenece al usuario autenticado',
  });
});
```

## Pattern 6: Configuracion — Security Headers

```typescript
it('debe configurar todos los headers de seguridad HTTP requeridos', async () => {
  await allure.epic('Configuracion');
  await allure.feature('Security Headers');
  await allure.story('Headers HTTP de seguridad configurados correctamente');
  await allure.severity('critical');
  await allure.tag('Config');
  await allure.tag('OWASP A05');
  await allure.tag('GOES Checklist R44');
  await allure.tag('GOES Checklist R45');
  await allure.tag('GOES Checklist R46');
  await allure.description(
    '## Objetivo\n' +
    'Verificar que los headers de seguridad HTTP estan configurados\n' +
    'segun la Guia GOES seccion 8 y OWASP Security Headers.\n\n' +
    '## Headers requeridos\n' +
    '- Content-Security-Policy\n' +
    '- X-Content-Type-Options: nosniff\n' +
    '- X-Frame-Options: DENY\n' +
    '- Strict-Transport-Security\n' +
    '- Referrer-Policy\n' +
    '- Permissions-Policy',
  );

  // Verificar configuracion de helmet o headers manuales
  await allure.step('Verificar: CSP configurado', async () => {
    // Adaptar al middleware real del proyecto (helmet, express, etc.)
    expect(helmetConfig).toHaveProperty('contentSecurityPolicy');
  });

  await allure.step('Verificar: X-Content-Type-Options nosniff', async () => {
    expect(headers['x-content-type-options']).toBe('nosniff');
  });

  // ... mas verificaciones por header ...
});
```

## Pattern 7: Auditoria — Audit Log

```typescript
it('debe registrar evento de auditoria al crear un recurso', async () => {
  await allure.epic('Auditoria');
  await allure.feature('Audit Log');
  await allure.story('Registrar evento CREATE en audit log');
  await allure.severity('normal');
  await allure.tag('Auditoria');
  await allure.tag('OWASP A09');
  await allure.tag('GOES Checklist R10');
  await allure.description(
    '## Objetivo\n' +
    'Verificar que cada operacion de escritura genera un registro\n' +
    'de auditoria con usuario, accion, recurso y timestamp.\n\n' +
    '## Referencia\n' +
    'Guia GOES Seccion 9.2 — Que Loguear',
  );

  await allure.parameter('accion', 'CREATE');
  await allure.parameter('recurso', 'Sistema');

  await allure.step('Ejecutar: service.create(dto)', async () => {
    await service.create(dto);
  });

  await allure.step('Verificar: audit log fue creado', async () => {
    expect(prisma.auditLog.create).toHaveBeenCalledWith({
      data: expect.objectContaining({
        action: 'CREATE',
        resource: 'Sistema',
        userId: expect.any(String),
      }),
    });
  });

  await attach('Evento de auditoria (output)', {
    action: 'CREATE', resource: 'Sistema', userId: 'user-123',
  });
});
```

## Severidades — Guia de asignacion

| Severity | Cuando usar | Ejemplo |
|----------|-------------|---------|
| blocker | Si falla, el sistema no puede operar de forma segura | Login roto, RBAC bypasseado, SQL injection posible |
| critical | Funcionalidad critica comprometida | Token sin validar, password sin hashear, CORS abierto |
| normal | Funcionalidad estandar | Filtros, paginacion, audit log |
| minor | Edge cases no criticos | Logout sin token, ruta inexistente |
| trivial | Cosmeticos | Formato de respuesta |
