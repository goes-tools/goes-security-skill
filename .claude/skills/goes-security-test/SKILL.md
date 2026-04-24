---
name: goes-security-testing
description: "Genera tests de seguridad con un Custom HTML + Excel Reporter (sin Java ni Allure) cubriendo el Checklist de Ciberseguridad GOES, OWASP Top 10 y OWASP API Security Top 10. Configura Jest + custom reporter y crea specs completas con evidencia JSON, trazabilidad regulatoria y clasificacion por epicas. El reporte incluye sidebar Epic-Feature-Story, graficos SVG, tema oscuro, busqueda y buckets (Pasados / Desactivados / Migrados / No aplicables). Activar cuando el usuario pida: tests de seguridad, checklist GOES, pentest tests, security specs, reporte de seguridad HTML o Excel, security report, o cualquier variante."
---

# GOES Security Testing — Skill para NestJS + Jest + Custom HTML Reporter

## Resumen

Este skill genera tests de seguridad profesionales para proyectos **NestJS + Jest** con un **Custom HTML Reporter** (sin Java, sin Allure).
Cubre **60 items** del Checklist de Ciberseguridad GOES, **OWASP Top 10** y **OWASP API Security Top 10**.

El reporter produce DOS artefactos autocontenidos:

- **HTML** (`reports/security/security-report.html`)
  - Sidebar con navegacion por Epic > Feature > Story
  - Modal de detalle con severity badges, tags, steps y evidencia JSON con syntax highlighting
  - Graficos SVG (pie de pass/fail, barras por severity, donut OWASP)
  - Buckets: Pasados / Desactivados / Migrados / No aplicables
  - Tema oscuro, busqueda en tiempo real
- **Excel** (`reports/security/security-report.xlsx`)
  - Una hoja por bucket, una fila por test con todos los campos de metadata y tags
  - Ideal como anexo de auditoria regulatoria

---

## PASO 1: Analizar el proyecto

Antes de generar cualquier codigo, analizar la estructura del proyecto:

```
1. Leer package.json para identificar:
   - Framework (NestJS, Express, etc.)
   - ORM (Prisma, TypeORM, Sequelize, etc.)
   - Version de Jest
   - Dependencias de auth (passport, @nestjs/jwt, bcrypt, etc.)

2. Listar los modulos/servicios existentes:
   - Glob: src/modules/**/**.service.ts
   - Glob: src/modules/**/**.controller.ts
   - Glob: src/**/*.module.ts

3. Identificar que ya existe:
   - Archivos .spec.ts existentes
   - Configuracion de Jest actual
   - Guards, interceptors, middleware de seguridad
```

---

## PASO 2: Instalar dependencias (si no existen)

Verificar en package.json y solo instalar lo que falte:

```bash
npm install --save-dev ts-jest @types/jest xlsx
```

`xlsx` (SheetJS) es usado por el reporter para generar el Excel (`security-report.xlsx`) en paralelo al HTML.

**NO instalar Allure, allure-commandline, allure-jest ni jest-html-reporters.** Este sistema usa un reporter custom puro Node.js que no necesita Java ni dependencias externas de reporte.

---

## PASO 3: Configurar el custom reporter

Este skill incluye el reporter bundled en `.claude/skills/goes-security-test/reporter/`. **NO copiar los archivos** — referenciarlos directamente para evitar duplicados.

El reporter consiste en dos archivos:

- **`reporter/html-reporter.js`** — Jest custom reporter (JavaScript puro, ~1780 lineas). Implementa `onRunComplete(testContexts, results)`: lee metadata JSON temporales, cruza con resultados de Jest y genera DOS artefactos autocontenidos:
  - `reports/security/security-report.html` — sidebar Epic > Feature > Story, modal de detalle, charts SVG, dark theme, busqueda en tiempo real.
  - `reports/security/security-report.xlsx` — workbook con hojas por bucket (Pasados / Desactivados / Migrados / No aplicables), una fila por test con todos los campos de metadata, severity y tags — listo para auditoria y evidencia regulatoria.
- **`reporter/metadata.ts`** — Collector de metadata. Exporta `report()` y `AllureCompat`. Cada test registra epic, feature, story, severity, tags, steps, evidencia y se escribe a archivos JSON temporales via `flush()`.

**NO modificar los archivos del reporter.** Estan listos para usar.

### Buckets de clasificacion

El reporter clasifica automaticamente cada test en uno de cuatro buckets segun sus tags:

| Bucket | Criterio | Cuando usarlo |
|--------|----------|---------------|
| **Pasados** | Tests que corrieron y pasaron | Control implementado y verificado |
| **Desactivados** | Tests con `.skip()` que tienen metadata | Control temporalmente deshabilitado (ej: BE caido, feature flag apagado) |
| **Migrados** | Tests con tag `Migrado` | Control que antes aplicaba y ahora vive en otra capa |
| **No aplicables** | Tests con tag `N/A` | Controles fuera del scope del servicio (ej: UI-only en un backend) |

Cada bucket aparece como una hoja separada en el Excel y como una seccion filtrable en el HTML.

La estructura del proyecto sera:

```
test/security/
├── *.security-html.spec.ts         ← Archivos de specs de seguridad
└── jest-security-html.config.ts    ← Config de Jest (apunta a .claude/ reporter)
```

---

## PASO 4: Configurar Jest

### jest-security-html.config.ts

```typescript
import type { Config } from 'jest';
import * as path from 'path';

const reporterPath = path.resolve(__dirname, '../../.claude/skills/goes-security-test/reporter');

const config: Config = {
  moduleFileExtensions: ['js', 'json', 'ts'],
  rootDir: '../..',
  testMatch: ['<rootDir>/test/security/**/*.security-html.spec.ts'],
  transform: {
    '^.+\\.(t|j)s$': 'ts-jest',
  },
  testEnvironment: 'node',
  moduleNameMapper: {
    '^src/(.*)$': '<rootDir>/src/$1',
    '^@security-reporter/(.*)$': path.join(reporterPath, '$1'),
  },
  reporters: [
    'default',
    [
      path.join(reporterPath, 'html-reporter.js'),
      {
        outputPath: './reports/security/security-report.html',
      },
    ],
  ],
};

export default config;
```

El alias `@security-reporter` permite que los specs importen metadata limpiamente:

```typescript
import { report } from '@security-reporter/metadata';
```

### Scripts npm (agregar a package.json)

```json
{
  "test:security": "jest --config test/security/jest-security-html.config.ts --verbose"
}
```

Para generar el reporte:
```bash
npm run test:security
# HTML:  reports/security/security-report.html
# Excel: reports/security/security-report.xlsx
```

Ambos archivos se regeneran en cada corrida. El HTML es autocontenido (se puede abrir sin servidor). El Excel incluye una hoja por bucket y es ideal para anexos de auditoria.

---

## PASO 5: Crear archivos de soporte

### .gitignore (agregar si no existen)
```
/coverage
/reports
```

### eslint.config.mjs (agregar override para archivos de test)
```javascript
// Dentro del array de configuracion, agregar:
{
  files: ['**/*.spec.ts', '**/*.e2e-spec.ts', 'test/**/*.ts'],
  rules: {
    '@typescript-eslint/no-unsafe-assignment': 'off',
    '@typescript-eslint/no-unsafe-member-access': 'off',
    '@typescript-eslint/no-unsafe-call': 'off',
    '@typescript-eslint/no-unsafe-return': 'off',
    '@typescript-eslint/no-unsafe-argument': 'off',
    '@typescript-eslint/no-unused-vars': 'warn',
    '@typescript-eslint/require-await': 'off',
  },
}
```

---

## PASO 6: Generar tests

Para CADA servicio/controller del proyecto, crear un archivo `.security-html.spec.ts` siguiendo las reglas de este skill.

### Estructura obligatoria de cada test

Hay dos patrones equivalentes. Usar el que resulte mas conveniente:

#### Patron A: report() directo (recomendado para tests nuevos)

```typescript
import { Test, TestingModule } from '@nestjs/testing';
import { report } from '@security-reporter/metadata';

describe('NombreDelServicio', () => {
  // setup...

  it('nombre descriptivo del test', async () => {
    const t = report();

    // 1. METADATA (obligatorio en cada test)
    t.epic('Seguridad');
    t.feature('Input Validation');
    t.story('Rechazar payload con SQL injection');
    t.severity('blocker');
    t.tag('Pentest', 'OWASP A03', 'GOES Checklist R5');

    // 2. PARAMETERS (inputs visibles en el reporte)
    t.parameter('email', 'test@goes.gob.sv');
    t.parameter('payload', "' OR 1=1 --");

    // 3. STEPS (patron Preparar/Ejecutar/Verificar)
    t.step('Preparar: crear payload malicioso');
    const payload = { email: "' OR 1=1 --" };
    t.evidence('Input (payload)', payload);

    t.step('Ejecutar: enviar al servicio');
    const result = await service.validate(payload);

    t.step('Verificar: debe rechazar la inyeccion');
    expect(result.valid).toBe(false);
    t.evidence('Resultado (output)', result);

    await t.flush();
  });
});
```

#### Patron B: AllureCompat (migracion desde Allure existente)

Para migrar tests que ya usaban `allure-js-commons`, se puede usar `AllureCompat` que espeja la misma API:

```typescript
import { AllureCompat } from '@security-reporter/metadata';

it('nombre del test', async () => {
  const allure = new AllureCompat();

  allure.epic('Seguridad');
  allure.feature('Input Validation');
  allure.severity('blocker');
  allure.tag('Pentest', 'OWASP A03');
  allure.parameter('key', 'value');

  const result = allure.step('Ejecutar accion', () => {
    return someFunction();
  });

  await allure.attachment('Resultado', JSON.stringify(result), {
    contentType: 'application/json',
  });

  expect(result).toBeDefined();
  await allure.flush();
});
```

### Reglas de metadata

| Campo | Regla |
|-------|-------|
| `epic` | Area del sistema: Seguridad, Autenticacion, Dominio, Configuracion, Auditoria, Infraestructura, Archivos |
| `feature` | Funcionalidad especifica: Timing Attack Prevention, RBAC, Input Validation, etc. |
| `story` | Escenario concreto que se prueba |
| `severity` | blocker = sistema no opera / critical = funcionalidad comprometida / normal = estandar / minor = edge case |
| `tag` | SIEMPRE incluir: tag de categoria (Pentest, CRUD, Auth, Config) + tag de normativa (OWASP Axx, GOES Checklist Rxx) |
| `parameter` | Inputs clave del test: payloads, emails, tokens, configuraciones |
| `step` | Pasos descriptivos del test en formato Preparar/Ejecutar/Verificar |
| `evidence` | Objetos JSON con datos de entrada/salida que se muestran con syntax highlighting en el reporte |

### Regla para tests de PENTEST

- Prefijo "PENTEST:" en el nombre del it()
- Epic: 'Seguridad'
- Tag: 'Pentest' + referencia OWASP
- Steps con ## Vulnerabilidad que previene y ## Defensa implementada
- Evidence con payload del atacante (input) y respuesta de defensa (output)

### Regla critica: flush()

Cada test DEBE llamar `await t.flush()` (o `await allure.flush()`) al final. Sin flush, la metadata no se escribe y el reporte no tendra los detalles del test.

---

## CHECKLIST COMPLETO DE CIBERSEGURIDAD GOES

Este es el checklist oficial. Cada item DEBE tener al menos 1 test que lo cubra.
El tag de trazabilidad es `GOES Checklist Rxx` donde xx es el numero de fila.

### CATEGORIA 1: Contenido Web

| ID | Tarea | Epic | Feature sugerida | Severity |
|----|-------|------|------------------|----------|
| R3 | No dejar contenido sensible (llaves, IDs, info personal) en archivos del proyecto | Seguridad | Sensitive Data Exposure | critical |
| R4 | No exponer JS sensible o logica de negocio | Seguridad | Business Logic Exposure | critical |
| R5 | Sanitizar entrada de datos (probar inyeccion de scripts) | Seguridad | XSS Prevention / Input Sanitization | blocker |
| R6 | Si el sitio es publico, configurar Robot.txt y sitemap.xml | Configuracion | Public Site Config | minor |

### CATEGORIA 2: Entrada y salida de datos por el servidor

| ID | Tarea | Epic | Feature sugerida | Severity |
|----|-------|------|------------------|----------|
| R8 | Mensajes 2xx, 4xx, 5xx genericos (no exponer detalles internos) | Seguridad | Generic Error Messages | critical |
| R9 | Validacion por rol o permiso dentro del servidor | Autenticacion | RBAC Enforcement | blocker |
| R10 | Eliminar todo log visible por el usuario | Seguridad | Log Exposure Prevention | critical |
| R11 | Validacion de tipo de datos via DTO, longitud de campo, cantidad maxima de registros | Dominio | DTO Validation / Input Constraints | critical |

### CATEGORIA 3: Autenticacion, registro y acciones de usuarios

| ID | Tarea | Epic | Feature sugerida | Severity |
|----|-------|------|------------------|----------|
| R13 | Tiempo de vida bajo para access token (5 min) y refresh token | Seguridad | Token Lifetime Enforcement | critical |
| R14 | Auth failures: MFA, rate limit, session management | Seguridad | Auth Failure Handling | blocker |
| R15 | Encriptacion bcrypt, Argon2 o PBKDF2 en hashing | Seguridad | Password Hashing Strength | blocker |
| R16 | Uso de algoritmo RS256 para firma de tokens | Seguridad | JWT Signing Algorithm | blocker |
| R17 | Session ID minimo 128 bits de entropia | Seguridad | Session ID Entropy | critical |
| R18 | Session ID regenerado post-login | Seguridad | Session Fixation Prevention | critical |
| R19 | Validar signature, exp, iat, iss, aud en cada request | Seguridad | JWT Claims Validation | blocker |
| R20 | NO almacenar datos sensibles en el payload del JWT | Seguridad | JWT Payload Security | critical |
| R21 | Bloquear navegacion forzada a rutas no autorizadas | Autenticacion | Forced Browsing Prevention | critical |
| R22 | 404 para cualquier ruta que no coincida con las definidas | Configuracion | Unknown Route Handling | normal |
| R23 | Proteccion contra IDOR | Seguridad | IDOR Prevention | blocker |
| R24 | Usuarios con bajo privilegio no acceden a acciones de alto privilegio | Autenticacion | Privilege Escalation Prevention | blocker |
| R25 | Mismo usuario no puede registrarse repetidamente | Autenticacion | Duplicate Registration Prevention | critical |
| R26 | No aceptar disposable emails | Autenticacion | Disposable Email Rejection | normal |
| R27 | Bloquear cuenta tras 3-5 intentos fallidos | Seguridad | Brute Force Protection | blocker |
| R28 | Metodo de recuperacion de cuenta | Autenticacion | Account Recovery | critical |
| R29 | Si hay "recuerdame", la contrasena debe estar encriptada | Seguridad | Remember Me Security | critical |
| R30 | Almacenamiento de contrasena del lado del servidor | Seguridad | Server-side Password Storage | blocker |
| R31 | No permitir usar el username como password | Seguridad | Weak Password Prevention | critical |
| R32 | Renovar refresh token despues de cada uso (rotacion) | Seguridad | Token Rotation | blocker |
| R33 | Validar token en cada peticion privada | Autenticacion | Token Validation Per Request | blocker |
| R34 | Implementar RBAC | Autenticacion | Role-Based Access Control | blocker |
| R35 | Sistema de inactividad (timeout de sesion) | Seguridad | Session Inactivity Timeout | critical |

### CATEGORIA 4: Configuracion

| ID | Tarea | Epic | Feature sugerida | Severity |
|----|-------|------|------------------|----------|
| R37 | Implementar ORM (no queries raw) | Seguridad | ORM Usage / SQL Injection Prevention | blocker |
| R38 | CORS: Access-Control-Allow-Origin solo origenes permitidos | Configuracion | CORS Origin Restriction | critical |
| R39 | CORS: Access-Control-Allow-Methods solo metodos permitidos | Configuracion | CORS Methods Restriction | critical |
| R40 | Access-Control-Allow-Credentials: true solo si necesario | Configuracion | CORS Credentials Policy | normal |
| R41 | Access-Control-Max-Age: 3600 | Configuracion | CORS Preflight Cache | normal |
| R42 | Cookies con HttpOnly, Secure, SameSite | Seguridad | Cookie Security Flags | blocker |
| R43 | Debug mode deshabilitado | Configuracion | Debug Mode Disabled | critical |
| R44 | Content-Security-Policy configurado | Configuracion | CSP Header | critical |
| R45 | X-Content-Type-Options nosniff, X-Frame-Options DENY | Configuracion | Security Headers (XCT, XFO) | critical |
| R46 | Strict-Transport-Security (HSTS) | Configuracion | HSTS Header | critical |
| R47 | X-XSS-Protection 0 (deprecado, usar CSP) | Configuracion | XSS Protection Header | normal |
| R48 | Referrer-Policy strict-origin-when-cross-origin | Configuracion | Referrer Policy Header | normal |
| R49 | Permissions-Policy geolocation=(), camera=(), microphone=() | Configuracion | Permissions Policy Header | normal |
| R50 | Cache-Control no-store para respuestas sensibles | Configuracion | Cache Control Header | critical |
| R51 | Cookie max-age = refresh token duration | Configuracion | Cookie Lifetime Alignment | critical |
| R52 | Deshabilitar metodo PUT | Configuracion | HTTP PUT Disabled | normal |
| R53 | Deshabilitar metodo TRACE | Configuracion | HTTP TRACE Disabled | critical |
| R54 | Deshabilitar http override | Configuracion | HTTP Override Disabled | critical |
| R55 | Rate limit: 100 req/min normal, 5 req/min login | Seguridad | Rate Limiting | blocker |

### CATEGORIA 5: Manejo de archivos

| ID | Tarea | Epic | Feature sugerida | Severity |
|----|-------|------|------------------|----------|
| R57 | Validar extension de archivo y magic byte | Archivos | File Extension + Magic Byte Validation | blocker |
| R58 | Whitelist de extensiones permitidas | Archivos | File Extension Whitelist | blocker |
| R59 | Limitar tamano de archivo | Archivos | File Size Limit | critical |
| R60 | Renombrar archivos con UUID | Archivos | File UUID Rename | critical |

---

## OWASP TOP 10 — Tests requeridos

Cada item debe tener al menos 1 test con tag `OWASP Axx`.

| ID | Vulnerabilidad | Tests a generar | Tag |
|----|---------------|-----------------|-----|
| A01 | Broken Access Control | IDOR, Privilege Escalation, Forced Browsing, Missing Auth | OWASP A01 |
| A02 | Cryptographic Failures | Weak Hashing, Sensitive Data in JWT, Missing TLS | OWASP A02 |
| A03 | Injection | SQL Injection (ORM bypass), XSS, Command Injection | OWASP A03 |
| A04 | Insecure Design | Business Logic Flaws, Missing Rate Limit | OWASP A04 |
| A05 | Security Misconfiguration | Debug Mode, Default Creds, Missing Headers, CORS | OWASP A05 |
| A06 | Vulnerable Components | (SCA - fuera del scope de unit tests) | OWASP A06 |
| A07 | Auth Failures | Brute Force, Timing Attack, Weak Password, Token Replay | OWASP A07 |
| A08 | Data Integrity Failures | (CI/CD - fuera del scope de unit tests) | OWASP A08 |
| A09 | Logging Failures | Missing Audit Log, Sensitive Data in Logs | OWASP A09 |
| A10 | SSRF | URL Validation, Whitelist Enforcement | OWASP A10 |

---

## OWASP API SECURITY TOP 10 — Tests requeridos

| ID | Vulnerabilidad | Tests a generar | Tag |
|----|---------------|-----------------|-----|
| API1 | Broken Object Level Auth | Acceder a recurso de otro usuario por ID | OWASP API1 |
| API2 | Broken Authentication | Rate limit en login, MFA bypass | OWASP API2 |
| API3 | Broken Object Property Auth | No exponer propiedades sensibles, DTOs por rol | OWASP API3 |
| API4 | Unrestricted Resource Consumption | Rate limit, pagination limit, payload max | OWASP API4 |
| API5 | Broken Function Level Auth | Admin endpoint accedido por user normal | OWASP API5 |
| API8 | Security Misconfiguration | Headers, CORS, metodos HTTP | OWASP API8 |

---

## GUIA GOES — Secciones de referencia

Estas son las secciones de la Guia de Desarrollo Seguro GOES que deben mapearse a tests:

### Seccion 3: Validacion de Entrada
- Validar en el servidor (nunca confiar en el cliente)
- Whitelist sobre Blacklist
- Sanitizar y Escapar
- Validar tipo, longitud, formato y rango
- Canonicalizar antes de validar (UTF-8)

### Seccion 4: Autenticacion y Autorizacion
- 4.1 Politicas de Contrasenas: minimo 12 chars, verificar contra listas comprometidas, bcrypt/Argon2/PBKDF2
- 4.2 Gestion de Sesiones: 128 bits entropia, regenerar post-login, HttpOnly/Secure/SameSite, timeout inactividad 15-30 min
- 4.3 JWT Best Practices: RS256, validar signature/exp/iat/iss/aud, access token 15 min max, refresh en httpOnly cookie, rotacion por uso, NO datos sensibles en payload
- 4.4 Control de Acceso RBAC/ABAC: denegar por defecto, validar en CADA endpoint, no confiar en IDs de URL, least privilege, log accesos denegados

### Seccion 5: Criptografia
- Hashing passwords: Argon2id, bcrypt, PBKDF2 (NUNCA MD5, SHA1, SHA256 solo)
- TLS 1.2 minimo, ideal 1.3
- NO hardcodear secretos en codigo fuente, usar vault

### Seccion 6: Seguridad en APIs
- OWASP API Security Top 10 (ver tabla arriba)
- Rate Limiting: 5 intentos/min login, 100-1000 req/min APIs, responder 429 + Retry-After

### Seccion 7: Seguridad en Base de Datos
- SIEMPRE queries parametrizadas / ORM
- NUNCA concatenar strings en queries
- Principio de menor privilegio en BD

### Seccion 8: Headers de Seguridad HTTP
- Content-Security-Policy: default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'
- X-Content-Type-Options: nosniff
- X-Frame-Options: DENY (o SAMEORIGIN)
- Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
- X-XSS-Protection: 0 (deprecado, usar CSP)
- Referrer-Policy: strict-origin-when-cross-origin
- Permissions-Policy: geolocation=(), camera=(), microphone=()
- Cache-Control: no-store (para respuestas sensibles)
- CORS: origenes especificos, solo metodos necesarios, credentials solo si necesario, max-age 3600

### Seccion 9: Manejo de Errores y Logging
- 9.1 Errores Seguros: mensajes genericos al usuario, NUNCA exponer stack traces/queries/rutas/versiones
- 9.2 Que Loguear: intentos login con IP/timestamp, cambios password, acceso denegado (403), operaciones admin, errores validacion, creacion/eliminacion recursos sensibles
- 9.3 Que NO Loguear: contrasenas, tokens, API keys, tarjetas, datos personales sensibles, secretos

### Seccion 10: Upload de Archivos
- Validar extension Y magic bytes
- Whitelist de extensiones (.pdf, .jpg, .png)
- Limitar tamano maximo (ej: 10MB)
- Renombrar con UUID
- Almacenar fuera del webroot o en storage externo (S3)
- NO ejecutar archivos subidos

---

## PASO 7: Verificacion

Despues de generar todos los archivos:

1. Ejecutar `npm run test:security` para verificar que todos los tests pasan y los reportes se generan
2. Si hay errores de ESLint, corregirlos (especialmente require-await en archivos de test)
3. Verificar que se generaron AMBOS artefactos:
   - `reports/security/security-report.html`
   - `reports/security/security-report.xlsx`
4. Mostrar resumen: cuantos tests generados, cuantos items del checklist cubiertos, cuantos OWASP cubiertos, distribucion por bucket (pasados / desactivados / migrados / no aplicables)

---

## NOTAS IMPORTANTES PARA LA IA

1. **NUNCA generar tests vacios o placeholder** — cada test debe tener assertions reales contra el codigo del proyecto.
2. **Analizar el codigo real del servicio** antes de escribir tests. Leer el archivo .service.ts o .controller.ts y entender los metodos, validaciones y logica.
3. **Si un item del checklist no aplica** al servicio actual (ej: el servicio no maneja archivos, no generar tests de R57-R60), documentar en un comentario que items se omitieron y por que.
4. **Priorizar tests de seguridad** sobre tests funcionales. Cada servicio debe tener al minimo:
   - Tests funcionales (CRUD/logica) con epic 'Dominio'
   - Tests de validacion de entrada con epic 'Seguridad'
   - Tests de autorizacion si aplica con epic 'Autenticacion'
5. **Los comentarios en el codigo van SIN tildes** (ASCII puro). Los strings de metadata (descriptions, steps, stories) SI llevan tildes.
6. **Respetar el tsconfig.json del proyecto.** Si `ignoreDeprecations` esta en "5.0", NO cambiarlo a "6.0".
7. **Cada test debe ser independiente** — no depender del orden de ejecucion ni de estado compartido.
8. **Usar mocks del ORM** (Prisma, TypeORM, etc.) — no conectar a BD real en unit tests.
9. **Cada test DEBE terminar con `await t.flush()`** — sin esto la metadata no llega al reporte.
10. **NO instalar Allure, allure-commandline ni Java** — este sistema usa un reporter custom puro Node.js que genera HTML directamente.
11. **El reporter html-reporter.js DEBE ser JavaScript puro** — Jest carga reporters con `require()`, no pasa por ts-jest. Si necesitas modificarlo, no lo conviertas a TypeScript.
12. **Los archivos de spec deben terminar en `.security-html.spec.ts`** — este es el patron que el jest config busca.
