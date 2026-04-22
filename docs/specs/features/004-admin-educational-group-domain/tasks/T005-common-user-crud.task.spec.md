# Task Spec: Common User Independent CRUD

- **Feature ID**: 004-admin-educational-group-domain
- **Task ID**: T005
- **Status**: Done
- **Type**: Task
- **Parallelizable**: Yes
- **Parallelization Notes**: Can be developed in parallel with T004 (subscription CRUD). Both depend on T003 (educational group CRUD) being done, but do not depend on each other.
- **Date**: 2026-04-20
- **Last Updated**: 2026-04-20
- **Owner**: Guilherme Moraes
- **Feature Folder**: docs/specs/features/004-admin-educational-group-domain/
- **Feature Spec**: docs/specs/features/004-admin-educational-group-domain/feature.spec.md
- **Feature Plan**: docs/specs/features/004-admin-educational-group-domain/plan.spec.md
- **Task File**: docs/specs/features/004-admin-educational-group-domain/tasks/T005-common-user-crud.task.spec.md
- **Related ADRs**: ADR-004, ADR-005, ADR-010
- **Dependencies**: T002, T003

## 1. Purpose

Implement the common-users sub-module for independent user management per educational group. This allows admin users to create, list, view, update, and soft-delete common users (usuario_comum) within a group, separately from group creation/editing. New common users get an auto-generated password stored hashed.

## 2. Scope

### In Scope

- `CommonUsersModule` — sub-module with controller, service, and DTOs
- `CommonUsersController` — 5 endpoints: POST, GET list, GET by ID, PATCH, DELETE
- `CommonUsersService` — business logic: create with auto-generated password, list by group, get by ID, update (email/password), soft-delete
- DTOs: `CreateCommonUserDto`, `UpdateCommonUserDto`, `CommonUserResponseDto`
- JWT authentication guard on all endpoints
- Scalar API documentation decorators (ADR-010)
- Register `CommonUsersModule` in `EducationalGroupModule`
- Unit tests for `CommonUsersService`

### Out of Scope

- Educational group CRUD (T003)
- Subscription CRUD (T004)
- Integration tests (T006)
- Email verification flow (`verificado` field is stored but not used)
- Common user authentication (common users are not used for login in this feature)

## 3. Context from Feature Plan

- **Plan slice**: Slice 4 — Common User Independent CRUD
- **Requirement refs**: FR-007 (separate endpoint groups), FR-008 (JWT protection)
- **Acceptance covered**: S4 (all)
- **Affected Paths**:
  - `apps/admin/backend/src/educational-group/common-users/common-users.module.ts` (NEW)
  - `apps/admin/backend/src/educational-group/common-users/common-users.controller.ts` (NEW)
  - `apps/admin/backend/src/educational-group/common-users/common-users.service.ts` (NEW)
  - `apps/admin/backend/src/educational-group/common-users/dto/create-common-user.dto.ts` (NEW)
  - `apps/admin/backend/src/educational-group/common-users/dto/update-common-user.dto.ts` (NEW)
  - `apps/admin/backend/src/educational-group/common-users/dto/common-user-response.dto.ts` (NEW)
  - `apps/admin/backend/src/educational-group/educational-group.module.ts` (MODIFIED — import CommonUsersModule)
  - `apps/admin/backend-tests/src/unit/common-users.service.spec.ts` (NEW)
- **Why this task exists**: Common users must be manageable independently from group creation/editing (feature spec Scenario 4).

## 4. Technical Specification (Required)

### 4.1 Backend and API Contracts (when applicable)

#### POST /usuarios-comuns — Create common user

- **Use case flow**: Request → validate DTO → verify group exists (throw 404) → generate random password → hash password → create user with `verificado = false` → return response
- **Endpoint**: `POST /usuarios-comuns`
- **Auth**: `@UseGuards(JwtAuthGuard)`

**Request contract**:

| Field | Type | Required | Validation | Notes |
| ----- | ---- | -------- | ---------- | ----- |
| `email` | string | Yes | `@IsEmail()` | User email |
| `idGrupoEducacional` | string | Yes | `@IsUUID("4")` | Must reference existing group |

**Response contract (201)**:

| Field | Type | Notes |
| ----- | ---- | ----- |
| `idUsuarioComum` | string | UUID |
| `email` | string | |
| `verificado` | boolean | Always false on creation |
| `idGrupoEducacional` | string | UUID |
| `criadoAs` | string | ISO timestamp |

**Error contract**:
- 400: Validation error (missing fields, invalid email, invalid UUID)
- 401: Unauthorized
- 404: `"Grupo educacional não encontrado"` — invalid group ID

#### GET /usuarios-comuns?idGrupoEducacional=:id — List common users for a group

- **Endpoint**: `GET /usuarios-comuns`
- **Auth**: `@UseGuards(JwtAuthGuard)`

**Query parameters**:

| Field | Type | Required | Validation | Notes |
| ----- | ---- | -------- | ---------- | ----- |
| `idGrupoEducacional` | string | Yes | `@IsUUID("4")` | Filter users by group |

**Response contract (200)**: Array of common user objects (same fields as POST response).

**Error contract**:
- 401: Unauthorized
- 404: `"Grupo educacional não encontrado"`

#### GET /usuarios-comuns/:id — Get common user by ID

- **Endpoint**: `GET /usuarios-comuns/:id`
- **Auth**: `@UseGuards(JwtAuthGuard)`

**Response contract (200)**: Single common user object (same fields as POST response).

**Error contract**:
- 401: Unauthorized
- 404: `"Usuário comum não encontrado"`

#### PATCH /usuarios-comuns/:id — Update common user

- **Endpoint**: `PATCH /usuarios-comuns/:id`
- **Auth**: `@UseGuards(JwtAuthGuard)`

**Request contract**:

| Field | Type | Required | Validation | Notes |
| ----- | ---- | -------- | ---------- | ----- |
| `email` | string | No | `@IsOptional()`, `@IsEmail()` | |
| `senha` | string | No | `@IsOptional()`, `@IsString()`, `@MinLength(8)` | New password (will be hashed) |

**Response contract (200)**: Updated common user object (same fields as POST response).

**Error contract**:
- 400: Validation error
- 401: Unauthorized
- 404: `"Usuário comum não encontrado"`

#### DELETE /usuarios-comuns/:id — Soft-delete common user

- **Endpoint**: `DELETE /usuarios-comuns/:id`
- **Auth**: `@UseGuards(JwtAuthGuard)`

**Response**: 204 No Content.

**Error contract**:
- 401: Unauthorized
- 404: `"Usuário comum não encontrado"`

### 4.2 Database Specification (mandatory when data impact exists)

N/A — Uses entities from T002. No schema changes.

### 4.3 Frontend and UX Contracts (when applicable)

N/A — Backend-only feature.

### 4.4 Cross-Cutting Constraints

- **Security/authorization**: All endpoints protected by `JwtAuthGuard`. Passwords are bcrypt-hashed before storage. Password is never returned in any response.
- **Performance expectations**: No special concerns — standard CRUD operations.
- **Observability**: Standard NestJS logging.
- **Compatibility constraints**: None.

## 5. Implementation Steps

1. **Create `CommonUsersModule`** at `apps/admin/backend/src/educational-group/common-users/common-users.module.ts`:
   - Import `TypeOrmModule.forFeature([UsuarioComum, GrupoEducacional])`
   - Provide `CommonUsersService`

2. **Create DTOs** at `apps/admin/backend/src/educational-group/common-users/dto/`:
   - `create-common-user.dto.ts` — `CreateCommonUserDto` with `email`, `idGrupoEducacional`, class-validator decorators, `@ApiProperty`
   - `update-common-user.dto.ts` — `UpdateCommonUserDto` with optional `email`, optional `senha`, class-validator decorators, `@ApiProperty`
   - `common-user-response.dto.ts` — `CommonUserResponseDto` with `idUsuarioComum`, `email`, `verificado`, `idGrupoEducacional`, `criadoAs`

3. **Create `CommonUsersService`** at `apps/admin/backend/src/educational-group/common-users/common-users.service.ts`:
   - Inject repositories for `UsuarioComum` and `GrupoEducacional`
   - `create(dto: CreateCommonUserDto)`:
     1. Verify group exists (throw NotFoundException if not)
     2. Generate random password using `generateRandomPassword` from `@satie/utils`
     3. Hash password with bcrypt
     4. Create `UsuarioComum` with `email`, hashed password, `verificado = false`, `idGrupoEducacional`
     5. Save and return (without password)
   - `findByGroup(idGrupoEducacional: string)`:
     1. Verify group exists (throw NotFoundException if not)
     2. Query `UsuarioComum` where `idGrupoEducacional` matches
     3. Return users array
   - `findOne(id: string)`:
     1. Find by `idUsuarioComum`
     2. Throw NotFoundException if not found
     3. Return user
   - `update(id: string, dto: UpdateCommonUserDto)`:
     1. Find user (throw NotFoundException if not)
     2. If `email` provided, update it
     3. If `senha` provided, hash it with bcrypt and update
     4. Save and return
   - `remove(id: string)`:
     1. Find user (throw NotFoundException if not)
     2. Soft-delete the user

4. **Create `CommonUsersController`** at `apps/admin/backend/src/educational-group/common-users/common-users.controller.ts`:
   - `@Controller("usuarios-comuns")`
   - `@UseGuards(JwtAuthGuard)` at class level
   - `@ApiTags("Usuários Comuns")`
   - `@ApiBearerAuth()`
   - 5 endpoints: POST, GET (query param), GET /:id, PATCH /:id, DELETE /:id
   - `@ApiResponse` decorators with typed response DTOs

5. **Update `EducationalGroupModule`**: Import `CommonUsersModule`

6. **Write unit tests** at `apps/admin/backend-tests/src/unit/common-users.service.spec.ts`:
   - Test all service methods with mocked repositories
   - Test create with password generation and hashing
   - Test update with optional email and password
   - Test 404 scenarios

## 6. Acceptance Criteria

- AC-S4-1: POST creates a common user with `verificado = false` and linked to the group.
- AC-S4-2: PATCH updates email and/or password. Password is hashed before storage.
- AC-S4-3: DELETE soft-deletes the user (sets `deletadoAs`).
- AC-GET-LIST: GET list returns all non-deleted common users for a group.
- AC-GET-ONE: GET by ID returns a single common user.
- AC-404-GROUP: POST and GET list return 404 when the group doesn't exist.
- AC-404-USER: GET by ID, PATCH, and DELETE return 404 when the user doesn't exist.
- AC-AUTH: All endpoints reject unauthenticated requests with 401.
- AC-PASSWORD: Password is never returned in any response.

## 7. Test Scenarios

### Unit Tests (CommonUsersService)

- **create — happy path**: User created with auto-generated hashed password, `verificado = false`.
- **create — group not found**: Throws NotFoundException.
- **create — password hashing**: Verify bcrypt.hash is called.
- **findByGroup — happy path**: Returns users array.
- **findByGroup — group not found**: Throws NotFoundException.
- **findOne — found**: Returns user.
- **findOne — not found**: Throws NotFoundException.
- **update — email only**: Only email updated.
- **update — password only**: Password hashed and updated.
- **update — both fields**: Email and password updated.
- **update — not found**: Throws NotFoundException.
- **remove — happy path**: User soft-deleted.
- **remove — not found**: Throws NotFoundException.

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
| 1 | `npx jest --config jest.unit.config.js --rootDir .` (from `apps/admin/backend-tests/`) | 7 suites passed, 78 tests passed, 0 failed, 0 skipped | 2026-04-20 |
| 2 | `npx biome check apps/admin/backend/src/educational-group/common-users/ ...` | Checked 8 files. No fixes applied. | 2026-04-20 |
| 3 | `pnpm nx build admin-backend` | webpack compiled successfully | 2026-04-20 |

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
| 2026-04-20 | Awaiting Dependency | Waiting for T002 and T003 to complete |
| 2026-04-20 | Ready | Dependencies T002 and T003 are Done |
| 2026-04-20 | In Progress | Implementation started |
| 2026-04-20 | Done | Implementation complete — all tests pass, Biome clean, build succeeds |

## 12. Observations

- The common user created during group creation (T003) follows the same pattern as this task's create endpoint, but is embedded in the group creation transaction. This task handles standalone creation only.
- Password for newly created common users is auto-generated (same policy as master user — using `generateRandomPassword` from `@satie/utils`). The password is NOT returned in the response.
- Follow the Identity domain's `UsersService` for bcrypt hash patterns and the `UserResponseDto` pattern for excluding `senha` from responses.
