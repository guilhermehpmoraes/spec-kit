# Feature Plan: Admin Educational Group Domain

- **Feature ID**: 004-admin-educational-group-domain
- **Plan ID**: PLAN-004-admin-educational-group-domain
- **Status**: Completed
- **Date**: 2026-04-20
- **Owner**: Guilherme Moraes
- **Feature Folder**: docs/specs/features/004-admin-educational-group-domain/
- **Feature Spec**: docs/specs/features/004-admin-educational-group-domain/feature.spec.md
- **Plan File**: docs/specs/features/004-admin-educational-group-domain/plan.spec.md
- **Task Output Folder**: docs/specs/features/004-admin-educational-group-domain/tasks/
- **Related ADRs**: ADR-004, ADR-005, ADR-010

## 1. Planning Goal

Define the complete implementation approach for the educational group domain in the Admin backend — including database entities, migrations, CRUD endpoints for educational groups, subscriptions, and common users — with enough detail to generate self-sufficient task specs.

## 2. Summary

- **Feature objective**: Deliver the educational group management module in the Admin backend, covering atomic group creation (with auto-provisioned master user, database parameters, first common user, and plan subscriptions), independent subscription and common user management, and full CRUD across three endpoint groups.
- **Why now**: Educational groups are the tenant/client entity that represents schools — the core business entity needed to onboard clients to the platform.
- **Expected outcome**: Admin users can create, view, update, and delete educational groups; manage plan subscriptions independently; and manage common users independently — all through JWT-protected REST endpoints.
- **Implementation direction**: New `educational-group/` domain module under `apps/admin/backend/src/` following existing DDD patterns (Identity, Subscription). Five new database tables with a TypeORM migration. Three sub-modules (groups, subscriptions, common-users) each with their own controller/service/DTOs.

## 3. Technical Context

### Stack Baseline (Satie)

- **Monorepo**: Nx + pnpm
- **Backend**: NestJS
- **Frontend**: N/A (backend-only feature)
- **Database**: PostgreSQL + TypeORM (SnakeNamingStrategy)
- **Tests**: Jest + Supertest (backend integration + unit)

### Feature-Specific Context

- **Touched apps/packages**: `apps/admin/backend/` (new domain module, entities, migration, config), `apps/admin/backend-tests/` (new tests), `packages/utils/` (new shared utility package)
- **New dependencies**: `@satie/utils` (new workspace package — slug, password, naming utilities)
- **Data impact**: 5 new tables (`usuario_mestre`, `parametros_grupo_educacional`, `grupo_educacional`, `usuario_comum`, `assinatura`)
- **Constraints**: Cross-domain FK from `assinatura` to `plano` (Subscription domain)

### Constraints and Assumptions

- **Inputs**: JSON payloads via REST API from authenticated admin users
- **Outputs**: JSON responses with entity details; Scalar API documentation
- **Boundary conditions**: Slug collision (unique constraints), 1-plan-per-product validation, immutable fields after creation, cascade soft-delete
- **Resolved assumptions**:
  - `APP_ENVIRONMENT` env var (values: `dev` or `prod`) — new, added to NestJS ConfigModule
  - Master user password: 16 characters, bcrypt-hashed, **NOT returned in any response** (stored hashed only)
  - Common user password: auto-generated random password at creation (same policy as master user), stored hashed
  - Slug generation strips accents/diacritics to ASCII before kebab-case conversion
  - Cascade soft-delete: deleting a group soft-deletes all related records in a transaction

## 3.1 Technical Design Baseline (Required)

### Database Design

#### Tables and Actions

| Schema | Table | Action | Purpose |
| ------ | ----- | ------ | ------- |
| public | `usuario_mestre` | Create | Master user credentials per educational group |
| public | `parametros_grupo_educacional` | Create | Database parameters (nome_banco) per group |
| public | `grupo_educacional` | Create | Core tenant entity — educational group |
| public | `usuario_comum` | Create | Common users within a group |
| public | `assinatura` | Create | Plan subscription links (group ↔ plan) |

#### Columns Specification

**Table: `usuario_mestre`**

| Campo | Tipo SQL | Nullable | Default | PK | FK Ref | Unique | Index | Notes |
| ----- | -------- | -------- | ------- | -- | ------ | ------ | ----- | ----- |
| `id_usuario_mestre` | uuid | No | gen_random_uuid() | Yes | N/A | Yes | PK | Primary key |
| `usuario` | varchar(255) | No | N/A | No | N/A | Yes | UQ | Auto-generated: `mestre-{env}-{slug}` |
| `senha` | varchar(255) | No | N/A | No | N/A | No | N/A | Bcrypt-hashed, 16-char random |
| `criado_por` | varchar(255) | No | N/A | No | N/A | No | N/A | EntidadeBase |
| `modificado_por` | varchar(255) | No | N/A | No | N/A | No | N/A | EntidadeBase |
| `criado_as` | timestamp | No | NOW() | No | N/A | No | N/A | EntidadeBase |
| `modificado_as` | timestamp | No | NOW() | No | N/A | No | N/A | EntidadeBase |
| `deletado_as` | timestamp | Yes | NULL | No | N/A | No | N/A | EntidadeBase soft delete |
| `deletado_por` | varchar(255) | Yes | NULL | No | N/A | No | N/A | EntidadeBase |

**Table: `parametros_grupo_educacional`**

| Campo | Tipo SQL | Nullable | Default | PK | FK Ref | Unique | Index | Notes |
| ----- | -------- | -------- | ------- | -- | ------ | ------ | ----- | ----- |
| `id_parametros_grupo_educacional` | uuid | No | gen_random_uuid() | Yes | N/A | Yes | PK | Primary key |
| `nome_banco` | varchar(255) | No | N/A | No | N/A | Yes | UQ | Auto-generated: `db_{env}_{slug}` |
| `criado_por` | varchar(255) | No | N/A | No | N/A | No | N/A | EntidadeBase |
| `modificado_por` | varchar(255) | No | N/A | No | N/A | No | N/A | EntidadeBase |
| `criado_as` | timestamp | No | NOW() | No | N/A | No | N/A | EntidadeBase |
| `modificado_as` | timestamp | No | NOW() | No | N/A | No | N/A | EntidadeBase |
| `deletado_as` | timestamp | Yes | NULL | No | N/A | No | N/A | EntidadeBase soft delete |
| `deletado_por` | varchar(255) | Yes | NULL | No | N/A | No | N/A | EntidadeBase |

**Table: `grupo_educacional`**

| Campo | Tipo SQL | Nullable | Default | PK | FK Ref | Unique | Index | Notes |
| ----- | -------- | -------- | ------- | -- | ------ | ------ | ----- | ----- |
| `id_grupo_educacional` | uuid | No | gen_random_uuid() | Yes | N/A | Yes | PK | Primary key |
| `descricao` | varchar(255) | No | N/A | No | N/A | No | N/A | Group description/name |
| `id_usuario_mestre` | uuid | No | N/A | No | usuario_mestre(id_usuario_mestre) | Yes | UQ + FK | 1:1 relationship |
| `id_parametros_grupo_educacional` | uuid | No | N/A | No | parametros_grupo_educacional(id_parametros_grupo_educacional) | Yes | UQ + FK | 1:1 relationship |
| `criado_por` | varchar(255) | No | N/A | No | N/A | No | N/A | EntidadeBase |
| `modificado_por` | varchar(255) | No | N/A | No | N/A | No | N/A | EntidadeBase |
| `criado_as` | timestamp | No | NOW() | No | N/A | No | N/A | EntidadeBase |
| `modificado_as` | timestamp | No | NOW() | No | N/A | No | N/A | EntidadeBase |
| `deletado_as` | timestamp | Yes | NULL | No | N/A | No | N/A | EntidadeBase soft delete |
| `deletado_por` | varchar(255) | Yes | NULL | No | N/A | No | N/A | EntidadeBase |

**Table: `usuario_comum`**

| Campo | Tipo SQL | Nullable | Default | PK | FK Ref | Unique | Index | Notes |
| ----- | -------- | -------- | ------- | -- | ------ | ------ | ----- | ----- |
| `id_usuario_comum` | uuid | No | gen_random_uuid() | Yes | N/A | Yes | PK | Primary key |
| `email` | varchar(255) | No | N/A | No | N/A | No | N/A | User email |
| `senha` | varchar(255) | No | N/A | No | N/A | No | N/A | Bcrypt-hashed, auto-generated |
| `verificado` | boolean | No | false | No | N/A | No | N/A | Future email verification |
| `id_grupo_educacional` | uuid | No | N/A | No | grupo_educacional(id_grupo_educacional) | No | FK + IDX | N:1 relationship |
| `criado_por` | varchar(255) | No | N/A | No | N/A | No | N/A | EntidadeBase |
| `modificado_por` | varchar(255) | No | N/A | No | N/A | No | N/A | EntidadeBase |
| `criado_as` | timestamp | No | NOW() | No | N/A | No | N/A | EntidadeBase |
| `modificado_as` | timestamp | No | NOW() | No | N/A | No | N/A | EntidadeBase |
| `deletado_as` | timestamp | Yes | NULL | No | N/A | No | N/A | EntidadeBase soft delete |
| `deletado_por` | varchar(255) | Yes | NULL | No | N/A | No | N/A | EntidadeBase |

**Table: `assinatura`**

| Campo | Tipo SQL | Nullable | Default | PK | FK Ref | Unique | Index | Notes |
| ----- | -------- | -------- | ------- | -- | ------ | ------ | ----- | ----- |
| `id_assinatura` | uuid | No | gen_random_uuid() | Yes | N/A | Yes | PK | Primary key |
| `id_grupo_educacional` | uuid | No | N/A | No | grupo_educacional(id_grupo_educacional) | No | FK + IDX | Part of unique constraint |
| `id_plano` | uuid | No | N/A | No | plano(id_plano) | No | FK | Part of unique constraint (cross-domain) |
| `criado_por` | varchar(255) | No | N/A | No | N/A | No | N/A | EntidadeBase |
| `modificado_por` | varchar(255) | No | N/A | No | N/A | No | N/A | EntidadeBase |
| `criado_as` | timestamp | No | NOW() | No | N/A | No | N/A | EntidadeBase |
| `modificado_as` | timestamp | No | NOW() | No | N/A | No | N/A | EntidadeBase |
| `deletado_as` | timestamp | Yes | NULL | No | N/A | No | N/A | EntidadeBase soft delete |
| `deletado_por` | varchar(255) | Yes | NULL | No | N/A | No | N/A | EntidadeBase |

#### Constraints and Indexes

- **UQ_usuario_mestre_usuario**: UNIQUE on `usuario_mestre(usuario)` — prevents slug collision
- **UQ_parametros_grupo_educacional_nome_banco**: UNIQUE on `parametros_grupo_educacional(nome_banco)` — prevents slug collision
- **UQ_grupo_educacional_id_usuario_mestre**: UNIQUE on `grupo_educacional(id_usuario_mestre)` — 1:1
- **UQ_grupo_educacional_id_parametros**: UNIQUE on `grupo_educacional(id_parametros_grupo_educacional)` — 1:1
- **FK_grupo_educacional_usuario_mestre**: FK `grupo_educacional(id_usuario_mestre)` → `usuario_mestre(id_usuario_mestre)`
- **FK_grupo_educacional_parametros**: FK `grupo_educacional(id_parametros_grupo_educacional)` → `parametros_grupo_educacional(id_parametros_grupo_educacional)`
- **FK_usuario_comum_grupo_educacional**: FK `usuario_comum(id_grupo_educacional)` → `grupo_educacional(id_grupo_educacional)`
- **FK_assinatura_grupo_educacional**: FK `assinatura(id_grupo_educacional)` → `grupo_educacional(id_grupo_educacional)`
- **FK_assinatura_plano**: FK `assinatura(id_plano)` → `plano(id_plano)` (cross-domain)
- **UQ_assinatura_grupo_plano**: UNIQUE on `assinatura(id_grupo_educacional, id_plano)` — prevents duplicate plan per group
- **IDX_usuario_comum_grupo**: INDEX on `usuario_comum(id_grupo_educacional)` — efficient lookup by group
- **IDX_assinatura_grupo**: INDEX on `assinatura(id_grupo_educacional)` — efficient lookup by group

#### Migration Strategy

- **Migration file**: `apps/admin/backend/src/migrations/1713100000003-CreateEducationalGroupTables.ts`
- **Forward migration** (ordered by FK dependencies):
  1. Create `usuario_mestre` table
  2. Create `parametros_grupo_educacional` table
  3. Create `grupo_educacional` table (FKs to 1 and 2)
  4. Create `usuario_comum` table (FK to 3)
  5. Create `assinatura` table (FKs to 3 and `plano`)
- **Rollback** (reverse order):
  1. Drop `assinatura`
  2. Drop `usuario_comum`
  3. Drop `grupo_educacional`
  4. Drop `parametros_grupo_educacional`
  5. Drop `usuario_mestre`
- **Backfill**: N/A (new tables, no existing data)
- **Compatibility**: Forward-only — no coexistence window needed

### API and Integration Contracts

#### Endpoint Group 1: Educational Groups (`/grupos-educacionais`)

**POST /grupos-educacionais** — Create educational group

Request:

| Field | Type | Required | Validation | Notes |
| ----- | ---- | -------- | ---------- | ----- |
| `descricao` | string | Yes | min 1, max 255 | Group name/description |
| `email` | string | Yes | valid email format | Email for the first common user |
| `planoIds` | string[] | Yes | array of UUIDs, min 1 item | Plan IDs to subscribe |

Response (201):

| Field | Type | Nullable | Source | Notes |
| ----- | ---- | -------- | ------ | ----- |
| `idGrupoEducacional` | string | No | DB | UUID |
| `descricao` | string | No | DB | Group description |
| `usuarioMestre` | object | No | DB | `{ usuario }` — password NOT included |
| `parametros` | object | No | DB | `{ nomeBanco }` |
| `assinaturas` | object[] | No | DB | `[{ idAssinatura, idPlano }]` |
| `usuarioComum` | object | No | DB | `{ idUsuarioComum, email, verificado }` |
| `criadoAs` | string | No | DB | ISO timestamp |

Error responses:
- 400: Validation error (missing fields, invalid email, empty planoIds)
- 400: "Não é permitido mais de um plano por produto" (duplicate product in plans)
- 401: Unauthorized
- 404: "Plano não encontrado" (invalid plan ID)
- 409: "Nome de banco já existe" or "Usuário mestre já existe" (slug collision)

**GET /grupos-educacionais** — List educational groups

Response (200): Array of:

| Field | Type | Nullable | Notes |
| ----- | ---- | -------- | ----- |
| `idGrupoEducacional` | string | No | UUID |
| `descricao` | string | No | |
| `criadoAs` | string | No | ISO timestamp |

**GET /grupos-educacionais/:id** — Get educational group details

Response (200):

| Field | Type | Nullable | Notes |
| ----- | ---- | -------- | ----- |
| `idGrupoEducacional` | string | No | UUID |
| `descricao` | string | No | |
| `usuarioMestre` | object | No | `{ idUsuarioMestre, usuario }` — no password |
| `parametros` | object | No | `{ idParametrosGrupoEducacional, nomeBanco }` |
| `assinaturas` | object[] | No | `[{ idAssinatura, idPlano, plano: { nome, idProduto } }]` |
| `usuariosComuns` | object[] | No | `[{ idUsuarioComum, email, verificado }]` |
| `criadoAs` | string | No | ISO timestamp |
| `modificadoAs` | string | No | ISO timestamp |

**PATCH /grupos-educacionais/:id** — Update educational group

Request:

| Field | Type | Required | Validation | Notes |
| ----- | ---- | -------- | ---------- | ----- |
| `descricao` | string | No | min 1, max 255 | Only editable field on the group itself |

Response (200): Same shape as GET by ID response.

**DELETE /grupos-educacionais/:id** — Soft-delete educational group

Response: 204 No Content. Cascade soft-deletes all related records.

#### Endpoint Group 2: Subscriptions (`/assinaturas`)

**GET /assinaturas?idGrupoEducacional=:id** — List subscriptions for a group

Response (200): Array of:

| Field | Type | Notes |
| ----- | ---- | ----- |
| `idAssinatura` | string | UUID |
| `idGrupoEducacional` | string | UUID |
| `idPlano` | string | UUID |
| `plano` | object | `{ nome, idProduto }` |

**PATCH /assinaturas/:idGrupoEducacional** — Replace plan subscriptions

Request:

| Field | Type | Required | Validation | Notes |
| ----- | ---- | -------- | ---------- | ----- |
| `planoIds` | string[] | Yes | array of UUIDs, min 1 item | New plan IDs (replaces all) |

Response (200): Updated array of subscriptions (same shape as GET).

Error responses:
- 400: "Não é permitido mais de um plano por produto"
- 401: Unauthorized
- 404: "Grupo educacional não encontrado" or "Plano não encontrado"

#### Endpoint Group 3: Common Users (`/usuarios-comuns`)

**POST /usuarios-comuns** — Create common user

Request:

| Field | Type | Required | Validation | Notes |
| ----- | ---- | -------- | ---------- | ----- |
| `email` | string | Yes | valid email format | |
| `idGrupoEducacional` | string | Yes | UUID | Must reference existing group |

Response (201):

| Field | Type | Notes |
| ----- | ---- | ----- |
| `idUsuarioComum` | string | UUID |
| `email` | string | |
| `verificado` | boolean | Always false |
| `idGrupoEducacional` | string | UUID |
| `criadoAs` | string | ISO timestamp |

**GET /usuarios-comuns?idGrupoEducacional=:id** — List common users for a group

Response (200): Array of common user objects (same fields as POST response).

**GET /usuarios-comuns/:id** — Get common user by ID

Response (200): Single common user object.

**PATCH /usuarios-comuns/:id** — Update common user

Request:

| Field | Type | Required | Validation | Notes |
| ----- | ---- | -------- | ---------- | ----- |
| `email` | string | No | valid email format | |
| `senha` | string | No | min 8 chars | New password (will be hashed) |

Response (200): Updated common user object.

**DELETE /usuarios-comuns/:id** — Soft-delete common user

Response: 204 No Content.

### Utility Functions

#### Slug Generation

Function: `generateSlug(descricao: string): string`
- Normalize Unicode (NFD) → strip combining diacritical marks (accents)
- Lowercase
- Replace spaces and non-alphanumeric chars with hyphens
- Collapse consecutive hyphens
- Trim leading/trailing hyphens
- Example: "Colégio São Paulo CCCL" → "colegio-sao-paulo-cccl"

#### Environment-Based Naming

Function: `generateMasterUsername(slug: string, env: string): string` → `mestre-{env}-{slug}`
Function: `generateDatabaseName(slug: string, env: string): string` → `db_{env}_{slug}`

- `env` comes from `ConfigService.get('APP_ENVIRONMENT')` with default `'dev'`

#### Password Generation

Function: `generateRandomPassword(length: number = 16): string`
- Must contain at least: 1 uppercase, 1 lowercase, 1 digit, 1 special character
- Characters from: `A-Z`, `a-z`, `0-9`, `!@#$%^&*()-_=+[]{}|;:,.<>?`
- Use `crypto.randomBytes` or `crypto.randomInt` for randomness (not Math.random)
- Shuffle the result to avoid predictable character positions

### UI Contracts

N/A — backend-only feature.

## 4. Planning Gates (Step 2)

- [x] Feature spec status is `Approved`.
- [x] Feature scope is explicit and bounded.
- [x] Required contracts are defined or referenced.
- [x] Acceptance criteria are testable.
- [x] Dependencies are completed or properly sequenced.
- [x] Risks and edge cases are reviewed.
- [x] Technical design baseline is complete for all impacted layers.

### Engineering Gates

- [x] KISS-first approach is documented for major slices.
- [x] Pattern choice aligns with ADR baseline or deviation is justified.
- [x] Comment strategy is documented where comments are truly needed.
- [x] Testability is confirmed for slice boundaries.
- [x] Complexity hotspots are identified with mitigation.

## 5. Scope-to-Execution Mapping

### Slice 0 — Shared Utilities Package (`@satie/utils`)

- **Goal**: Create a new shared workspace package `@satie/utils` at `packages/utils/` with reusable utility functions: slug generation, random password generation, and environment-based naming.
- **Files to create**:
  - `packages/utils/package.json`
  - `packages/utils/project.json`
  - `packages/utils/tsconfig.json`
  - `packages/utils/tsconfig.lib.json`
  - `packages/utils/src/index.ts`
  - `packages/utils/src/slug.ts`
  - `packages/utils/src/password.ts`
  - `packages/utils/src/naming.ts`
- **Files to modify**:
  - `tsconfig.base.json` — add `@satie/utils` path mapping
- **Requirements covered**: FR-002, FR-003, FR-004
- **Acceptance covered**: S1-2, S1-3, S1-4
- **Expected outputs**: Working shared package with exported utility functions

### Slice 1 — Database Entities and Migration

- **Goal**: Create all 5 TypeORM entities and the migration file. Register entities in app.module and typeorm.config.
- **Files to create**:
  - `apps/admin/backend/src/educational-group/entities/usuario-mestre.entity.ts`
  - `apps/admin/backend/src/educational-group/entities/parametros-grupo-educacional.entity.ts`
  - `apps/admin/backend/src/educational-group/entities/grupo-educacional.entity.ts`
  - `apps/admin/backend/src/educational-group/entities/usuario-comum.entity.ts`
  - `apps/admin/backend/src/educational-group/entities/assinatura.entity.ts`
  - `apps/admin/backend/src/migrations/1713100000003-CreateEducationalGroupTables.ts`
- **Files to modify**:
  - `apps/admin/backend/src/app/app.module.ts` — add entities to TypeORM forRootAsync config
  - `apps/admin/backend/src/config/typeorm.config.ts` — add entities to DataSource config
- **Requirements covered**: FR-009
- **Acceptance covered**: All (foundation for all scenarios)
- **Expected outputs**: Entity files, migration file, updated module config

### Slice 2 — Module Setup, Utilities, and Educational Group CRUD

- **Goal**: Create the `EducationalGroupModule` with three sub-modules. Implement the groups sub-module with full CRUD including atomic creation, slug generation, password generation, and cascade soft-delete.
- **Files to create**:
  - `apps/admin/backend/src/educational-group/educational-group.module.ts`
  - `apps/admin/backend/src/educational-group/groups/groups.module.ts`
  - `apps/admin/backend/src/educational-group/groups/groups.controller.ts`
  - `apps/admin/backend/src/educational-group/groups/groups.service.ts`
  - `apps/admin/backend/src/educational-group/groups/dto/create-group.dto.ts`
  - `apps/admin/backend/src/educational-group/groups/dto/update-group.dto.ts`
  - `apps/admin/backend/src/educational-group/groups/dto/group-response.dto.ts`
  - `apps/admin/backend/src/educational-group/groups/dto/group-detail-response.dto.ts`
  - `apps/admin/backend/src/educational-group/groups/dto/group-list-response.dto.ts`
  - (utilities imported from `@satie/utils` — see Slice 0)
- **Files to modify**:
  - `apps/admin/backend/src/app/app.module.ts` — import `EducationalGroupModule`
- **Requirements covered**: FR-001, FR-002, FR-003, FR-004, FR-005, FR-006, FR-007, FR-008, FR-010, FR-011
- **Acceptance covered**: S1 (all), S2 (all), S5 (all), S6 (all)
- **Expected outputs**: Working CRUD endpoints for groups with full atomic creation

### Slice 3 — Subscription Independent CRUD

- **Goal**: Implement the subscriptions sub-module for independent plan management per group.
- **Files to create**:
  - `apps/admin/backend/src/educational-group/subscriptions/subscriptions.module.ts`
  - `apps/admin/backend/src/educational-group/subscriptions/subscriptions.controller.ts`
  - `apps/admin/backend/src/educational-group/subscriptions/subscriptions.service.ts`
  - `apps/admin/backend/src/educational-group/subscriptions/dto/update-subscriptions.dto.ts`
  - `apps/admin/backend/src/educational-group/subscriptions/dto/subscription-response.dto.ts`
- **Requirements covered**: FR-006, FR-007, FR-008
- **Acceptance covered**: S3 (all)
- **Expected outputs**: Working subscription endpoints

### Slice 4 — Common User Independent CRUD

- **Goal**: Implement the common-users sub-module for independent user management per group.
- **Files to create**:
  - `apps/admin/backend/src/educational-group/common-users/common-users.module.ts`
  - `apps/admin/backend/src/educational-group/common-users/common-users.controller.ts`
  - `apps/admin/backend/src/educational-group/common-users/common-users.service.ts`
  - `apps/admin/backend/src/educational-group/common-users/dto/create-common-user.dto.ts`
  - `apps/admin/backend/src/educational-group/common-users/dto/update-common-user.dto.ts`
  - `apps/admin/backend/src/educational-group/common-users/dto/common-user-response.dto.ts`
- **Requirements covered**: FR-007, FR-008
- **Acceptance covered**: S4 (all)
- **Expected outputs**: Working common user endpoints

### Slice 5 — Tests

- **Goal**: Unit tests for utility functions and service logic; integration tests for all CRUD endpoints.
- **Files to create**:
  - (unit tests for utils live in `packages/utils-tests/` or alongside the package — see T001)
  - `apps/admin/backend-tests/src/backend/educational-group/groups.spec.ts`
  - `apps/admin/backend-tests/src/backend/educational-group/subscriptions.spec.ts`
  - `apps/admin/backend-tests/src/backend/educational-group/common-users.spec.ts`
- **Requirements covered**: All FRs
- **Acceptance covered**: All scenarios
- **Expected outputs**: Passing unit and integration test suites

### Slice 6 — Documentation Updates

- **Goal**: Create domain spec, update app architecture, and update feature spec status.
- **Files to create**:
  - `docs/specs/domains/educational-group.md`
- **Files to modify**:
  - `docs/specs/apps/admin/architecture.md` — add Educational Group domain entry and module hierarchy
- **Requirements covered**: N/A (documentation)
- **Expected outputs**: Domain spec file, updated architecture doc

## 6. Task Generation Matrix

| Task ID | Title | Type | Plan Slice | Parallelizable | Dependencies | Requirement Refs | Technical Scope | Output File |
| ------- | ----- | ---- | ---------- | -------------- | ------------ | ---------------- | --------------- | ----------- |
| T001 | Shared utilities package (@satie/utils) | Task | Slice 0 | No | N/A | FR-002, FR-003, FR-004 | New package: slug, password, naming utils + tests | docs/specs/features/004-admin-educational-group-domain/tasks/T001-shared-utils-package.task.spec.md |
| T002 | Database entities and migration | Task | Slice 1 | No | T001 | FR-009 | 5 entity files, 1 migration, update app.module + typeorm.config | docs/specs/features/004-admin-educational-group-domain/tasks/T002-database-entities-migration.task.spec.md |
| T003 | Educational group CRUD | Task | Slice 2 | No | T001, T002 | FR-001–FR-008, FR-010, FR-011 | Module, controller, service, DTOs (imports @satie/utils) | docs/specs/features/004-admin-educational-group-domain/tasks/T003-educational-group-crud.task.spec.md |
| T004 | Subscription independent CRUD | Task | Slice 3 | No | T002, T003 | FR-006, FR-007, FR-008 | Sub-module, controller, service, DTOs | docs/specs/features/004-admin-educational-group-domain/tasks/T004-subscription-crud.task.spec.md |
| T005 | Common user independent CRUD | Task | Slice 4 | Yes (with T004) | T002, T003 | FR-007, FR-008 | Sub-module, controller, service, DTOs | docs/specs/features/004-admin-educational-group-domain/tasks/T005-common-user-crud.task.spec.md |
| T006 | Integration tests | Task | Slice 5 | No | T002, T003, T004, T005 | All FRs | Integration tests for all endpoints | docs/specs/features/004-admin-educational-group-domain/tasks/T006-integration-tests.task.spec.md |
| T007 | Documentation updates | Task | Slice 6 | Yes (with T006) | T003 | N/A | Domain spec, architecture doc update | docs/specs/features/004-admin-educational-group-domain/tasks/T007-documentation.task.spec.md |

## 7. Task File Generation Rules

- Create one file per task under `docs/specs/features/004-admin-educational-group-domain/tasks/`.
- Use naming pattern: `TXXX-<short-title>.task.spec.md`.
- Use `docs/specs/templates/task.spec.md` for every task file.
- Each task must reference the same feature spec and this feature plan.
- Start each task with status `Draft`; set to `Ready` after task-level approval.
- If a task cannot be delivered safely in one branch cycle, split it before approval.
- For data-impact tasks, include explicit table, field, SQL type, nullability, default, PK/FK, constraints, indexes, and migration notes.
- For API/UI-impact tasks, include explicit request/response/UI state contracts.

## 8. Repository Impact

```text
packages/
  utils/                            (NEW — @satie/utils shared package)
    src/
      index.ts
      slug.ts
      password.ts
      naming.ts
apps/
  admin/
    backend/
      src/
        educational-group/         (NEW — entire domain module)
          entities/                 (5 entity files)
          groups/                   (controller, service, DTOs)
          subscriptions/            (controller, service, DTOs)
          common-users/             (controller, service, DTOs)
          educational-group.module.ts
        migrations/                 (1 new migration file)
        app/
          app.module.ts             (MODIFIED — import EducationalGroupModule + entities)
        config/
          typeorm.config.ts         (MODIFIED — add entities)
    backend-tests/
      src/
        backend/
          educational-group/        (NEW — integration tests)
docs/
  specs/
    domains/
      educational-group.md          (NEW — domain spec)
    apps/
      admin/
        architecture.md             (MODIFIED — add domain entry)
```

### Planned Changes by Area

- **Backend**: New `educational-group` domain module with 3 sub-modules (groups, subscriptions, common-users), 5 entities, 3 controllers, 3 services, 8 DTOs, 1 migration
- **Frontend**: N/A
- **Shared packages**: New `@satie/utils` package with slug, password, and naming utilities (+ unit tests)
- **Database**: 5 new tables with FKs, unique constraints, and indexes
- **QA**: Unit tests for utils (in packages/utils); integration tests for all 12 endpoints

## 9. Validation Strategy

- **Unit tests**:
  - `generateSlug()`: accents, special chars, spaces, edge cases
  - `generateRandomPassword()`: length, character class requirements, randomness
  - `generateMasterUsername()` / `generateDatabaseName()`: format with env prefix
  - Plan-per-product validation logic in service
- **Integration tests**:
  - Group creation: atomic creation, slug generation, response shape
  - Group creation: duplicate product plan rejection
  - Group update: only descricao updated, immutable fields unchanged
  - Group get by ID: all relations loaded correctly
  - Group list: returns all non-deleted groups
  - Group delete: cascade soft-delete
  - Subscription update: replace plans, 1-per-product validation
  - Common user CRUD: create, read, update, soft-delete
  - Auth: all endpoints reject unauthenticated requests
- **E2E tests**: N/A (backend-only)
- **Non-functional**: N/A

## 10. Rollout and Safety

- **Feature flags**: N/A — new endpoints, no existing behavior impacted
- **Backward compatibility**: Fully compatible — new tables and endpoints only, no changes to existing entities
- **Monitoring/observability**: Standard NestJS logging (existing infrastructure)
- **Rollback plan**: Drop migration reverses all tables. Remove module import from `app.module.ts`.

## 11. Step 2 Completion Checklist

- [x] Plan is approved.
- [x] Technical design baseline is complete for all impacted layers.
- [x] Scope-to-execution mapping covers all feature requirements.
- [x] Task generation matrix is defined and consistent with scope.
- [x] Dependencies between planned slices are explicit.
- [x] Feature spec status is updated to `Planned`.
- [x] Plan is ready to hand off to Step 3 (Task Breakdown).

## 12. Post-Implementation Feedback

_To be completed after feature delivery._

### What worked in planning

-

### What caused friction

-

### Planning standard updates

-
