# goes-security-skill

Skill de Claude para generar tests de seguridad automatizados con **Allure Report** en proyectos **NestJS + Jest**.

Cubre el **Checklist de Ciberseguridad GOES** (60 items), **OWASP Top 10** y **OWASP API Security Top 10**.

---

## Instalacion

### 1. Clonar este repo

```bash
git clone https://github.com/goes-tools/goes-security-skill.git
```

### 2. Copiar la carpeta `.claude/` a tu proyecto NestJS

```bash
cp -r goes-security-skill/.claude/ /ruta/a/tu-proyecto/.claude/
```

Tu proyecto queda asi:

```
tu-proyecto-nestjs/
  .claude/
    skills/
      goes-security-test/
        SKILL.md                    <-- instrucciones para Claude
        references/
          goes-checklist.md         <-- 60 items del checklist GOES
          test-patterns.md          <-- 7 patrones de codigo
  src/
  package.json
  ...
```

### 3. Abrir Claude en tu proyecto

Abrir **Claude Code** o **Claude Cowork** en la carpeta del proyecto.

### 4. Pedirle a Claude que genere los tests

```
> Genera los tests de seguridad del servicio AuthService
```

Claude lee el skill, analiza el codigo real del servicio, instala las dependencias necesarias, y genera los tests automaticamente.

---

## Que hace Claude cuando activas el skill

1. Analiza los servicios y controllers del proyecto
2. Instala dependencias si faltan (`allure-jest`, `allure-js-commons`, `allure-commandline`)
3. Configura Jest para generar reportes Allure
4. Genera archivos `.spec.ts` completos con:
   - Metadata Allure (epic, feature, story, severity, tags)
   - Trazabilidad triple: `GOES Checklist Rxx` + `OWASP Axx` + `OWASP APIxx`
   - Steps visibles (Preparar / Ejecutar / Verificar)
   - Attachments JSON (payload del atacante + respuesta de defensa)
   - Descriptions Markdown (vulnerabilidad + defensa implementada)

---

## Ejemplos de uso

```
> Genera los tests de seguridad del servicio AuthService
> Genera los tests de seguridad del servicio UsersService
> Genera los tests de seguridad de todos los servicios
```

### Ver el reporte interactivo

```bash
npm test
npx allure generate allure-results --clean -o allure-report
npx allure open allure-report
```

### Filtrar en Allure Report

- **Epic**: Seguridad, Autenticacion, Dominio, Configuracion, Auditoria, Archivos
- **Tag**: `Pentest`, `OWASP A07`, `GOES Checklist R27`, `CRUD`, `Auth`, `Config`
- **Severity**: blocker, critical, normal, minor

---

## Cobertura

### Checklist de Ciberseguridad GOES (60 items)

| Categoria | Items | Que cubre |
|-----------|-------|-----------|
| Contenido Web | R3-R6 | Datos sensibles, XSS, sanitizacion |
| Entrada/Salida del servidor | R8-R11 | Errores genericos, RBAC, DTO validation |
| Autenticacion y sesiones | R13-R35 | JWT, bcrypt, RS256, brute force, IDOR, token rotation, RBAC |
| Configuracion | R37-R55 | CORS, headers HTTP, cookies, rate limit, ORM |
| Manejo de archivos | R57-R60 | Magic bytes, whitelist, tamano, UUID |

### OWASP Top 10

A01 (Broken Access Control), A02 (Crypto Failures), A03 (Injection), A05 (Security Misconfiguration), A07 (Auth Failures), A09 (Logging Failures)

### OWASP API Security Top 10

API1 (Object Level Auth), API2 (Broken Auth), API3 (Property Auth), API4 (Resource Consumption), API5 (Function Level Auth), API8 (Security Misconfiguration)

---

## Requisitos

- **Proyecto NestJS** con Jest configurado
- **Node.js 18+**
- **Java** (para Allure CLI):
  - macOS: `brew install openjdk`
  - Linux: `sudo apt install default-jdk`
  - Windows: descargar de [adoptium.net](https://adoptium.net/)

---

## Estructura del repo

```
goes-security-skill/
  .claude/
    skills/
      goes-security-test/
        SKILL.md                 <-- instrucciones para Claude
        references/
          goes-checklist.md      <-- checklist completo (60 items)
          test-patterns.md       <-- 7 patrones de codigo
  README.md
```

---

## Contribuir

Se aceptan PRs para:

- Agregar nuevos patrones de test en `references/test-patterns.md`
- Actualizar el checklist si cambia la normativa GOES en `references/goes-checklist.md`
- Mejorar las instrucciones del skill en `SKILL.md`
- Agregar soporte para otros frameworks (Express, Fastify, etc.)

### Como contribuir

1. Fork del repo
2. Crear branch: `git checkout -b feature/nuevo-patron`
3. Hacer cambios
4. PR con descripcion de que se agrego/cambio

---

## Licencia

Uso interno — Gobierno de El Salvador (GOES)
