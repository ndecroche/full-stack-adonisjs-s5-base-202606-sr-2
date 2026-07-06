# Auditoría de documentación — FlowSync / full-stack-adonisjs-master

> **Fecha:** 2026-07-06  
> **Alcance:** documentación técnica y de producto del monorepo (backend, frontend, openspec, docs).  
> **Código de referencia:** estado del repo en la Sesión 5 (starter post S1–S2 con auth + OpenSpec).

## Cómo consultar este artefacto

- **Por tipo de documentación:** busca la sección (§1–§9) o filtra la tabla resumen por columna `Tipo`.
- **Por estado:** usa los valores `Adecuado`, `Parcial`, `Insuficiente`, `Ausente`, `Divergencia`, `Gap de implementación`, `Ambigüedad`, `Fuera de alcance`.
- **Por búsqueda en repo:** `rg "Divergencia" docs/AUDITORIA-DOCUMENTACION.md`

### Tabla resumen

| ID | Tipo de documentación | Estado | Observación (una línea) |
|----|------------------------|--------|-------------------------|
| AUD-01 | README de proyecto | Parcial | El root README está completo pero no permite arrancar el proyecto; falta README del frontend y detalle de errores frecuentes |
| AUD-02 | Arquitectura general | Insuficiente | No hay doc de arquitectura ni diagramas; solo estructura de carpetas |
| AUD-03 | API / endpoints | Parcial | OpenSpec permite integrar auth/users sin `routes.ts`; faltan schema de `user`, formato de errores 422 y `/health` fuera de spec |
| AUD-04 | Docstrings / TSDoc | Parcial | Controllers con comentarios de ruta; modelos y frontend sin documentación inline |
| AUD-05 | ADRs | Ausente | `docs/README.md` anuncia ADRs futuros; no existe ningún ADR en el repo |
| AUD-06 | Guía operacional | Ausente | No hay despliegue, runbooks ni troubleshooting |
| AUD-07 | Convenciones de código | Parcial | `CLAUDE.md` y `backend/README.md` cubren backend; sin `AGENTS.md` ni guía frontend |
| AUD-08 | OpenSpec ↔ código | Divergencia | Auth alineada; `GET /users/active` en spec pero no en código; `/health` en código pero no en spec |
| AUD-09 | PRD ↔ implementación | Gap de implementación | El PRD describe el MVP completo de FlowSync; el código actual solo cubre auth |

### Top 3 carencias que más duelen

1. **Divergencia OpenSpec ↔ código en `GET /api/v1/users/active`.** La spec de `users` documenta el endpoint como existente (con scenarios, filtros por `last_seen_at` y códigos HTTP), pero el backend no lo expone. Importa porque quien confíe en la documentación fallará en runtime y erosiona la credibilidad del flujo spec-driven; es la trampa didáctica más grave del repo (§8).

2. **Sin arquitectura general ni ADRs.** No hay diagrama de componentes, flujo FE↔BE ni registro de decisiones (SQLite, access tokens, transformers, OpenSpec). Importa porque un dev nuevo entiende carpetas pero no el recorrido de una petición ni el *por qué* de las elecciones técnicas; las decisiones quedan dispersas en `CLAUDE.md` (orientado a un solo copiloto) y comentarios sueltos (§2, §5).

3. **API documentada de forma fragmentada para quien no vive en el repo.** OpenSpec cubre bien auth/users para quien lo conoce, pero no hay OpenAPI, el schema de `user` no está en la spec, `/health` solo está en README y un integrador externo no tiene un contrato único importable. Importa porque la integración sin leer código es parcial y depende de saber que OpenSpec existe dentro del monorepo (§3).

### Top 3 cosas que ya están bien

1. **OpenSpec de `authentication` alineada con el código.** Register, login, logout y profile están documentados con rutas, payloads, reglas VineJS, Bearer `oat_`, códigos HTTP y escenarios de error; coincide con controllers y rutas reales. No tocar esta spec al refactorizar: es el modelo a seguir (§8).

2. **README raíz con arranque y mapa de endpoints.** Node 24+, install, `.env`, `generate:key`, migraciones, `npm run dev` en backend y frontend, más tabla de rutas con auth. Permite levantar el proyecto y orientarse sin abrir `routes.ts` (§1, §3).

3. **Convenciones de backend explícitas y coherentes con el código.** `CLAUDE.md` y `backend/README.md` documentan el patrón controller → validator → transformer, prefijo `/api/v1`, `middleware.auth()` y subpath imports; el código los cumple de forma uniforme. Cualquier mejora documental debería extender este patrón, no sustituirlo (§7).

---

## §1 — README de proyecto

**Pregunta:** ¿Permite que alguien nuevo arranque el proyecto desde cero sin preguntar?

| Campo | Valor |
|-------|-------|
| **Estado** | Parcial |
| **Referencias** | `README.md`, `backend/README.md`, `frontend/` (sin README) |
| **Observación** | El README raíz cubre bien el flujo mínimo: requisitos (Node 24+), `npm install`, copiar `.env.example`, `node ace generate:key`, migraciones y `npm run dev` en backend y frontend. Hay un gran error que no permite a un desarrollador nuevo levantar auth y dashboard. No existe `frontend/README.md`|

---

## §2 — Descripción de la arquitectura general

**Pregunta:** ¿Hay un documento que explique componentes y relaciones del sistema?

| Campo | Valor |
|-------|-------|
| **Estado** | Insuficiente |
| **Referencias** | `README.md` (árbol de carpetas), `backend/README.md` (estructura `app/`), `docs/PRD.md` §5, `docs/README.md` |
| **Observación** | No existe un documento de arquitectura (ni C4, ni diagrama de componentes, ni flujo auth FE↔BE). El PRD §5 describe el stack de forma informativa, no la organización del código. `backend/README.md` lista carpetas (`controllers`, `models`, `validators`, `transformers`) pero no explica el flujo de una petición (ruta → middleware → validator → controller → transformer → respuesta) ni la relación con el SPA React. |

---

## §3 — Documentación de la API / endpoints

**Pregunta:** ¿Está documentada para que otro integre sin leer el código de las rutas?

| Campo | Valor |
|-------|-------|
| **Estado** | Parcial |
| **Referencias** | `README.md` §Endpoints, `openspec/specs/authentication/spec.md`, `openspec/specs/users/spec.md`, `backend/start/routes.ts` |
|**Observación:** La API **sí está documentada para integrarse sin `routes.ts` pero solo para endpoints sencillos**, gracias sobre todo a OpenSpec. El estado es **Parcial** porque carece de OpenAPI/ejemplos. Pero un tercero no tendría acceso a OpenSpec por lo que no es suficiente del todo.|

**Endpoints documentados vs implementados**

| Método | Ruta | README | OpenSpec | Código | Nota para integrador |
|--------|------|--------|----------|--------|----------------------|
| GET | `/api/v1/health` | Sí | No | Sí | Usar README; no está en OpenSpec |
| POST | `/api/v1/account/register` | Sí | Sí | Sí | Spec completa de entrada; salida `user` sin schema de campos |
| POST | `/api/v1/account/login` | Sí | Sí | Sí | Igual que register |
| POST | `/api/v1/account/logout` | Sí | Sí | Sí | Documentado Bearer + `{ revoked: true }` |
| GET | `/api/v1/account/profile` | Sí | Sí | Sí | Documentado |
| GET | `/api/v1/users` | Sí | Sí | Sí | Orden `created_at` desc en spec |
| GET | `/api/v1/users/:id` | Sí | Sí | Sí | Incluye `404` en spec |
| GET | `/api/v1/users/active` | Sí | Sí | **No** | **No usar** — spec promete endpoint inexistente que se implementa en otra sección |

---

## §4 — Docstrings y comentarios significativos (TSDoc/JSDoc)

**Pregunta:** ¿Controllers, services y modelos tienen documentación inline útil?

| Campo | Valor |
|-------|-------|
| **Estado** | Parcial |
| **Referencias** | `backend/app/controllers/*.ts`, `backend/app/models/user.ts`, `backend/app/transformers/user_transformer.ts`, `backend/app/validators/auth.ts`, `frontend/src/services/authService.ts` |
| **Observación** | **Backend — controllers:** comentarios de bloque con ruta HTTP y descripción de una línea (`POST /account/login`, etc.). Útil para orientarse, pero **no es TSDoc completo**: sin `@param`, `@returns`, códigos de error ni ejemplos. **Modelo `User`:** sin ningún comentario; campos como `lastSeenAt` no están documentados inline (aunque la spec sí los explica). **`UserTransformer`:** buen comentario de módulo explicando el propósito (no exponer `password`, single source of truth). **Validators:** comentarios mínimos pero claros. **Frontend:** `authService.ts` y componentes React sin JSDoc; la API se infiere del código. **Caso especial:** `users_controller.ts` incluye un comentario de formador sobre `/users/active` pendiente de demo, no documentación de API para consumidores. |

**Evidencia**

- Controllers con `/** GET|POST ... */`: `health_controller.ts`, `access_tokens_controller.ts`, `new_accounts_controller.ts`, `users_controller.ts`, `profiles_controller.ts`.
- `user.ts`: 0 comentarios.
- `authService.ts`: 0 comentarios de documentación.

---

## §5 — Decisiones técnicas registradas (ADRs)

**Pregunta:** ¿Existen ADRs que justifiquen decisiones y alternativas descartadas?

| Campo | Valor |
|-------|-------|
| **Estado** | Ausente |
| **Referencias** | `docs/README.md`, búsqueda `ADR*` en el repo |
| **Observación** | No hay ningún Architecture Decision Record. Decisiones relevantes del proyecto (SQLite vs PostgreSQL, access tokens vs JWT, transformers vs serialización en modelo, OpenSpec como SDD) están implícitas en `CLAUDE.md`, `openspec/config.yaml` y comentarios sueltos, pero **no hay registro formal** con contexto, opciones consideradas, decisión y consecuencias. Tampoco serviría para otros IDEs tipo Cursor, ya que CLAUDE.md es exclusivo de Claude. |

**Decisiones no registradas que convendría ADR**

- Auth por access tokens.
- Base de datos SQLite embebido.
- Patrón controller + VineJS validator + transformer para contrato de salida.
- OpenSpec como fuente de verdad de comportamiento.

---

## §6 — Guía operacional

**Pregunta:** ¿Hay despliegue, runbooks o troubleshooting?

| Campo | Valor |
|-------|-------|
| **Estado** | Ausente |
| **Referencias** | Búsqueda en repo de `deploy`, `runbook`, `troubleshoot`, `despliegue` |
| **Observación** | Solo existe documentación de **desarrollo local** (`npm run dev`, migraciones). No hay: instrucciones de build/deploy a staging/producción; variables de entorno de producción más allá de `.env.example` |

---

## §7 — Convenciones de código del proyecto

**Pregunta:** ¿Hay documento que explique naming, estructura y patrones?

| Campo | Valor |
|-------|-------|
| **Estado** | Parcial |
| **Referencias** | `CLAUDE.md`, `backend/README.md` §Convenciones, `openspec/config.yaml` |
| **Observación** | Cuenta con lo básico en Convenciones del backend dentro de CLAUDE.md, más los detalles de OpenSpec, faltarían ejemplos, patrones, naming a utilizar |

---

## §8 — OpenSpec y trazabilidad con el código

**Pregunta:** ¿La spec está al día? ¿Hay divergencias?

| Campo | Valor |
|-------|-------|
| **Estado** | Divergencia |
| **Referencias** | `openspec/specs/authentication/spec.md`, `openspec/specs/users/spec.md`, `backend/start/routes.ts`, `backend/app/controllers/users_controller.ts`, `backend/app/models/user.ts` |
| **Observación** | La spec de **authentication** está alineada con el código: los cuatro endpoints (`register`, `login`, `logout`, `profile`), validaciones VineJS, actualización de `last_seen_at` en login y serialización sin `password` coinciden con controllers y rutas. La spec de **users** tiene una **divergencia principal**: el requirement "Listado de usuarios activos" documenta `GET /api/v1/users/active` con filtro `last_seen_at` en 24h, pero **no hay ruta ni método `active`** en el backend. El dato sí existe (`last_seen_at` en migración, modelo y actualización en login), lo que hace la divergencia realista: "el dato está, el endpoint no". **Divergencia inversa:** `GET /api/v1/health` está implementado y en README, pero **no aparece en ninguna spec OpenSpec**. Los requirements `index` y `show` de users sí coinciden con el código. |

### Matriz de trazabilidad OpenSpec ↔ código

| Capability | OpenSpec | Código | Estado |
|------------|----------|--------|--------|
| Registro de cuenta | `authentication` → Registro | `NewAccountsController.store` | Alineado |
| Login + `last_seen_at` | `authentication` → Inicio de sesión | `AccessTokensController.store` | Alineado |
| Logout | `authentication` → Cierre de sesión | `AccessTokensController.destroy` | Alineado |
| Perfil | `authentication` → Perfil | `ProfilesController.show` | Alineado |
| Listado usuarios | `users` → Listado | `UsersController.index` | Alineado |
| Usuario por id | `users` → Consulta por id | `UsersController.show` | Alineado |
| Usuarios activos 24h | `users` → Listado activos | **No implementado** | **Divergencia** |
| Health check | **No documentado** | `HealthController.index` | **Gap en spec** |


---
