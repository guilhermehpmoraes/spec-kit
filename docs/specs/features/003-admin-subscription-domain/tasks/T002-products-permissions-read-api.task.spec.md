# Task Spec: Products and Permissions Read API

- **Feature ID**: 003-admin-subscription-domain
- **Task ID**: T002
- **Status**: Done
- **Type**: Task
- **Parallelizable**: No
- **Parallelization Notes**: Depends on T001 for entities. T003 depends on this task for ProductsModule exports.
- **Date**: 2026-04-17
- **Owner**: Guilherme Moraes
- **Feature Folder**: docs/specs/features/003-admin-subscription-domain/
- **Feature Spec**: docs/specs/features/003-admin-subscription-domain/feature.spec.md
- **Feature Plan**: docs/specs/features/003-admin-subscription-domain/plan.spec.md
- **Task File**: docs/specs/features/003-admin-subscription-domain/tasks/T002-products-permissions-read-api.task.spec.md
- **Related ADRs**: ADR-004, ADR-005, ADR-010
- **Dependencies**: T001

## 1. Purpose

Implement the read-only Products and Permissions API (3 GET endpoints), the `SubscriptionModule` domain boundary, the `ProductsModule`, and update the admin architecture documentation. This gives operators visibility into the product/permission catalog needed for plan creation. Also includes unit tests for the products service.

## 2. Scope

### In Scope

- `SubscriptionModule` — domain boundary module importing `ProductsModule`
- `ProductsModule` — module registering `Produto` and `Permissao` entities, controller, and service
- `ProductsController` — 3 read-only endpoints with JWT auth and Swagger decorators
- `ProductsService` — read-only operations (`findAll`, `findOne`, `findPermissionsByProduct`)
- `ProductResponseDto`, `ProductDetailResponseDto`, `PermissionResponseDto` — response DTOs
- Import `SubscriptionModule` in `AppModule`
- Unit tests for `ProductsService`
- Update `docs/specs/apps/admin/architecture.md` with Subscription domain entry

### Out of Scope

- Plans CRUD (T003)
- Integration tests (T004)
- Seed data for products/permissions (future task)

## 3. Context from Feature Plan

- **Plan slice**: Slice 2 — Products and Permissions Read-Only API
- **Requirement refs**: FR-001, FR-002, FR-010, SC-002, AC S1.1, AC S1.2
- **Affected Paths**:
  - `apps/admin/backend/src/subscription/subscription.module.ts` (create)
  - `apps/admin/backend/src/subscription/products/products.module.ts` (create)
  - `apps/admin/backend/src/subscription/products/products.controller.ts` (create)
  - `apps/admin/backend/src/subscription/products/products.service.ts` (create)
  - `apps/admin/backend/src/subscription/products/dto/product-response.dto.ts` (create)
  - `apps/admin/backend/src/subscription/products/dto/product-detail-response.dto.ts` (create)
  - `apps/admin/backend/src/subscription/products/dto/permission-response.dto.ts` (create)
  - `apps/admin/backend/src/app/app.module.ts` (modify — add SubscriptionModule import)
  - `apps/admin/backend-tests/src/unit/products.service.spec.ts` (create)
  - `docs/specs/apps/admin/architecture.md` (modify — add Subscription domain)
- **Why this task exists**: Read-only product/permission endpoints are a prerequisite for plan CRUD (T003), since plan creation requires selecting a product and its permissions.

## 4. Technical Specification (Required)

### 4.1 Backend and API Contracts (when applicable)

#### GET /api/produtos — List all products

- **Use case flow**: request → `ProductsController.findAll()` → `ProductsService.findAll()` → `Produto` repository → response
- **Endpoint**: `GET /api/produtos`
- **Request contract**: No body or query params.
- **Response contract** (`200 OK`):

| Field | Type | Nullable | Source | Notes |
| ----- | ---- | -------- | ------ | ----- |
| `idProduto` | string (UUID) | No | `produto.id_produto` | Primary key |
| `codigoProduto` | string | No | `produto.codigo_produto` | Unique product code |
| `descricao` | string | No | `produto.descricao` | Product description |
| `criadoAs` | Date | No | `produto.criado_as` | Creation timestamp |
| `modificadoAs` | Date | No | `produto.modificado_as` | Last update timestamp |

- **Error contract**:

| Status | Condition | Error Message |
|--------|-----------|---------------|
| `401` | Missing/invalid JWT | Standard JwtAuthGuard response |

#### GET /api/produtos/:id — Get product by ID with permissions

- **Use case flow**: request → `ProductsController.findOne(id)` → `ProductsService.findOne(id)` → `Produto` repository (with `permissoes` relation) → response
- **Endpoint**: `GET /api/produtos/:id`
- **Request contract**: `id` path param (UUID, validated by `ParseUUIDPipe`).
- **Response contract** (`200 OK` — `ProductDetailResponseDto`):

| Field | Type | Nullable | Source | Notes |
| ----- | ---- | -------- | ------ | ----- |
| `idProduto` | string (UUID) | No | `produto.id_produto` | Primary key |
| `codigoProduto` | string | No | `produto.codigo_produto` | Unique product code |
| `descricao` | string | No | `produto.descricao` | Product description |
| `criadoAs` | Date | No | `produto.criado_as` | Creation timestamp |
| `modificadoAs` | Date | No | `produto.modificado_as` | Last update timestamp |
| `permissoes` | `PermissionResponseDto[]` | No | Related permissions | Nested permissions |

- **Error contract**:

| Status | Condition | Error Message |
|--------|-----------|---------------|
| `401` | Missing/invalid JWT | Standard JwtAuthGuard response |
| `404` | Product not found | "Produto não encontrado" |

#### GET /api/produtos/:idProduto/permissoes — List permissions for a product

- **Use case flow**: request → `ProductsController.findPermissionsByProduct(idProduto)` → `ProductsService.findPermissionsByProduct(idProduto)` → verify product exists → `Permissao` repository (where `idProduto`) → response
- **Endpoint**: `GET /api/produtos/:idProduto/permissoes`
- **Request contract**: `idProduto` path param (UUID, validated by `ParseUUIDPipe`).
- **Response contract** (`200 OK`):

| Field | Type | Nullable | Source | Notes |
| ----- | ---- | -------- | ------ | ----- |
| `idPermissao` | string (UUID) | No | `permissao.id_permissao` | Primary key |
| `codigoPermissao` | string | No | `permissao.codigo_permissao` | Unique permission code |
| `descricao` | string | No | `permissao.descricao` | Permission description |
| `criadoAs` | Date | No | `permissao.criado_as` | Creation timestamp |
| `modificadoAs` | Date | No | `permissao.modificado_as` | Last update timestamp |

- **Error contract**:

| Status | Condition | Error Message |
|--------|-----------|---------------|
| `401` | Missing/invalid JWT | Standard JwtAuthGuard response |
| `404` | Product not found | "Produto não encontrado" |

#### PermissionResponseDto

| Field | Type | Nullable | Source | Notes |
| ----- | ---- | -------- | ------ | ----- |
| `idPermissao` | string (UUID) | No | `permissao.id_permissao` | Primary key |
| `codigoPermissao` | string | No | `permissao.codigo_permissao` | Unique permission code |
| `descricao` | string | No | `permissao.descricao` | Permission description |
| `criadoAs` | Date | No | `permissao.criado_as` | Creation timestamp |
| `modificadoAs` | Date | No | `permissao.modificado_as` | Last update timestamp |

### 4.2 Database Specification (mandatory when data impact exists)

N/A — No database changes in this task. Entities created in T001.

### 4.3 Frontend and UX Contracts (when applicable)

N/A — Backend-only task.

### 4.4 Cross-Cutting Constraints

- **Security/authorization**: All 3 endpoints require JWT authentication via `@UseGuards(JwtAuthGuard)` and `@ApiBearerAuth()` at class level. The guard is reused from the Identity domain — no import of `AuthModule` is needed (Passport global strategy registration).
- **Performance expectations**: Product/permission catalogs are small — no pagination needed.
- **Observability**: Standard NestJS logging.
- **Compatibility constraints**: Purely additive — new endpoints, no existing routes affected.

## 5. Implementation Steps

1. **Create `PermissionResponseDto`** at `apps/admin/backend/src/subscription/products/dto/permission-response.dto.ts`:
   - Fields: `idPermissao`, `codigoPermissao`, `descricao`, `criadoAs`, `modificadoAs`.
   - All fields have `@ApiProperty` with `description` and `example` (ADR-010).
   - Static `fromEntity(entity: Permissao): PermissionResponseDto` mapper method.

2. **Create `ProductResponseDto`** at `apps/admin/backend/src/subscription/products/dto/product-response.dto.ts`:
   - Fields: `idProduto`, `codigoProduto`, `descricao`, `criadoAs`, `modificadoAs`.
   - All fields have `@ApiProperty` with `description` and `example`.
   - Static `fromEntity(entity: Produto): ProductResponseDto` mapper method.

3. **Create `ProductDetailResponseDto`** at `apps/admin/backend/src/subscription/products/dto/product-detail-response.dto.ts`:
   - Fields: same as `ProductResponseDto` + `permissoes: PermissionResponseDto[]`.
   - All fields have `@ApiProperty`.
   - Static `fromEntity(entity: Produto): ProductDetailResponseDto` mapper method (maps `entity.permissoes` using `PermissionResponseDto.fromEntity`).

4. **Create `ProductsService`** at `apps/admin/backend/src/subscription/products/products.service.ts`:
   - `@Injectable()`.
   - Inject `@InjectRepository(Produto) produtoRepository: Repository<Produto>` and `@InjectRepository(Permissao) permissaoRepository: Repository<Permissao>`.
   - `findAll(): Promise<ProductResponseDto[]>` — query all products, map to `ProductResponseDto`.
   - `findOne(id: string): Promise<ProductDetailResponseDto>` — query product by `idProduto` with `relations: ["permissoes"]`, throw `NotFoundException("Produto não encontrado")` if not found, map to `ProductDetailResponseDto`.
   - `findPermissionsByProduct(idProduto: string): Promise<PermissionResponseDto[]>` — first verify product exists (throw `NotFoundException("Produto não encontrado")` if not), then query permissions where `idProduto`, map to `PermissionResponseDto`.

5. **Create `ProductsController`** at `apps/admin/backend/src/subscription/products/products.controller.ts`:
   - `@ApiTags("Produtos")`, `@ApiBearerAuth()`, `@UseGuards(JwtAuthGuard)`, `@Controller("produtos")`.
   - `@Get()` → `findAll()` — returns `ProductResponseDto[]`. Swagger: `@ApiOperation`, `@ApiResponse({ status: 200, type: [ProductResponseDto] })`, `@ApiResponse({ status: 401 })`.
   - `@Get(":id")` → `findOne(@Param("id", ParseUUIDPipe) id)` — returns `ProductDetailResponseDto`. Swagger: `@ApiResponse({ status: 200, type: ProductDetailResponseDto })`, `@ApiResponse({ status: 401 })`, `@ApiResponse({ status: 404 })`.
   - `@Get(":idProduto/permissoes")` → `findPermissionsByProduct(@Param("idProduto", ParseUUIDPipe) idProduto)` — returns `PermissionResponseDto[]`. Swagger: `@ApiResponse({ status: 200, type: [PermissionResponseDto] })`, `@ApiResponse({ status: 401 })`, `@ApiResponse({ status: 404 })`.

6. **Create `ProductsModule`** at `apps/admin/backend/src/subscription/products/products.module.ts`:
   - `imports: [TypeOrmModule.forFeature([Produto, Permissao])]`.
   - `controllers: [ProductsController]`.
   - `providers: [ProductsService]`.
   - `exports: [ProductsService]`.

7. **Create `SubscriptionModule`** at `apps/admin/backend/src/subscription/subscription.module.ts`:
   - `imports: [ProductsModule]`.
   - `exports: [ProductsModule]`.

8. **Update `AppModule`** at `apps/admin/backend/src/app/app.module.ts`:
   - Import `SubscriptionModule`.
   - Add to `imports` array.

9. **Create unit tests** at `apps/admin/backend-tests/src/unit/products.service.spec.ts`:
   - Mock `Repository<Produto>` and `Repository<Permissao>`.
   - Test `findAll()`: returns mapped DTOs.
   - Test `findOne()`: returns product with permissions when found.
   - Test `findOne()`: throws `NotFoundException` when product not found.
   - Test `findPermissionsByProduct()`: returns permissions for existing product.
   - Test `findPermissionsByProduct()`: throws `NotFoundException` when product not found.

10. **Update admin architecture doc** at `docs/specs/apps/admin/architecture.md`:
    - Add **Subscription** domain entry under "## Domains" section.
    - Add module hierarchy entries for `subscription/` under "### Module Hierarchy".

11. **Run Biome check**: `pnpm nx run-many -t check` to verify compliance.

## 6. Acceptance Criteria

- AC1: `GET /api/produtos` returns all active products with correct DTO fields (maps to AC S1.1).
- AC2: `GET /api/produtos/:id` returns the product with nested permissions (maps to AC S1.2).
- AC3: `GET /api/produtos/:id` returns 404 when product does not exist.
- AC4: `GET /api/produtos/:idProduto/permissoes` returns all permissions for the given product.
- AC5: `GET /api/produtos/:idProduto/permissoes` returns 404 when product does not exist.
- AC6: All 3 endpoints return 401 when no JWT token is provided.
- AC7: All endpoints are documented in Swagger/Scalar with correct response types and descriptions.
- AC8: Unit tests for `ProductsService` cover all service methods and error paths.
- AC9: Biome reports zero warnings/errors on all changed files.

## 7. Test Scenarios

### Unit Tests (ProductsService)

- **Happy path — findAll()**: Given products exist in the repository, when `findAll()` is called, then all products are returned as `ProductResponseDto[]`.
- **Happy path — findOne()**: Given a product with permissions exists, when `findOne(id)` is called, then the product is returned as `ProductDetailResponseDto` with nested `PermissionResponseDto[]`.
- **Error path — findOne() not found**: Given no product with the given ID exists, when `findOne(id)` is called, then `NotFoundException("Produto não encontrado")` is thrown.
- **Happy path — findPermissionsByProduct()**: Given a product exists with permissions, when `findPermissionsByProduct(idProduto)` is called, then permissions are returned as `PermissionResponseDto[]`.
- **Error path — findPermissionsByProduct() product not found**: Given no product with the given ID exists, when `findPermissionsByProduct(idProduto)` is called, then `NotFoundException("Produto não encontrado")` is thrown.
- **Edge case — findAll() empty**: Given no products exist, when `findAll()` is called, then an empty array is returned.
- **Edge case — findPermissionsByProduct() no permissions**: Given a product exists but has no permissions, when `findPermissionsByProduct(idProduto)` is called, then an empty array is returned.

## 8. Definition of Ready (to start Step 4)

- [x] Status is `Ready`.
- [x] Scope is explicit and bounded.
- [x] Required contracts are defined (API, DTO, event, UI state, schema).
- [x] Technical specification is detailed enough for independent implementation.
- [x] For data-impact tasks, table/field/type/constraint/index/migration details are fully documented.
- [x] Acceptance criteria are testable.
- [x] Open questions are resolved or captured as explicit assumptions.
- [x] All dependency tasks are `Done` (T001 must be completed first).

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
| 1 | `npx jest --config jest.unit.config.js --testPathPatterns="products.service"` | 7 passed, 0 failed, 0 skipped | 2026-04-17 |
| 2 | `npx biome check apps/admin/backend/src/subscription/ apps/admin/backend-tests/src/unit/products.service.spec.ts apps/admin/backend/src/app/app.module.ts` | Checked 13 files. No fixes applied. | 2026-04-17 |

**Rules**:
- Every test suite relevant to the task must have a row in this table.
- "Result" must include pass/fail/skip counts copied from actual terminal output.
- If any test fails, the task stays `In Progress` — do not fill this section with failing results and mark Done.
- This section is never pre-filled during Step 3 (Task Breakdown) — it is populated only during Step 4 (Implementation).

## 11. Jira Mapping (Optional)

- **Issue Type**: Task
- **Summary**: Implement products and permissions read-only API
- **Description**: Create SubscriptionModule, ProductsModule with read-only endpoints for products and permissions, response DTOs, unit tests, and architecture doc update.
- **Priority**: Highest
- **Labels**: Backend
- **Custom field (Tipo)**: Feature
- **Custom field (Tamanho)**: 1 Dia

## 12. Status Log

| Date       | Status      | Notes |
| ---------- | ----------- | ----- |
| 2026-04-17 | Draft       | Task created from approved feature plan |
| 2026-04-17 | Awaiting Dependency | Task approved; waiting for T001 to complete |
| 2026-04-17 | Ready | T001 completed; all dependencies met |
| 2026-04-17 | In Progress | Implementation started |
| 2026-04-17 | Done | All code implemented, 7/7 unit tests pass, Biome clean |

## 13. Observations

- The `ProductsModule` exports `ProductsService` so that `PlansModule` (T003) can inject it for product existence validation during plan creation.
- The controller uses `@Controller("produtos")` — the `/api` prefix is applied globally via NestJS `setGlobalPrefix` in `main.ts`.
- `@UseGuards(JwtAuthGuard)` at class level applies to all 3 endpoints. No `AuthModule` import needed — Passport registers the JWT strategy globally.
- The `GET /api/produtos/:idProduto/permissoes` route must be defined AFTER `GET /api/produtos/:id` in the controller to avoid route conflict (NestJS evaluates routes in order). Alternatively, use a distinct path segment to avoid ambiguity.
