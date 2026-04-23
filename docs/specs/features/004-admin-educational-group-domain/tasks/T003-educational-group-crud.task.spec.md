# Task Spec: Educational Group CRUD

- **Feature ID**: 004-admin-educational-group-domain
- **Task ID**: T003
- **Status**: Done
- **Type**: Task
- **Parallelizable**: No
- **Parallelization Notes**: Depends on T001 (utils) and T002 (entities). Creates the parent `EducationalGroupModule` that T004 and T005 will extend.
- **Date**: 2026-04-20
- **Owner**: Guilherme Moraes
- **Feature Folder**: docs/specs/features/004-admin-educational-group-domain/
- **Feature Spec**: docs/specs/features/004-admin-educational-group-domain/feature.spec.md
- **Feature Plan**: docs/specs/features/004-admin-educational-group-domain/plan.spec.md
- **Task File**: docs/specs/features/004-admin-educational-group-domain/tasks/T003-educational-group-crud.task.spec.md
- **Related ADRs**: ADR-004, ADR-005, ADR-010
- **Dependencies**: T001, T002

## 1. Purpose

Create the `EducationalGroupModule` with the groups sub-module, implementing full CRUD for educational groups. This includes atomic group creation (with auto-provisioned master user, database parameters, first common user, and plan subscriptions), update (description only), list, get-by-ID (with all relations), and cascade soft-delete. This is the core business logic task for the feature.

## 2. Scope

### In Scope

- `EducationalGroupModule` — domain boundary module importing the groups sub-module (and registering the other sub-modules when T004/T005 are done)
- `GroupsModule` — sub-module with controller, service, and DTOs
- `GroupsController` — 5 endpoints: POST, GET list, GET by ID, PATCH, DELETE
- `GroupsService` — business logic: atomic creation, slug generation, password hashing, cascade soft-delete
- DTOs: `CreateGroupDto`, `UpdateGroupDto`, `GroupResponseDto`, `GroupDetailResponseDto`, `GroupListResponseDto`
- `APP_ENVIRONMENT` config variable integration
- JWT authentication guard on all endpoints
- Scalar API documentation decorators (ADR-010)
- Import `app.module.ts` with `EducationalGroupModule`
- Unit tests for `GroupsService` in `apps/admin/backend-tests/src/unit/`

### Out of Scope

- Subscription sub-module endpoints (T004)
- Common user sub-module endpoints (T005)
- Integration tests (T006)
- Documentation updates (T007)

## 3. Context from Feature Plan

- **Plan slice**: Slice 2 — Module Setup, Utilities, and Educational Group CRUD
- **Requirement refs**: FR-001 (atomic creation), FR-002 (master username format), FR-003 (database name format), FR-004 (password generation), FR-005 (immutable fields), FR-006 (1-plan-per-product), FR-007 (endpoint groups), FR-008 (JWT protection), FR-009 (EntidadeBase), FR-010 (bcrypt hashing), FR-011 (first common user)
- **Acceptance covered**: S1 (all), S2 (all), S5 (all), S6 (all)
- **Affected Paths**:
  - `apps/admin/backend/src/educational-group/educational-group.module.ts` (NEW)
  - `apps/admin/backend/src/educational-group/groups/groups.module.ts` (NEW)
  - `apps/admin/backend/src/educational-group/groups/groups.controller.ts` (NEW)
  - `apps/admin/backend/src/educational-group/groups/groups.service.ts` (NEW)
  - `apps/admin/backend/src/educational-group/groups/dto/create-group.dto.ts` (NEW)
  - `apps/admin/backend/src/educational-group/groups/dto/update-group.dto.ts` (NEW)
  - `apps/admin/backend/src/educational-group/groups/dto/group-response.dto.ts` (NEW)
  - `apps/admin/backend/src/educational-group/groups/dto/group-detail-response.dto.ts` (NEW)
  - `apps/admin/backend/src/educational-group/groups/dto/group-list-response.dto.ts` (NEW)
  - `apps/admin/backend/src/app/app.module.ts` (MODIFIED — import EducationalGroupModule)
  - `apps/admin/backend-tests/src/unit/groups.service.spec.ts` (NEW)
- **Why this task exists**: The groups CRUD is the core functionality of the feature. All other tasks extend or test this foundation.

## 4. Technical Specification (Required)

### 4.1 Backend and API Contracts (when applicable)

#### POST /grupos-educacionais — Create educational group

- **Use case flow**: Request → validate DTO → validate plans (exist, 1-per-product) → generate slug → generate master username + password → hash password → create all entities in transaction → return response
- **Endpoint**: `POST /grupos-educacionais`
- **Auth**: `@UseGuards(JwtAuthGuard)`

**Request contract**:

| Field | Type | Required | Validation | Notes |
| ----- | ---- | -------- | ---------- | ----- |
| `descricao` | string | Yes | `@IsString()`, `@IsNotEmpty()`, `@MaxLength(255)` | Group name/description |
| `email` | string | Yes | `@IsEmail()` | Email for the first common user |
| `planoIds` | string[] | Yes | `@IsArray()`, `@ArrayMinSize(1)`, `@IsUUID("4", { each: true })` | Plan IDs to subscribe |

**Response contract (201)**:

| Field | Type | Nullable | Source | Notes |
| ----- | ---- | -------- | ------ | ----- |
| `idGrupoEducacional` | string | No | DB | UUID |
| `descricao` | string | No | DB | Group description |
| `usuarioMestre` | object | No | DB | `{ usuario }` — password NOT included |
| `parametros` | object | No | DB | `{ nomeBanco }` |
| `assinaturas` | object[] | No | DB | `[{ idAssinatura, idPlano }]` |
| `usuarioComum` | object | No | DB | `{ idUsuarioComum, email, verificado }` |
| `criadoAs` | string | No | DB | ISO timestamp |

**Error contract**:
- 400: Validation error (missing fields, invalid email, empty planoIds) — NestJS default validation pipe
- 400: `"Não é permitido mais de um plano por produto"` — duplicate product in plans
- 401: Unauthorized — missing/invalid JWT
- 404: `"Plano não encontrado"` — invalid plan ID
- 409: `"Nome de banco já existe"` or `"Usuário mestre já existe"` — slug collision

#### GET /grupos-educacionais — List educational groups

- **Endpoint**: `GET /grupos-educacionais`
- **Auth**: `@UseGuards(JwtAuthGuard)`

**Response contract (200)**: Array of:

| Field | Type | Nullable | Notes |
| ----- | ---- | -------- | ----- |
| `idGrupoEducacional` | string | No | UUID |
| `descricao` | string | No | |
| `criadoAs` | string | No | ISO timestamp |

#### GET /grupos-educacionais/:id — Get educational group details

- **Endpoint**: `GET /grupos-educacionais/:id`
- **Auth**: `@UseGuards(JwtAuthGuard)`

**Response contract (200)**:

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

**Error contract**:
- 401: Unauthorized
- 404: `"Grupo educacional não encontrado"`

#### PATCH /grupos-educacionais/:id — Update educational group

- **Endpoint**: `PATCH /grupos-educacionais/:id`
- **Auth**: `@UseGuards(JwtAuthGuard)`

**Request contract**:

| Field | Type | Required | Validation | Notes |
| ----- | ---- | -------- | ---------- | ----- |
| `descricao` | string | No | `@IsOptional()`, `@IsString()`, `@IsNotEmpty()`, `@MaxLength(255)` | Only editable field on the group itself |

**Response contract (200)**: Same shape as GET by ID response.

**Error contract**:
- 400: Validation error
- 401: Unauthorized
- 404: `"Grupo educacional não encontrado"`

#### DELETE /grupos-educacionais/:id — Soft-delete educational group

- **Endpoint**: `DELETE /grupos-educacionais/:id`
- **Auth**: `@UseGuards(JwtAuthGuard)`

**Response**: 204 No Content.

Cascade soft-deletes in a transaction:
1. Soft-delete all `assinatura` records for the group
2. Soft-delete all `usuario_comum` records for the group
3. Soft-delete the `grupo_educacional` record
4. Soft-delete the `parametros_grupo_educacional` record
5. Soft-delete the `usuario_mestre` record

**Error contract**:
- 401: Unauthorized
- 404: `"Grupo educacional não encontrado"`

### 4.2 Database Specification (mandatory when data impact exists)

N/A — Database entities and migration are handled in T002. This task consumes the entities via TypeORM repositories.

### 4.3 Frontend and UX Contracts (when applicable)

N/A — Backend-only feature.

### 4.4 Cross-Cutting Constraints

- **Security/authorization**: All endpoints protected by `JwtAuthGuard` from the Identity domain. Passwords are bcrypt-hashed before storage (reuse `bcrypt` library already used by Identity domain).
- **Performance expectations**: Atomic creation runs in a single database transaction to ensure consistency. Cascade soft-delete also runs in a transaction.
- **Observability**: Standard NestJS logging (existing infrastructure).
- **Compatibility constraints**: `APP_ENVIRONMENT` env var must be available via `ConfigService` (values: `dev` or `prod`, default `dev`).

## 5. Implementation Steps

1. **Create `EducationalGroupModule`** at `apps/admin/backend/src/educational-group/educational-group.module.ts`:
   - Import `GroupsModule`
   - Later T004/T005 will add `SubscriptionsModule` and `CommonUsersModule`

2. **Create `GroupsModule`** at `apps/admin/backend/src/educational-group/groups/groups.module.ts`:
   - Import `TypeOrmModule.forFeature([GrupoEducacional, UsuarioMestre, ParametrosGrupoEducacional, UsuarioComum, Assinatura, Plano])`
   - Import `ConfigModule`
   - Provide `GroupsService`
   - Export `GroupsService` if needed by other sub-modules

3. **Create DTOs** at `apps/admin/backend/src/educational-group/groups/dto/`:
   - `create-group.dto.ts` — `CreateGroupDto` with `descricao`, `email`, `planoIds` with class-validator decorators and `@ApiProperty` with description + example
   - `update-group.dto.ts` — `UpdateGroupDto` with optional `descricao`
   - `group-response.dto.ts` — `GroupResponseDto` for creation response
   - `group-detail-response.dto.ts` — `GroupDetailResponseDto` for GET by ID response (includes all relations)
   - `group-list-response.dto.ts` — `GroupListResponseDto` for GET list response

4. **Create `GroupsService`** at `apps/admin/backend/src/educational-group/groups/groups.service.ts`:
   - Inject repositories for all 5 entities + `Plano`, inject `ConfigService`
   - `create(dto: CreateGroupDto)`:
     1. Fetch plans by IDs — throw 404 if any not found
     2. Validate 1-plan-per-product: group plans by `idProduto`, throw 400 if any product has >1 plan
     3. Generate slug from `descricao` using `generateSlug` from `@satie/utils`
     4. Get `env` from `ConfigService.get('APP_ENVIRONMENT')` with default `'dev'`
     5. Generate master username and database name using `@satie/utils`
     6. Generate random password using `generateRandomPassword` from `@satie/utils`
     7. Hash password with bcrypt
     8. Open transaction via `DataSource.transaction()`:
        - Create and save `UsuarioMestre`
        - Create and save `ParametrosGrupoEducacional`
        - Create and save `GrupoEducacional` (linking to above)
        - Create and save `UsuarioComum` (with email from DTO, auto-generated hashed password, `verificado = false`)
        - Create and save `Assinatura` records for each plan
     9. Return created group with relations
   - `findAll()`: Query `GrupoEducacional` (non-deleted), return list
   - `findOne(id: string)`: Query with relations (`usuarioMestre`, `parametros`, `assinaturas.plano`, `usuariosComuns`), throw 404 if not found
   - `update(id: string, dto: UpdateGroupDto)`: Find group, update `descricao`, save, return with relations
   - `remove(id: string)`: Find group with relations, soft-delete all in transaction (assinaturas → usuariosComuns → grupoEducacional → parametros → usuarioMestre)

5. **Create `GroupsController`** at `apps/admin/backend/src/educational-group/groups/groups.controller.ts`:
   - `@Controller("grupos-educacionais")`
   - `@UseGuards(JwtAuthGuard)` at class level
   - `@ApiTags("Grupos Educacionais")`
   - `@ApiBearerAuth()`
   - 5 endpoints: POST, GET, GET /:id, PATCH /:id, DELETE /:id
   - Each endpoint has `@ApiResponse` decorators with typed response DTOs

6. **Update `app.module.ts`**: Import `EducationalGroupModule`

7. **Write unit tests** at `apps/admin/backend-tests/src/unit/groups.service.spec.ts`:
   - Test `create()`: mock repositories and `ConfigService`, verify transaction, slug generation, password hashing, 1-per-product validation
   - Test `findAll()`: verify query returns non-deleted groups
   - Test `findOne()`: verify relations loaded, 404 on not found
   - Test `update()`: verify only `descricao` updated
   - Test `remove()`: verify cascade soft-delete order

## 6. Acceptance Criteria

- AC-S1-1: POST creates `grupo_educacional`, `parametros_grupo_educacional`, `usuario_mestre`, `usuario_comum`, and `assinatura` records atomically in a single transaction.
- AC-S1-2: POST rejects requests with plans from the same product with `"Não é permitido mais de um plano por produto"`.
- AC-S1-3: With `APP_ENVIRONMENT=prod`, `nome_banco = "db_prod_colegio-cccl"` and `usuario = "mestre-prod-colegio-cccl"` for description "Colegio CCCL".
- AC-S1-4: Generated password contains at least one uppercase, one lowercase, one digit, and one symbol.
- AC-S2-1: PATCH updates only `descricao`; `usuario_mestre` and `nome_banco` remain unchanged.
- AC-S2-2: PATCH ignores `usuarioMestre` or `nomeBanco` fields if sent (they are not in the DTO).
- AC-S5-1: GET list returns all non-deleted groups with `idGrupoEducacional`, `descricao`, `criadoAs`.
- AC-S5-2: GET by ID returns group details with all relations (`usuarioMestre` without password, `parametros`, `assinaturas` with plan details, `usuariosComuns`).
- AC-S6-1: DELETE soft-deletes the group and all related records (parametros, usuario_mestre, assinaturas, usuario_comum) in a transaction.
- AC-FR-008: All endpoints reject unauthenticated requests with 401.
- AC-FR-010: `usuario_mestre.senha` is stored bcrypt-hashed, never returned in any response.

## 7. Test Scenarios

### Unit Tests (GroupsService)

- **Create — happy path**: Mocked repositories and ConfigService, verify all entities created in correct order within transaction.
- **Create — plan not found**: One plan ID doesn't exist → throws NotFoundException.
- **Create — duplicate product**: Two plans with same `idProduto` → throws BadRequestException with correct message.
- **Create — slug generation**: Description "Colégio São Paulo" produces correct master username and database name.
- **Create — password hashing**: Verify bcrypt.hash is called on the generated password.
- **Create — env variable**: ConfigService returns `'prod'` → naming uses `prod` prefix.
- **Create — default env**: ConfigService returns undefined → naming uses `dev` prefix.
- **FindAll**: Returns array of groups.
- **FindOne — found**: Returns group with all relations loaded.
- **FindOne — not found**: Throws NotFoundException.
- **Update — happy path**: Only `descricao` is updated.
- **Update — not found**: Throws NotFoundException.
- **Remove — happy path**: All related entities are soft-deleted in a transaction.
- **Remove — not found**: Throws NotFoundException.

## 8. Definition of Ready (to start Step 4)

- [x] Status is `Ready`.
- [x] Scope is explicit and bounded.
- [x] Required contracts are defined (API, DTO, event, UI state, schema).
- [x] Technical specification is detailed enough for independent implementation.
- [x] For data-impact tasks, table/field/type/constraint/index/migration details are fully documented.
- [x] Acceptance criteria are testable.
- [x] Open questions are resolved or captured as explicit assumptions.
- [x] All dependency tasks are `Done` (if any dependencies exist).

## 9. Definition of Done

- [x] Status moved to `Done`.
- [x] All related tests (unit and integration) pass — no test failures allowed.
- [x] **Section 10 (Test Evidence) is filled** with the exact command(s) executed and their output summary proving all tests pass.
- [x] Biome reports zero warnings and zero errors on all changed files.
- [x] Acceptance criteria are validated by tests or clear verification evidence.

## 10. Test Evidence

This section is **mandatory** before marking the task as `Done`. Paste the exact test command(s) executed and a summary of their output. If tests were not executed, this section must remain empty and the task **cannot** transition to `Done`.

| # | Command | Result | Timestamp |
| - | ------- | ------ | --------- |
| 1 | `pnpm nx test admin-backend-tests -- --testPathPatterns='groups.service'` | 1 suite passed, 17 tests passed, 0 failed, 0 skipped | 2026-04-20 |
| 2 | `pnpm nx test admin-backend-tests` | 6 suites passed, 65 tests passed, 0 failed, 0 skipped | 2026-04-20 |
| 3 | `npx biome check apps/admin/backend/src/educational-group/ apps/admin/backend-tests/src/unit/groups.service.spec.ts apps/admin/backend/src/app/app.module.ts` | Checked 16 files. No fixes applied. 0 errors. | 2026-04-20 |

**Rules**:
- Every test suite relevant to the task must have a row in this table.
- "Result" must include pass/fail/skip counts copied from actual terminal output.
- If any test fails, the task stays `In Progress` — do not fill this section with failing results and mark Done.
- This section is never pre-filled during Step 3 (Task Breakdown) — it is populated only during Step 4 (Implementation).
- [x] Edge cases listed in this task are covered.
- [x] Links to changed files/PR/tests are registered.
- [x] Feature spec and plan traceability remains intact.

## 11. Status Log

| Date       | Status | Notes |
| ---------- | ------ | ----- |
| 2026-04-20 | Draft  | Task created from approved feature plan |
| 2026-04-20 | Awaiting Dependency | Waiting for T001 and T002 to complete |
| 2026-04-20 | Ready | Dependencies T001 and T002 are Done |
| 2026-04-20 | In Progress | Implementation started |
| 2026-04-20 | Done | All code implemented, 17 unit tests passing, Biome clean |

## 12. Observations

- Follow the Identity domain's `UsersService` and `AuthService` patterns for service structure, dependency injection, and error handling.
- Follow the Subscription domain's `PlansController` for Scalar API documentation decorators.
- The `EducationalGroupModule` is created in this task as the domain boundary module, but only imports `GroupsModule` initially. T004 and T005 will add their sub-modules to it.
- Bcrypt usage: reuse the same `bcrypt` library and hash rounds already used by the Identity domain's `UsersService`.
- The `usuario_comum` created during group creation gets an auto-generated password (same policy as `usuario_mestre`), stored hashed. The password is NOT returned in the response.
