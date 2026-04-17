# Checklist de Ciberseguridad GOES — Referencia completa

Cada item DEBE tener al menos 1 test. El tag de trazabilidad es `GOES Checklist Rxx`.

## Categoria 1: Contenido Web

| ID | Tarea | Epic | Feature | Severity |
|----|-------|------|---------|----------|
| R3 | No dejar contenido sensible (llaves, IDs, info personal) en archivos del proyecto | Seguridad | Sensitive Data Exposure | critical |
| R4 | No exponer JS sensible o logica de negocio | Seguridad | Business Logic Exposure | critical |
| R5 | Sanitizar entrada de datos (probar inyeccion de scripts XSS) | Seguridad | XSS Prevention / Input Sanitization | blocker |
| R6 | Si el sitio es publico, configurar Robot.txt y sitemap.xml | Configuracion | Public Site Config | minor |

## Categoria 2: Entrada y salida de datos por el servidor

| ID | Tarea | Epic | Feature | Severity |
|----|-------|------|---------|----------|
| R8 | Mensajes 2xx, 4xx, 5xx genericos (no exponer detalles internos, stack traces, queries) | Seguridad | Generic Error Messages | critical |
| R9 | Validacion por rol o permiso dentro del servidor | Autenticacion | RBAC Enforcement | blocker |
| R10 | Eliminar todo log visible por el usuario (no exponer console.log en produccion) | Seguridad | Log Exposure Prevention | critical |
| R11 | Validacion de tipo de datos via DTO, longitud de campo, cantidad maxima de registros | Dominio | DTO Validation / Input Constraints | critical |

## Categoria 3: Autenticacion, registro y acciones de usuarios

| ID | Tarea | Epic | Feature | Severity |
|----|-------|------|---------|----------|
| R13 | Tiempo de vida bajo para access token (5 min) y refresh token | Seguridad | Token Lifetime Enforcement | critical |
| R14 | Auth failures: MFA, rate limit, session management | Seguridad | Auth Failure Handling | blocker |
| R15 | Encriptacion bcrypt, Argon2 o PBKDF2 en hashing de passwords | Seguridad | Password Hashing Strength | blocker |
| R16 | Uso de algoritmo RS256 para firma de tokens JWT | Seguridad | JWT Signing Algorithm | blocker |
| R17 | Session ID minimo 128 bits de entropia generado con CSPRNG | Seguridad | Session ID Entropy | critical |
| R18 | Session ID regenerado post-login (previene session fixation) | Seguridad | Session Fixation Prevention | critical |
| R19 | Validar signature, exp, iat, iss, aud en cada request JWT | Seguridad | JWT Claims Validation | blocker |
| R20 | NO almacenar datos sensibles en el payload del JWT | Seguridad | JWT Payload Security | critical |
| R21 | Bloquear navegacion forzada a rutas donde el usuario no deberia entrar | Autenticacion | Forced Browsing Prevention | critical |
| R22 | 404 para cualquier ruta que no coincida con las definidas | Configuracion | Unknown Route Handling | normal |
| R23 | Proteccion contra IDOR (modificar query/uri params no muestra info de otros usuarios) | Seguridad | IDOR Prevention | blocker |
| R24 | Usuarios con bajo privilegio no acceden a acciones de alto privilegio | Autenticacion | Privilege Escalation Prevention | blocker |
| R25 | Mismo usuario no puede registrarse repetidamente | Autenticacion | Duplicate Registration Prevention | critical |
| R26 | No aceptar disposable emails como emails validos | Autenticacion | Disposable Email Rejection | normal |
| R27 | Bloquear cuenta tras 3-5 intentos fallidos | Seguridad | Brute Force Protection | blocker |
| R28 | Implementar metodo de recuperacion de cuenta | Autenticacion | Account Recovery | critical |
| R29 | Si hay "recuerdame", la contrasena debe estar encriptada | Seguridad | Remember Me Security | critical |
| R30 | Almacenamiento de contrasena del lado del servidor (nunca en cliente) | Seguridad | Server-side Password Storage | blocker |
| R31 | No permitir usar el username como password | Seguridad | Weak Password Prevention | critical |
| R32 | Renovar refresh token despues de cada uso (rotacion) | Seguridad | Token Rotation | blocker |
| R33 | Validar token en cada peticion privada | Autenticacion | Token Validation Per Request | blocker |
| R34 | Implementar RBAC (Role-Based Access Control) | Autenticacion | Role-Based Access Control | blocker |
| R35 | Sistema de inactividad: timeout de sesion si no hay actividad | Seguridad | Session Inactivity Timeout | critical |

## Categoria 4: Configuracion

| ID | Tarea | Epic | Feature | Severity |
|----|-------|------|---------|----------|
| R37 | Implementar ORM (no queries raw, prevenir SQL injection) | Seguridad | ORM Usage / SQL Injection Prevention | blocker |
| R38 | CORS: Access-Control-Allow-Origin solo origenes permitidos (no wildcard *) | Configuracion | CORS Origin Restriction | critical |
| R39 | CORS: Access-Control-Allow-Methods solo metodos permitidos (eliminar no usados) | Configuracion | CORS Methods Restriction | critical |
| R40 | Access-Control-Allow-Credentials: true solo si es necesario enviar cookies | Configuracion | CORS Credentials Policy | normal |
| R41 | Access-Control-Max-Age: 3600 (cachear preflight por 1 hora) | Configuracion | CORS Preflight Cache | normal |
| R42 | Cookies con HttpOnly, Secure, SameSite=Strict, Path=/ | Seguridad | Cookie Security Flags | blocker |
| R43 | Debug mode deshabilitado en produccion | Configuracion | Debug Mode Disabled | critical |
| R44 | Content-Security-Policy: default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline' | Configuracion | CSP Header | critical |
| R45 | X-Content-Type-Options: nosniff, X-Frame-Options: DENY (o SAMEORIGIN) | Configuracion | Security Headers (XCT, XFO) | critical |
| R46 | Strict-Transport-Security: max-age=31536000; includeSubDomains; preload | Configuracion | HSTS Header | critical |
| R47 | X-XSS-Protection: 0 (deprecado, usar CSP en su lugar) | Configuracion | XSS Protection Header | normal |
| R48 | Referrer-Policy: strict-origin-when-cross-origin | Configuracion | Referrer Policy Header | normal |
| R49 | Permissions-Policy: geolocation=(), camera=(), microphone=() | Configuracion | Permissions Policy Header | normal |
| R50 | Cache-Control: no-store para respuestas sensibles | Configuracion | Cache Control Header | critical |
| R51 | Cookie max-age y refresh token deben tener la misma duracion | Configuracion | Cookie Lifetime Alignment | critical |
| R52 | Deshabilitar metodo HTTP PUT | Configuracion | HTTP PUT Disabled | normal |
| R53 | Deshabilitar metodo HTTP TRACE | Configuracion | HTTP TRACE Disabled | critical |
| R54 | Deshabilitar http method override | Configuracion | HTTP Override Disabled | critical |
| R55 | Rate limit: 100 req/min normal, 5 req/min login | Seguridad | Rate Limiting | blocker |

## Categoria 5: Manejo de archivos

| ID | Tarea | Epic | Feature | Severity |
|----|-------|------|---------|----------|
| R57 | Validar extension de archivo Y magic byte (content-type puede ser falsificado) | Archivos | File Extension + Magic Byte Validation | blocker |
| R58 | Whitelist de extensiones permitidas (.pdf, .jpg, .png) | Archivos | File Extension Whitelist | blocker |
| R59 | Limitar tamano de archivo (ej: 10MB max) | Archivos | File Size Limit | critical |
| R60 | Renombrar archivos con UUID (nunca usar nombre original) | Archivos | File UUID Rename | critical |

---

## OWASP Top 10 — Mapeo a tests

| ID | Vulnerabilidad | Tests a generar | Tag |
|----|---------------|-----------------|-----|
| A01 | Broken Access Control | IDOR, Privilege Escalation, Forced Browsing, Missing Auth | OWASP A01 |
| A02 | Cryptographic Failures | Weak Hashing, Sensitive Data in JWT, Missing TLS | OWASP A02 |
| A03 | Injection | SQL Injection (ORM bypass), XSS, Command Injection | OWASP A03 |
| A04 | Insecure Design | Business Logic Flaws, Missing Rate Limit | OWASP A04 |
| A05 | Security Misconfiguration | Debug Mode, Default Creds, Missing Headers, CORS wildcard | OWASP A05 |
| A06 | Vulnerable Components | (SCA - fuera del scope de unit tests) | OWASP A06 |
| A07 | Auth Failures | Brute Force, Timing Attack, Weak Password, Token Replay, Enumeration | OWASP A07 |
| A08 | Data Integrity Failures | (CI/CD - fuera del scope de unit tests) | OWASP A08 |
| A09 | Logging Failures | Missing Audit Log, Sensitive Data in Logs | OWASP A09 |
| A10 | SSRF | URL Validation, Whitelist Enforcement | OWASP A10 |

## OWASP API Security Top 10 — Mapeo a tests

| ID | Vulnerabilidad | Tests a generar | Tag |
|----|---------------|-----------------|-----|
| API1 | Broken Object Level Auth | Acceder a recurso de otro usuario por ID | OWASP API1 |
| API2 | Broken Authentication | Rate limit en login, MFA bypass | OWASP API2 |
| API3 | Broken Object Property Auth | No exponer propiedades sensibles, DTOs por rol | OWASP API3 |
| API4 | Unrestricted Resource Consumption | Rate limit, pagination limit, payload max size | OWASP API4 |
| API5 | Broken Function Level Auth | Admin endpoint accedido por user normal | OWASP API5 |
| API6 | Unrestricted Access to Flows | Validar secuencias de negocio | OWASP API6 |
| API7 | Server Side Request Forgery | Whitelist URLs, validar esquemas | OWASP API7 |
| API8 | Security Misconfiguration | Headers, CORS, metodos HTTP innecesarios | OWASP API8 |
| API9 | Improper Inventory Management | Documentar APIs, deprecar versiones antiguas | OWASP API9 |
| API10 | Unsafe Consumption of APIs | Validar respuestas de terceros, timeouts, circuit breakers | OWASP API10 |

---

## Guia GOES — Secciones de referencia

### Seccion 3: Validacion de Entrada
- Validar en el servidor (nunca confiar en el cliente)
- Whitelist sobre Blacklist
- Sanitizar y Escapar
- Validar tipo, longitud, formato y rango
- Canonicalizar antes de validar (UTF-8)

### Seccion 4: Autenticacion y Autorizacion
- 4.1 Politicas de Contrasenas: minimo 12 chars, verificar contra listas comprometidas (HaveIBeenPwned), bcrypt(12)/Argon2(memory=64MB,iterations=3)/PBKDF2
- 4.2 Gestion de Sesiones: 128 bits entropia CSPRNG, regenerar post-login, HttpOnly/Secure/SameSite=Strict/Path=/, timeout inactividad 15-30 min, timeout absoluto 4-8 horas, invalidar en logout server-side
- 4.3 JWT Best Practices: RS256 o ES256 (NUNCA none o HS256 con secreto debil), validar signature/exp/iat/iss/aud, access token 15 min max, refresh en httpOnly cookie con rotacion por uso, NO datos sensibles en payload
- 4.4 Control de Acceso RBAC/ABAC: denegar por defecto, validar en CADA endpoint, no confiar en IDs de URL (verificar ownership), least privilege, log accesos denegados

### Seccion 5: Criptografia
- Hashing passwords: Argon2id, bcrypt, PBKDF2 (NUNCA MD5, SHA1, SHA256 solo)
- TLS 1.2 minimo, ideal 1.3
- NO hardcodear secretos, usar vault (HashiCorp, AWS Secrets Manager)
- .gitignore para archivos sensibles, git-secrets para pre-commit hooks

### Seccion 6: Seguridad en APIs
- Rate Limiting: 5 intentos/min login por IP, 100-1000 req/min APIs, responder 429 + Retry-After
- Token bucket o sliding window algorithm

### Seccion 7: Seguridad en Base de Datos
- SIEMPRE queries parametrizadas / ORM (Prisma, TypeORM, Hibernate)
- NUNCA concatenar strings: "SELECT * FROM users WHERE id = " + userId // NUNCA
- Principio de menor privilegio: usuario BD especifico por app, solo SELECT/INSERT/UPDATE/DELETE, NUNCA root/sa/admin, NO permisos DDL (CREATE, DROP, ALTER)

### Seccion 8: Headers de Seguridad HTTP
- Content-Security-Policy: default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'
- X-Content-Type-Options: nosniff
- X-Frame-Options: DENY (o SAMEORIGIN si necesitas iframes)
- Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
- X-XSS-Protection: 0 (deprecado, usar CSP)
- Referrer-Policy: strict-origin-when-cross-origin
- Permissions-Policy: geolocation=(), camera=(), microphone=()
- Cache-Control: no-store (para respuestas sensibles)
- CORS: origenes especificos (NUNCA *), solo metodos necesarios (GET, POST), credentials solo si necesario, max-age 3600

### Seccion 9: Manejo de Errores y Logging
- 9.1 Errores Seguros: mensajes genericos "Ocurrio un error", NUNCA exponer stack traces/queries SQL/rutas de archivos/versiones de software, IDs de correlacion, loguear detalles en servidor (no al cliente), debug mode deshabilitado en produccion
- 9.2 Que Loguear: intentos login (exitosos y fallidos) con IP y timestamp, cambios password y datos perfil, acceso denegado (403), operaciones admin y cambios de roles, errores validacion de entrada (posibles ataques), creacion/eliminacion de recursos sensibles
- 9.3 Que NO Loguear: contrasenas (ni texto plano ni hasheadas), tokens de sesion/JWTs/API keys, numeros de tarjetas completos, datos personales sensibles (SSN, datos medicos), secretos de configuracion

### Seccion 10: Upload de Archivos
- Validar extension Y magic bytes (content-type puede ser falsificado)
- Whitelist de extensiones (.pdf, .jpg, .png)
- Limitar tamano maximo (ej: 10MB)
- Renombrar con UUID (nunca usar nombre original)
- Almacenar fuera del webroot o en storage externo (S3)
- Escanear con antivirus antes de procesar
- Servir con Content-Disposition: attachment
- NO ejecutar archivos subidos (deshabilitar ejecucion en directorio de uploads)
