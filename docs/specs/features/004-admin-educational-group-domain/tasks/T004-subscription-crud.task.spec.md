# Task Spec: Subscription Independent CRUD

- **Feature ID**: 004-admin-educational-group-domain
- **Task ID**: T004
- **Status**: Done
- **Type**: Task
- **Parallelizable**: Yes
- **Parallelization Notes**: Can be developed in parallel with T005 (common user CRUD). Both depend on T003 (educational group CRUD) being done, but do not depend on each other.
- **Date**: 2026-04-20
- **Last Updated**: 2026-04-20
- **Owner**: Guilherme Moraes
- **Feature Folder**: docs/specs/features/004-admin-educational-group-domain/
- **Feature Spec**: docs/specs/features/004-admin-educational-group-domain/feature.spec.md
- **Feature Plan**: docs/specs/features/004-admin-educational-group-domain/plan.spec.md
- **Task File**: docs/specs/features/004-admin-educational-group-domain/tasks/T004-subscription-crud.task.spec.md
- **Related ADRs**: ADR-004, ADR-005, ADR-010
- **Dependencies**: T002, T003

## 1. Purpose

Implement the subscriptions sub-module for independent plan management per educational group. This allows admin users to list and replace plan subscriptions (assinaturas) for a group without modifying the group itself. The replace operation enforces the 1-plan-per-product constraint.

## 2. Scope

### In Scope

- `SubscriptionsModule` — sub-module with controller, service, and DTOs
- `SubscriptionsController` — 2 endpoints: GET list, PATCH replace
- `SubscriptionsService` — business logic: list by group, replace plans with 1-per-product validation
- DTOs: `UpdateSubscriptionsDto`, `SubscriptionResponseDto`
- JWT authentication guard on all endpoints
- Scalar API documentation decorators (ADR-010)
- Register `SubscriptionsModule` in `EducationalGroupModule`
- Unit tests for `SubscriptionsService`

### Out of Scope

- Educational group CRUD (T003)
- Common user CRUD (T005)
- Integration tests (T006)
- Creating or deleting subscriptions via group creation/deletion (handled by T003)

## 3. Context from Feature Plan

- **Plan slice**: Slice 3 — Subscription Independent CRUD
- **Requirement refs**: FR-006 (1-plan-per-product), FR-007 (separate endpoint groups), FR-008 (JWT protection)
- **Acceptance covered**: S3 (all)
- **Affected Paths**:
  - `apps/admin/backend/src/educational-group/subscriptions/subscriptions.module.ts` (NEW)
  - `apps/admin/backend/src/educational-group/subscriptions/subscriptions.controller.ts` (NEW)
  - `apps/admin/backend/src/educational-group/subscriptions/subscriptions.service.ts` (NEW)
  - `apps/admin/backend/src/educational-group/subscriptions/dto/update-subscriptions.dto.ts` (NEW)
  - `apps/admin/backend/src/educational-group/subscriptions/dto/subscription-response.dto.ts` (NEW)
  - `apps/admin/backend/src/educational-group/educational-group.module.ts` (MODIFIED — import SubscriptionsModule)
  - `apps/admin/backend-tests/src/unit/subscriptions.service.spec.ts` (NEW)
- **Why this task exists**: Subscriptions must be manageable independently from group creation/editing (feature spec Scenario 3).

## 4. Technical Specification (Required)

### 4.1 Backend and API Contracts (when applicable)

#### GET /assinaturas?idGrupoEducacional=:id — List subscriptions for a group

- **Use case flow**: Request → validate query param → find group (throw 404 if not found) → query subscriptions with plan relation → return response
- **Endpoint**: `GET /assinaturas`
- **Auth**: `@UseGuards(JwtAuthGuard)`

**Query parameters**:

| Field | Type | Required | Validation | Notes |
| ----- | ---- | -------- | ---------- | ----- |
| `idGrupoEducacional` | string | Yes | `@IsUUID("4")` | Filter subscriptions by group |

**Response contract (200)**: Array of:

| Field | Type | Notes |
| ----- | ---- | ----- |
| `idAssinatura` | string | UUID |
| `idGrupoEducacional` | string | UUID |
| `idPlano` | string | UUID |
| `plano` | object | `{ nome, idProduto }` |

**Error contract**:
- 401: Unauthorized
- 404: `"Grupo educacional não encontrado"`

#### PATCH /assinaturas/:idGrupoEducacional — Replace plan subscriptions

- **Use case flow**: Request → validate DTO → find group (throw 404) → validate plans exist (throw 404) → validate 1-per-product (throw 400) → in transaction: soft-delete existing subscriptions, create new ones → return updated list
- **Endpoint**: `PATCH /assinaturas/:idGrupoEducacional`
- **Auth**: `@UseGuards(JwtAuthGuard)`

**Path parameters**:

| Field | Type | Notes |
| ----- | ---- | ----- |
| `idGrupoEducacional` | string | UUID of the group |

**Request contract**:

| Field | Type | Required | Validation | Notes |
| ----- | ---- | -------- | ---------- | ----- |
| `planoIds` | string[] | Yes | `@IsArray()`, `@ArrayMinSize(1)`, `@IsUUID("4", { each: true })` | New plan IDs (replaces all existing) |

**Response contract (200)**: Updated array of subscriptions (same shape as GET response).

**Error contract**:
- 400: Validation error (missing/empty planoIds)
- 400: `"Não é permitido mais de um plano por produto"` — duplicate product in plans
- 401: Unauthorized
- 404: `"Grupo educacional não encontrado"` or `"Plano não encontrado"` — invalid group/plan ID

### 4.2 Database Specification (mandatory when data impact exists)

N/A — Uses entities from T002. No schema changes.

### 4.3 Frontend and UX Contracts (when applicable)

N/A — Backend-only feature.

### 4.4 Cross-Cutting Constraints

- **Security/authorization**: All endpoints protected by `JwtAuthGuard`.
- **Performance expectations**: Replace operation uses a transaction for atomicity (soft-delete old + create new).
- **Observability**: Standard NestJS logging.
- **Compatibility constraints**: The `Plano` entity from the Subscription domain is referenced read-only for validation and response enrichment.

## 5. Implementation Steps

1. **Create `SubscriptionsModule`** at `apps/admin/backend/src/educational-group/subscriptions/subscriptions.module.ts`:
   - Import `TypeOrmModule.forFeature([Assinatura, GrupoEducacional, Plano])`
   - Provide `SubscriptionsService`

2. **Create DTOs** at `apps/admin/backend/src/educational-group/subscriptions/dto/`:
   - `update-subscriptions.dto.ts` — `UpdateSubscriptionsDto` with `planoIds` field, class-validator decorators, `@ApiProperty` with description + example
   - `subscription-response.dto.ts` — `SubscriptionResponseDto` with `idAssinatura`, `idGrupoEducacional`, `idPlano`, nested `plano` object

3. **Create `SubscriptionsService`** at `apps/admin/backend/src/educational-group/subscriptions/subscriptions.service.ts`:
   - Inject repositories for `Assinatura`, `GrupoEducacional`, `Plano`, inject `DataSource`
   - `findByGroup(idGrupoEducacional: string)`:
     1. Verify group exists (throw NotFoundException if not)
     2. Query `Assinatura` with `plano` relation where `idGrupoEducacional` matches
     3. Return subscriptions array
   - `replaceForGroup(idGrupoEducacional: string, dto: UpdateSubscriptionsDto)`:
     1. Verify group exists (throw NotFoundException if not)
     2. Fetch plans by IDs — throw NotFoundException if any not found
     3. Validate 1-plan-per-product: group plans by `idProduto`, throw BadRequestException if any product has >1 plan
     4. Open transaction via `DataSource.transaction()`:
        - Soft-delete all existing `Assinatura` records for the group
        - Create new `Assinatura` records for each plan
     5. Return updated subscriptions with plan relations

4. **Create `SubscriptionsController`** at `apps/admin/backend/src/educational-group/subscriptions/subscriptions.controller.ts`:
   - `@Controller("assinaturas")`
   - `@UseGuards(JwtAuthGuard)` at class level
   - `@ApiTags("Assinaturas")`
   - `@ApiBearerAuth()`
   - 2 endpoints: GET (query param), PATCH /:idGrupoEducacional
   - `@ApiResponse` decorators with typed response DTOs

5. **Update `EducationalGroupModule`**: Import `SubscriptionsModule`

6. **Write unit tests** at `apps/admin/backend-tests/src/unit/subscriptions.service.spec.ts`:
   - Test `findByGroup()`: returns subscriptions, throws 404 for missing group
   - Test `replaceForGroup()`: happy path, plan not found, duplicate product, transaction behavior

## 6. Acceptance Criteria

- AC-S3-1: PATCH replaces existing subscriptions with new ones (old soft-deleted, new created).
- AC-S3-2: PATCH rejects requests with two plans of the same product with `"Não é permitido mais de um plano por produto"`.
- AC-GET: GET returns subscriptions for a group with plan details (`nome`, `idProduto`).
- AC-404-GROUP: Both endpoints return 404 when the group doesn't exist.
- AC-404-PLAN: PATCH returns 404 when any plan ID doesn't exist.
- AC-AUTH: All endpoints reject unauthenticated requests with 401.

## 7. Test Scenarios

### Unit Tests (SubscriptionsService)

- **findByGroup — happy path**: Returns subscriptions array with plan relations.
- **findByGroup — group not found**: Throws NotFoundException.
- **replaceForGroup — happy path**: Old subscriptions soft-deleted, new ones created, returns updated list.
- **replaceForGroup — plan not found**: Throws NotFoundException with `"Plano não encontrado"`.
- **replaceForGroup — duplicate product**: Throws BadRequestException with `"Não é permitido mais de um plano por produto"`.
- **replaceForGroup — group not found**: Throws NotFoundException with `"Grupo educacional não encontrado"`.
- **replaceForGroup — transaction**: Verify all operations happen within a transaction (mock DataSource.transaction).

## 8. Definition of Ready (to start Step 4)

- [ ] Status is `Ready`.
- [ ] Scope is explicit and bounded.
- [x] Required contracts are defined (API, DTO, event, UI state, schema).
- [x] Technical specification is detailed enough for independent implementation.
- [x] For data-impact tasks, table/field/type/constraint/index/migration details are fully documented.
- [x] Acceptance criteria are testable.
- [x] Open questions are resolved or captured as explicit assumptions.
- [ ] All dependency tasks are `Done` (if any dependencies exist).

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
| 1 | `pnpm nx test admin-backend-tests -- --testPathPatterns="subscriptions.service"` | 8 passed, 0 failed, 0 skipped | 2026-04-20 |
| 2 | `pnpm nx test admin-backend-tests` | 73 passed, 0 failed, 0 skipped (7 suites) | 2026-04-20 |
| 3 | `npx biome check apps/admin/backend/src/educational-group/subscriptions/ ...` | 7 files checked, 0 errors, 0 warnings | 2026-04-20 |

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
| 2026-04-20 | Ready | Dependencies T002 and T003 completed |
| 2026-04-20 | In Progress | Implementation started |
| 2026-04-20 | Done | Implementation complete, all tests pass, Biome clean |

## 12. Observations

- The 1-plan-per-product validation logic is shared with T003 (group creation). Consider extracting a private helper method or reusing the validation from `GroupsService` if architecturally clean. If not, duplicate the validation — it's simple enough that duplication is preferable to coupling.
- The replace operation soft-deletes existing subscriptions rather than hard-deleting them, consistent with the project's soft-delete convention (ADR-005).
