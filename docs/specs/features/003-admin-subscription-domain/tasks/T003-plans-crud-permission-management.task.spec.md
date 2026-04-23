# Task Spec: Plans CRUD with Permission Management

- **Feature ID**: 003-admin-subscription-domain
- **Task ID**: T003
- **Status**: Done
- **Type**: Task
- **Parallelizable**: No
- **Parallelization Notes**: Depends on T001 (entities) and T002 (ProductsModule exports). T004 depends on this task.
- **Date**: 2026-04-17
- **Last Updated**: 2026-04-17
- **Owner**: Guilherme Moraes
- **Feature Folder**: docs/specs/features/003-admin-subscription-domain/
- **Feature Spec**: docs/specs/features/003-admin-subscription-domain/feature.spec.md
- **Feature Plan**: docs/specs/features/003-admin-subscription-domain/plan.spec.md
- **Task File**: docs/specs/features/003-admin-subscription-domain/tasks/T003-plans-crud-permission-management.task.spec.md
- **Related ADRs**: ADR-004, ADR-005, ADR-010
- **Dependencies**: T001, T002

## 1. Purpose

Implement full CRUD for subscription plans with rich payloads (create/update include product and permissions in a single request), cross-product permission validation, atomic transaction management, cascade soft-delete, and unit tests for the plans service. This is the core business logic of the subscription domain.

## 2. Scope

### In Scope

- `PlansModule` — module registering `Plano` and `PlanoPermissao` entities, importing `ProductsModule`
- `PlansController` — 5 endpoints (POST, GET list, GET detail, PATCH, DELETE) with JWT auth and Swagger decorators
- `PlansService` — CRUD with transaction management, cross-product validation, permission replacement, cascade soft-delete
- `CreatePlanDto` — rich payload with product + permissions
- `UpdatePlanDto` — standalone DTO (NOT `PartialType(CreatePlanDto)`) excluding `idProduto`
- `PlanListResponseDto` — plan with permissions (no product nesting)
- `PlanDetailResponseDto` — plan with product + permissions
- Update `SubscriptionModule` to import `PlansModule`
- Unit tests for `PlansService`

### Out of Scope

- Integration tests (T004)
- Products/permissions endpoints (T002)
- Frontend UI (future feature)
- Seed data (future task)

## 3. Context from Feature Plan

- **Plan slice**: Slice 3 — Plans CRUD API with Permission Management
- **Requirement refs**: FR-003, FR-004, FR-005, FR-006, FR-007, FR-008, FR-010, FR-012, SC-003, SC-004, SC-005, SC-006, AC S2.1–S2.4, AC S3.1–S3.3, AC S4.1, AC S5.1
- **Affected Paths**:
  - `apps/admin/backend/src/subscription/plans/plans.module.ts` (create)
  - `apps/admin/backend/src/subscription/plans/plans.controller.ts` (create)
  - `apps/admin/backend/src/subscription/plans/plans.service.ts` (create)
  - `apps/admin/backend/src/subscription/plans/dto/create-plan.dto.ts` (create)
  - `apps/admin/backend/src/subscription/plans/dto/update-plan.dto.ts` (create)
  - `apps/admin/backend/src/subscription/plans/dto/plan-list-response.dto.ts` (create)
  - `apps/admin/backend/src/subscription/plans/dto/plan-detail-response.dto.ts` (create)
  - `apps/admin/backend/src/subscription/subscription.module.ts` (modify — add PlansModule import)
  - `apps/admin/backend-tests/src/unit/plans.service.spec.ts` (create)
- **Why this task exists**: Plan CRUD with atomic permission management is the central business capability of the subscription domain.

## 4. Technical Specification (Required)

### 4.1 Backend and API Contracts (when applicable)

#### POST /api/planos — Create a plan with product and permissions

- **Use case flow**: request → `PlansController.create(dto, req)` → `PlansService.create(dto, actorId)` → validate product exists → validate permissions belong to product → transaction { create `Plano`, create `PlanoPermissao` records } → response
- **Endpoint**: `POST /api/planos`
- **Request contract** (`CreatePlanDto`):

| Field | Type | Required | Validation | Notes |
| ----- | ---- | -------- | ---------- | ----- |
| `nome` | string | Yes | `@IsString()`, `@IsNotEmpty()`, `@MaxLength(150)` | Plan name |
| `descricao` | string | No | `@IsOptional()`, `@IsString()` | Plan description |
| `cor` | string | Yes | `@IsString()`, `@Matches(/^#[0-9A-Fa-f]{6}$/)` | 7-char hex color. Error: "A cor deve ser um código hexadecimal de 7 caracteres (ex: #FF5733)" |
| `ativo` | boolean | No | `@IsOptional()`, `@IsBoolean()` | Defaults to `true` |
| `idProduto` | string (UUID) | Yes | `@IsUUID()` | FK to product |
| `permissoes` | string[] (UUIDs) | Yes | `@IsArray()`, `@IsUUID("4", { each: true })`, `@ArrayNotEmpty()` | Permission UUIDs |

- **Response contract** (`201 Created` — `PlanDetailResponseDto`):

| Field | Type | Nullable | Source | Notes |
| ----- | ---- | -------- | ------ | ----- |
| `idPlano` | string (UUID) | No | `plano.id_plano` | Primary key |
| `nome` | string | No | `plano.nome` | Plan name |
| `descricao` | string | Yes | `plano.descricao` | Plan description |
| `cor` | string | No | `plano.cor` | Hex color |
| `ativo` | boolean | No | `plano.ativo` | Active flag |
| `criadoAs` | Date | No | `plano.criado_as` | Creation timestamp |
| `modificadoAs` | Date | No | `plano.modificado_as` | Last update timestamp |
| `produto` | `ProductResponseDto` | No | Related product | Nested product |
| `permissoes` | `PermissionResponseDto[]` | No | Related permissions via plano_permissao | Nested permissions |

- **Error contract**:

| Status | Condition | Error Message |
|--------|-----------|---------------|
| `400` | Invalid color format | "A cor deve ser um código hexadecimal de 7 caracteres (ex: #FF5733)" |
| `400` | Missing required fields | Standard class-validator messages |
| `400` | Cross-product permission violation | "As permissões informadas não pertencem ao produto do plano" |
| `401` | Missing/invalid JWT | Standard JwtAuthGuard response |
| `404` | Product not found | "Produto não encontrado" |

#### GET /api/planos — List all plans with permissions

- **Use case flow**: request → `PlansController.findAll()` → `PlansService.findAll()` → `Plano` repository (with `planoPermissoes.permissao` relations) → response
- **Endpoint**: `GET /api/planos`
- **Request contract**: No body or query params.
- **Response contract** (`200 OK` — `PlanListResponseDto[]`):

| Field | Type | Nullable | Source | Notes |
| ----- | ---- | -------- | ------ | ----- |
| `idPlano` | string (UUID) | No | `plano.id_plano` | Primary key |
| `nome` | string | No | `plano.nome` | Plan name |
| `descricao` | string | Yes | `plano.descricao` | Plan description |
| `cor` | string | No | `plano.cor` | Hex color |
| `ativo` | boolean | No | `plano.ativo` | Active flag |
| `idProduto` | string (UUID) | No | `plano.id_produto` | FK to product |
| `criadoAs` | Date | No | `plano.criado_as` | Creation timestamp |
| `modificadoAs` | Date | No | `plano.modificado_as` | Last update timestamp |
| `permissoes` | `PermissionResponseDto[]` | No | Related permissions via plano_permissao | Nested permissions |

- **Error contract**:

| Status | Condition | Error Message |
|--------|-----------|---------------|
| `401` | Missing/invalid JWT | Standard JwtAuthGuard response |

#### GET /api/planos/:id — Get plan detail with product and permissions

- **Use case flow**: request → `PlansController.findOne(id)` → `PlansService.findOne(id)` → `Plano` repository (with `produto` and `planoPermissoes.permissao` relations) → response
- **Endpoint**: `GET /api/planos/:id`
- **Request contract**: `id` path param (UUID, validated by `ParseUUIDPipe`).
- **Response contract** (`200 OK` — `PlanDetailResponseDto`):

| Field | Type | Nullable | Source | Notes |
| ----- | ---- | -------- | ------ | ----- |
| `idPlano` | string (UUID) | No | `plano.id_plano` | Primary key |
| `nome` | string | No | `plano.nome` | Plan name |
| `descricao` | string | Yes | `plano.descricao` | Plan description |
| `cor` | string | No | `plano.cor` | Hex color |
| `ativo` | boolean | No | `plano.ativo` | Active flag |
| `criadoAs` | Date | No | `plano.criado_as` | Creation timestamp |
| `modificadoAs` | Date | No | `plano.modificado_as` | Last update timestamp |
| `produto` | `ProductResponseDto` | No | Related product | Nested product |
| `permissoes` | `PermissionResponseDto[]` | No | Related permissions via plano_permissao | Nested permissions |

- **Error contract**:

| Status | Condition | Error Message |
|--------|-----------|---------------|
| `401` | Missing/invalid JWT | Standard JwtAuthGuard response |
| `404` | Plan not found | "Plano não encontrado" |

#### PATCH /api/planos/:id — Update a plan (product is immutable)

- **Use case flow**: request → `PlansController.update(id, dto, req)` → `PlansService.update(id, dto, actorId)` → find plan → if `permissoes` provided: validate belong to plan's product → transaction { soft-delete old `PlanoPermissao`, create new `PlanoPermissao`, save plan } → response
- **Endpoint**: `PATCH /api/planos/:id`
- **Request contract** (`UpdatePlanDto`):

| Field | Type | Required | Validation | Notes |
| ----- | ---- | -------- | ---------- | ----- |
| `nome` | string | No | `@IsOptional()`, `@IsString()`, `@IsNotEmpty()`, `@MaxLength(150)` | Plan name |
| `descricao` | string | No | `@IsOptional()`, `@IsString()` | Plan description |
| `cor` | string | No | `@IsOptional()`, `@Matches(/^#[0-9A-Fa-f]{6}$/)` | 7-char hex color. Error: "A cor deve ser um código hexadecimal de 7 caracteres (ex: #FF5733)" |
| `ativo` | boolean | No | `@IsOptional()`, `@IsBoolean()` | Active flag |
| `permissoes` | string[] (UUIDs) | No | `@IsOptional()`, `@IsArray()`, `@IsUUID("4", { each: true })` | When provided, replaces all current permissions. Empty array clears all permissions. |

**Note**: `UpdatePlanDto` is a standalone class, NOT `PartialType(CreatePlanDto)`, because `idProduto` must be excluded entirely (immutable after creation) and `permissoes` has different validation semantics (optional on update, can be empty array).

- **Response contract** (`200 OK` — `PlanDetailResponseDto`): Same as GET /api/planos/:id response.

- **Error contract**:

| Status | Condition | Error Message |
|--------|-----------|---------------|
| `400` | Invalid color format | "A cor deve ser um código hexadecimal de 7 caracteres (ex: #FF5733)" |
| `400` | Cross-product permission violation | "As permissões informadas não pertencem ao produto do plano" |
| `401` | Missing/invalid JWT | Standard JwtAuthGuard response |
| `404` | Plan not found | "Plano não encontrado" |

#### DELETE /api/planos/:id — Soft-delete a plan

- **Use case flow**: request → `PlansController.remove(id, req)` → `PlansService.remove(id, actorId)` → find plan → transaction { soft-delete all `PlanoPermissao` for plan, soft-delete `Plano` } → 204
- **Endpoint**: `DELETE /api/planos/:id`
- **Request contract**: `id` path param (UUID, validated by `ParseUUIDPipe`).
- **Response contract**: `204 No Content` — empty body.

- **Error contract**:

| Status | Condition | Error Message |
|--------|-----------|---------------|
| `401` | Missing/invalid JWT | Standard JwtAuthGuard response |
| `404` | Plan not found | "Plano não encontrado" |

### 4.2 Database Specification (mandatory when data impact exists)

N/A — No database schema changes. Entities created in T001.

### 4.3 Frontend and UX Contracts (when applicable)

N/A — Backend-only task.

### 4.4 Cross-Cutting Constraints

- **Security/authorization**: All 5 endpoints require JWT via `@UseGuards(JwtAuthGuard)` and `@ApiBearerAuth()` at class level.
- **Performance expectations**: Transactions are short-lived (single product lookup + permission validation + inserts). No performance concerns at current catalog sizes.
- **Observability**: Standard NestJS logging.
- **Compatibility constraints**: Purely additive — new endpoints, no existing routes affected.

### Key Implementation Patterns

#### Transaction Management

Use `DataSource.transaction()` for all multi-entity operations:

- **Create**: `transaction(async (manager) => { save Plano → save PlanoPermissao[] })`.
- **Update with permissions**: `transaction(async (manager) => { softRemove old PlanoPermissao[] → save new PlanoPermissao[] → save Plano })`.
- **Delete**: `transaction(async (manager) => { softRemove PlanoPermissao[] → softRemove Plano })`.

#### Cross-Product Permission Validation

Before creating or updating permissions:

1. Query `Permissao` where `idProduto = plan.idProduto AND idPermissao IN (requestedIds)`.
2. Compare found count vs. requested count.
3. If mismatch → throw `BadRequestException("As permissões informadas não pertencem ao produto do plano")`.

#### Permission Replacement on Update

When `permissoes` is provided in the update DTO:

1. Set `deletadoPor` on each existing `PlanoPermissao`, then `softRemove()`.
2. Create new `PlanoPermissao` records for each permission in the new set.
3. All within the same transaction.

#### Cascade Soft-Delete on Plan Deletion

1. Find all non-deleted `PlanoPermissao` records for the plan.
2. Set `deletadoPor` on each, then `softRemove()`.
3. Set `deletadoPor` on the plan, then `softRemove()`.
4. All within a single transaction.

## 5. Implementation Steps

1. **Create `CreatePlanDto`** at `apps/admin/backend/src/subscription/plans/dto/create-plan.dto.ts`:
   - Fields with validations as specified in the request contract above.
   - `@ApiProperty` on every field with `description` and `example` (ADR-010).
   - Custom error message on `@Matches` for `cor`.

2. **Create `UpdatePlanDto`** at `apps/admin/backend/src/subscription/plans/dto/update-plan.dto.ts`:
   - Standalone class (NOT `PartialType(CreatePlanDto)`).
   - All fields optional. No `idProduto` field.
   - `@ApiProperty` on every field with `description`, `example`, and `required: false`.

3. **Create `PlanListResponseDto`** at `apps/admin/backend/src/subscription/plans/dto/plan-list-response.dto.ts`:
   - Fields: `idPlano`, `nome`, `descricao`, `cor`, `ativo`, `idProduto`, `criadoAs`, `modificadoAs`, `permissoes`.
   - `@ApiProperty` on every field.
   - Static `fromEntity(entity: Plano): PlanListResponseDto` mapper — maps `planoPermissoes` to `PermissionResponseDto[]` via `planoPermissao.permissao`.

4. **Create `PlanDetailResponseDto`** at `apps/admin/backend/src/subscription/plans/dto/plan-detail-response.dto.ts`:
   - Fields: `idPlano`, `nome`, `descricao`, `cor`, `ativo`, `criadoAs`, `modificadoAs`, `produto`, `permissoes`.
   - `@ApiProperty` on every field.
   - Static `fromEntity(entity: Plano): PlanDetailResponseDto` mapper — maps `produto` to `ProductResponseDto` and `planoPermissoes` to `PermissionResponseDto[]`.

5. **Create `PlansService`** at `apps/admin/backend/src/subscription/plans/plans.service.ts`:
   - `@Injectable()`.
   - Inject `@InjectRepository(Plano)`, `@InjectRepository(PlanoPermissao)`, `@InjectRepository(Permissao)`, `@InjectRepository(Produto)`, and `DataSource`.
   - `create(dto: CreatePlanDto, actorId: string): Promise<PlanDetailResponseDto>`:
     1. Find product by `dto.idProduto` — throw `NotFoundException("Produto não encontrado")` if not found.
     2. Validate permissions belong to product (cross-product check).
     3. Transaction: create `Plano`, create `PlanoPermissao[]`.
     4. Reload plan with relations and return `PlanDetailResponseDto`.
   - `findAll(): Promise<PlanListResponseDto[]>`:
     1. Query plans with `relations: ["planoPermissoes", "planoPermissoes.permissao"]`.
     2. Map to `PlanListResponseDto[]`.
   - `findOne(id: string): Promise<PlanDetailResponseDto>`:
     1. Query plan by `idPlano` with `relations: ["produto", "planoPermissoes", "planoPermissoes.permissao"]`.
     2. Throw `NotFoundException("Plano não encontrado")` if not found.
     3. Return `PlanDetailResponseDto`.
   - `update(id: string, dto: UpdatePlanDto, actorId: string): Promise<PlanDetailResponseDto>`:
     1. Find plan by `idPlano` — throw `NotFoundException`.
     2. Update scalar fields (`nome`, `descricao`, `cor`, `ativo`) if provided.
     3. If `dto.permissoes` is provided: validate permissions belong to plan's product → transaction { soft-delete old `PlanoPermissao`, create new ones, save plan }.
     4. If `dto.permissoes` is NOT provided: just save plan scalar changes.
     5. Reload and return `PlanDetailResponseDto`.
   - `remove(id: string, actorId: string): Promise<void>`:
     1. Find plan by `idPlano` — throw `NotFoundException`.
     2. Transaction: soft-delete all `PlanoPermissao`, soft-delete `Plano` (set `deletadoPor` on both).
   - Private helper `validatePermissionsBelongToProduct(permissaoIds: string[], idProduto: string): Promise<void>` — shared by create and update.

6. **Create `PlansController`** at `apps/admin/backend/src/subscription/plans/plans.controller.ts`:
   - `@ApiTags("Planos")`, `@ApiBearerAuth()`, `@UseGuards(JwtAuthGuard)`, `@Controller("planos")`.
   - `@Post()` → `create(@Request() req, @Body() dto: CreatePlanDto)` — returns `PlanDetailResponseDto`. Status `201`.
   - `@Get()` → `findAll()` — returns `PlanListResponseDto[]`.
   - `@Get(":id")` → `findOne(@Param("id", ParseUUIDPipe) id)` — returns `PlanDetailResponseDto`.
   - `@Patch(":id")` → `update(@Request() req, @Param("id", ParseUUIDPipe) id, @Body() dto: UpdatePlanDto)` — returns `PlanDetailResponseDto`.
   - `@Delete(":id")` → `remove(@Request() req, @Param("id", ParseUUIDPipe) id)` — `@HttpCode(204)`, returns `void`.
   - All with appropriate `@ApiOperation`, `@ApiResponse` decorators per endpoint.

7. **Create `PlansModule`** at `apps/admin/backend/src/subscription/plans/plans.module.ts`:
   - `imports: [TypeOrmModule.forFeature([Plano, PlanoPermissao, Permissao, Produto]), ProductsModule]`.
   - `controllers: [PlansController]`.
   - `providers: [PlansService]`.

8. **Update `SubscriptionModule`** at `apps/admin/backend/src/subscription/subscription.module.ts`:
   - Add `PlansModule` to `imports` and `exports`.

9. **Create unit tests** at `apps/admin/backend-tests/src/unit/plans.service.spec.ts`:
   - Mock `Repository<Plano>`, `Repository<PlanoPermissao>`, `Repository<Permissao>`, `Repository<Produto>`, `DataSource`.
   - Test scenarios listed in Section 7.

10. **Run Biome check**: `pnpm nx run-many -t check`.

## 6. Acceptance Criteria

- AC1: `POST /api/planos` creates a plan with product and permissions atomically and returns `PlanDetailResponseDto` (maps to AC S2.1).
- AC2: `POST /api/planos` rejects invalid color format with 400 (maps to AC S2.2).
- AC3: `POST /api/planos` rejects cross-product permissions with 400 (maps to AC S2.3).
- AC4: `POST /api/planos` rejects non-existent product with 404 (maps to AC S2.4).
- AC5: `GET /api/planos` returns all plans with permissions (no product nesting).
- AC6: `GET /api/planos/:id` returns plan with nested product and permissions (maps to AC S4.1).
- AC7: `PATCH /api/planos/:id` updates plan and replaces permissions when provided (maps to AC S3.1).
- AC8: `PATCH /api/planos/:id` deactivates plan while preserving permissions (maps to AC S3.2).
- AC9: `PATCH /api/planos/:id` rejects cross-product permissions on update with 400 (maps to AC S3.3).
- AC10: `DELETE /api/planos/:id` soft-deletes plan and cascade soft-deletes `plano_permissao` records (maps to AC S5.1).
- AC11: All 5 endpoints return 401 without JWT.
- AC12: All endpoints documented in Swagger/Scalar.
- AC13: Unit tests cover all service methods, validation, and error paths.
- AC14: Biome reports zero warnings/errors.

## 7. Test Scenarios

### Unit Tests (PlansService)

- **Happy path — create()**: Given valid plan data and valid permissions, when `create()` is called, then plan and permission associations are created in a transaction and `PlanDetailResponseDto` is returned.
- **Error path — create() product not found**: Given a non-existent `idProduto`, when `create()` is called, then `NotFoundException("Produto não encontrado")` is thrown.
- **Error path — create() cross-product permissions**: Given permissions that don't belong to the specified product, when `create()` is called, then `BadRequestException("As permissões informadas não pertencem ao produto do plano")` is thrown.
- **Happy path — findAll()**: Given plans exist with permissions, when `findAll()` is called, then all plans are returned as `PlanListResponseDto[]` with nested permissions.
- **Happy path — findOne()**: Given a plan exists with product and permissions, when `findOne(id)` is called, then `PlanDetailResponseDto` with nested product and permissions is returned.
- **Error path — findOne() not found**: Given no plan with the given ID, when `findOne(id)` is called, then `NotFoundException("Plano não encontrado")` is thrown.
- **Happy path — update() with permissions**: Given a plan exists and valid new permissions are provided, when `update()` is called, then old permissions are soft-deleted and new ones created in a transaction.
- **Happy path — update() without permissions**: Given a plan exists and only scalar fields are updated (no `permissoes` in DTO), when `update()` is called, then only scalar fields are updated and permissions remain unchanged.
- **Error path — update() cross-product permissions**: Given permissions belonging to a different product, when `update()` is called, then `BadRequestException` is thrown.
- **Error path — update() not found**: Given no plan with the given ID, when `update()` is called, then `NotFoundException` is thrown.
- **Happy path — remove()**: Given a plan exists with permissions, when `remove()` is called, then plan and all its `PlanoPermissao` records are soft-deleted in a transaction.
- **Error path — remove() not found**: Given no plan with the given ID, when `remove()` is called, then `NotFoundException` is thrown.
- **Edge case — update() empty permissions array**: Given a plan exists and `permissoes: []` is provided, when `update()` is called, then all existing permissions are soft-deleted and no new ones are created.

## 8. Definition of Ready (to start Step 4)

- [x] Status is `Ready`.
- [x] Scope is explicit and bounded.
- [x] Required contracts are defined (API, DTO, event, UI state, schema).
- [x] Technical specification is detailed enough for independent implementation.
- [x] For data-impact tasks, table/field/type/constraint/index/migration details are fully documented.
- [x] Acceptance criteria are testable.
- [x] Open questions are resolved or captured as explicit assumptions.
- [ ] All dependency tasks are `Done` (T001 and T002 must be completed first).

## 9. Definition of Done

- [x] Status moved to `Done`.
- [x] All related tests (unit and integration) pass — no test failures allowed.
- [x] **Section 10 (Test Evidence) is filled** with the exact command(s) executed and their output summary proving all tests pass.
- [x] Biome reports zero warnings and zero errors on all changed files.
- [x] Acceptance criteria are validated by tests or clear verification evidence.
- [x] Edge cases listed in this task are covered.
- [x] Links to changed files/PR/tests are registered.
- [x] Feature spec and plan traceability remains intact.

## 10. Test Evidence

This section is **mandatory** before marking the task as `Done`. Paste the exact test command(s) executed and a summary of their output. If tests were not executed, this section must remain empty and the task **cannot** transition to `Done`.

| # | Command | Result | Timestamp |
| - | ------- | ------ | --------- |
| 1 | `npx jest --config jest.unit.config.js --testPathPatterns="plans.service"` | 15 passed, 0 failed, 0 skipped | 2026-04-17 |
| 2 | `npx jest --config jest.unit.config.js` | 48 passed, 0 failed, 0 skipped (all suites) | 2026-04-17 |
| 3 | `pnpm biome check apps/admin/backend/src/subscription/plans/ apps/admin/backend-tests/src/unit/plans.service.spec.ts apps/admin/backend/src/subscription/subscription.module.ts` | 11 files checked, 0 warnings, 0 errors | 2026-04-17 |

**Rules**:
- Every test suite relevant to the task must have a row in this table.
- "Result" must include pass/fail/skip counts copied from actual terminal output.
- If any test fails, the task stays `In Progress` — do not fill this section with failing results and mark Done.
- This section is never pre-filled during Step 3 (Task Breakdown) — it is populated only during Step 4 (Implementation).

## 11. Jira Mapping (Optional)

- **Issue Type**: Task
- **Summary**: Implement plans CRUD with permission management
- **Description**: Create PlansModule with full CRUD endpoints, cross-product validation, atomic transactions, cascade soft-delete, DTOs, and unit tests.
- **Priority**: Highest
- **Labels**: Backend
- **Custom field (Tipo)**: Feature
- **Custom field (Tamanho)**: 2 Dias

## 12. Status Log

| Date       | Status      | Notes |
| ---------- | ----------- | ----- |
| 2026-04-17 | Draft       | Task created from approved feature plan |
| 2026-04-17 | Awaiting Dependency | Task approved; waiting for T001 and T002 to complete |
| 2026-04-17 | Ready | T001 and T002 completed; all dependencies met |
| 2026-04-17 | In Progress | Implementation started by Guilherme Moraes |
| 2026-04-17 | Done | All production code, DTOs, service, controller, module, and unit tests implemented. 15/15 tests pass. Biome clean. |

## 13. Observations

- `UpdatePlanDto` is intentionally NOT `PartialType(CreatePlanDto)` — `idProduto` must be completely absent (immutable after creation), and `permissoes` has different validation semantics (optional, can be empty array to clear all permissions vs. required + non-empty on create).
- The `PlansService` injects repositories for `Permissao` and `Produto` directly rather than using `ProductsService` — this avoids a circular dependency and keeps the permission validation query efficient (single `IN` query).
- Transaction management uses `DataSource.transaction()` rather than `QueryRunner` for simplicity — this is consistent with the KISS principle and sufficient for the current use case.
- `PlanListResponseDto` maps permissions from `planoPermissoes[].permissao` — the relation path is `plano → planoPermissoes → permissao`. The DTO flattens this to a simple `PermissionResponseDto[]` by extracting `permissao` from each `planoPermissao`.
