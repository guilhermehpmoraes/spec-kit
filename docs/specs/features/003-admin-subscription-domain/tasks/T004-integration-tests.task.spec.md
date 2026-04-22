# Task Spec: Integration Tests

- **Feature ID**: 003-admin-subscription-domain
- **Task ID**: T004
- **Status**: Done
- **Type**: Task
- **Parallelizable**: No
- **Parallelization Notes**: Depends on all previous tasks (T001–T003) since it tests the full API surface.
- **Date**: 2026-04-17
- **Last Updated**: 2026-04-17
- **Owner**: Guilherme Moraes
- **Feature Folder**: docs/specs/features/003-admin-subscription-domain/
- **Feature Spec**: docs/specs/features/003-admin-subscription-domain/feature.spec.md
- **Feature Plan**: docs/specs/features/003-admin-subscription-domain/plan.spec.md
- **Task File**: docs/specs/features/003-admin-subscription-domain/tasks/T004-integration-tests.task.spec.md
- **Related ADRs**: ADR-005, ADR-007, ADR-010
- **Dependencies**: T001, T002, T003

## 1. Purpose

Create end-to-end integration tests for all subscription domain endpoints (products, permissions, plans), covering all acceptance scenarios from the feature spec. Tests run against a live backend with real PostgreSQL, seeding test data directly via database client since products/permissions have no create API.

## 2. Scope

### In Scope

- Integration test file for products/permissions endpoints (`produtos.spec.ts`)
- Integration test file for plans endpoints (`planos.spec.ts`)
- Test data seeding via direct database insertion (products + permissions for test setup)
- Test data cleanup in `afterAll`
- All acceptance scenarios from the feature spec (AC S1.1–S5.1)
- JWT authentication validation (401 without token)

### Out of Scope

- Unit tests (covered in T002, T003)
- Frontend tests
- Performance/load testing
- Seed migration for production data

## 3. Context from Feature Plan

- **Plan slice**: Slice 4 — Integration Tests
- **Requirement refs**: All FRs (FR-001 through FR-015), SC-007, SC-008, all ACs (S1.1–S5.1)
- **Affected Paths**:
  - `apps/admin/backend-tests/src/backend/produtos.spec.ts` (create)
  - `apps/admin/backend-tests/src/backend/planos.spec.ts` (create)
- **Why this task exists**: Integration tests validate the complete API surface against the real database and authentication, ensuring all acceptance criteria are met end-to-end.

## 4. Technical Specification (Required)

### 4.1 Backend and API Contracts (when applicable)

No new API code — this task tests existing endpoints from T002 and T003. All request/response contracts are defined in those tasks.

**Test data seeding approach**: Direct database insertion via `pg.Client` in `beforeAll` hooks (same pattern as existing `usuarios-admin.spec.ts`). Products and permissions are catalog data with no create API, so test data must be inserted directly.

**Test data schema**:

- **Test Product 1** ("Produto Teste A"):
  - `id_produto`: fixed UUID (e.g., generated once and hardcoded)
  - `codigo_produto`: `"PROD_TEST_A"`
  - `descricao`: `"Produto de Teste A"`
  - With 3 permissions: `PERM_A1`, `PERM_A2`, `PERM_A3`

- **Test Product 2** ("Produto Teste B"):
  - `id_produto`: fixed UUID
  - `codigo_produto`: `"PROD_TEST_B"`
  - `descricao`: `"Produto de Teste B"`
  - With 2 permissions: `PERM_B1`, `PERM_B2`

This dual-product setup enables cross-product permission validation testing.

**Authentication**: Reuse the existing `login()` helper pattern from `usuarios-admin.spec.ts` — login with the seed admin credentials to get a JWT access token.

### 4.2 Database Specification (mandatory when data impact exists)

N/A — No schema changes. Test data is inserted and cleaned up within test lifecycle.

### 4.3 Frontend and UX Contracts (when applicable)

N/A — Backend integration tests only.

### 4.4 Cross-Cutting Constraints

- **Security/authorization**: Tests verify 401 responses for unauthenticated requests on all endpoints.
- **Performance expectations**: N/A for integration tests.
- **Observability**: N/A.
- **Compatibility constraints**: Tests must run against the existing Docker Compose setup (PostgreSQL + Redis + backend).

## 5. Implementation Steps

1. **Create `produtos.spec.ts`** at `apps/admin/backend-tests/src/backend/produtos.spec.ts`:
   - Import `axios`, `pg.Client`, and helper functions.
   - `beforeAll`: Create a `pg.Client`, insert test products and permissions via raw SQL.
   - `afterAll`: Delete test data (hard delete since it's test data), close client.
   - Implement all test scenarios listed in Section 7 (Products/Permissions tests).

2. **Create `planos.spec.ts`** at `apps/admin/backend-tests/src/backend/planos.spec.ts`:
   - Import `axios`, `pg.Client`, and helper functions.
   - `beforeAll`: Create a `pg.Client`, insert test products and permissions (same as `produtos.spec.ts` or shared), login to get JWT token.
   - `afterAll`: Clean up all test plans, test data, close client.
   - Implement all test scenarios listed in Section 7 (Plans tests).

3. **Run all integration tests**: `pnpm nx run admin-backend-tests:test` to verify all pass.

4. **Run Biome check**: `pnpm nx run-many -t check`.

## 6. Acceptance Criteria

- AC1: Products listing returns all seeded products (maps to AC S1.1).
- AC2: Product detail returns product with nested permissions (maps to AC S1.2).
- AC3: Product permissions endpoint returns permissions for the given product (maps to AC S1.2).
- AC4: Plan creation with valid data succeeds and returns plan with product + permissions (maps to AC S2.1).
- AC5: Plan creation with invalid color is rejected with 400 (maps to AC S2.2).
- AC6: Plan creation with cross-product permissions is rejected with 400 (maps to AC S2.3).
- AC7: Plan creation with non-existent product is rejected with 404 (maps to AC S2.4).
- AC8: Plan update with new permissions replaces old ones (maps to AC S3.1).
- AC9: Plan deactivation preserves permissions (maps to AC S3.2).
- AC10: Plan update with cross-product permissions is rejected with 400 (maps to AC S3.3).
- AC11: Plan detail returns plan with nested product and permissions (maps to AC S4.1).
- AC12: Plan soft-delete removes plan from listing (maps to AC S5.1).
- AC13: All endpoints return 401 without JWT token.
- AC14: Biome reports zero warnings/errors on test files.

## 7. Test Scenarios

### Products and Permissions Tests (`produtos.spec.ts`)

- **S1.1 — List products**: `GET /api/produtos` returns 200 with array containing seeded test products. Each product has `idProduto`, `codigoProduto`, `descricao`, `criadoAs`, `modificadoAs`.
- **S1.2 — Get product by ID with permissions**: `GET /api/produtos/:id` returns 200 with product data and nested `permissoes` array. Verify permission count matches seeded data.
- **Product not found**: `GET /api/produtos/:nonExistentId` returns 404 with `"Produto não encontrado"`.
- **List permissions for product**: `GET /api/produtos/:idProduto/permissoes` returns 200 with array of permissions for the given product. Each permission has `idPermissao`, `codigoPermissao`, `descricao`, `criadoAs`, `modificadoAs`.
- **Permissions for non-existent product**: `GET /api/produtos/:nonExistentId/permissoes` returns 404 with `"Produto não encontrado"`.
- **Unauthorized — list products**: `GET /api/produtos` without JWT returns 401.
- **Unauthorized — get product**: `GET /api/produtos/:id` without JWT returns 401.
- **Unauthorized — list permissions**: `GET /api/produtos/:id/permissoes` without JWT returns 401.

### Plans Tests (`planos.spec.ts`)

- **S2.1 — Create plan with product and permissions**: `POST /api/planos` with valid `nome`, `descricao`, `cor`, `ativo`, `idProduto`, `permissoes` (UUIDs of permissions belonging to the product) returns 201 with `PlanDetailResponseDto` including nested `produto` and `permissoes`.
- **S2.2 — Invalid color format**: `POST /api/planos` with `cor: "invalid"` returns 400.
- **S2.2b — Invalid color format (missing #)**: `POST /api/planos` with `cor: "FF5733"` (missing #) returns 400.
- **S2.3 — Cross-product permission violation**: `POST /api/planos` with `idProduto` of Product A and `permissoes` containing a permission from Product B returns 400 with `"As permissões informadas não pertencem ao produto do plano"`.
- **S2.4 — Non-existent product**: `POST /api/planos` with a non-existent `idProduto` UUID returns 404 with `"Produto não encontrado"`.
- **List plans**: `GET /api/planos` returns 200 with array of plans, each containing `permissoes` but no nested `produto`.
- **S4.1 — Get plan detail**: `GET /api/planos/:id` returns 200 with plan including nested `produto` (ProductResponseDto) and `permissoes` (PermissionResponseDto[]).
- **Plan not found**: `GET /api/planos/:nonExistentId` returns 404 with `"Plano não encontrado"`.
- **S3.1 — Update plan with permission replacement**: `PATCH /api/planos/:id` with new `permissoes` returns 200 with updated permissions. Old permissions are no longer in the response.
- **S3.1b — Update plan scalar fields**: `PATCH /api/planos/:id` with only `nome` and `cor` (no `permissoes`) returns 200 with updated fields and unchanged permissions.
- **S3.2 — Deactivate plan preserves permissions**: `PATCH /api/planos/:id` with `ativo: false` returns 200 with `ativo: false` and permissions intact.
- **S3.3 — Cross-product permission violation on update**: `PATCH /api/planos/:id` with `permissoes` containing a permission from a different product returns 400.
- **S5.1 — Soft-delete plan**: `DELETE /api/planos/:id` returns 204. Subsequent `GET /api/planos/:id` returns 404. Subsequent `GET /api/planos` does not include the deleted plan.
- **Delete non-existent plan**: `DELETE /api/planos/:nonExistentId` returns 404.
- **Unauthorized — create plan**: `POST /api/planos` without JWT returns 401.
- **Unauthorized — list plans**: `GET /api/planos` without JWT returns 401.
- **Unauthorized — get plan**: `GET /api/planos/:id` without JWT returns 401.
- **Unauthorized — update plan**: `PATCH /api/planos/:id` without JWT returns 401.
- **Unauthorized — delete plan**: `DELETE /api/planos/:id` without JWT returns 401.

## 8. Definition of Ready (to start Step 4)

- [x] Status is `Ready`.
- [x] Scope is explicit and bounded.
- [x] Required contracts are defined (API, DTO, event, UI state, schema).
- [x] Technical specification is detailed enough for independent implementation.
- [x] For data-impact tasks, table/field/type/constraint/index/migration details are fully documented.
- [x] Acceptance criteria are testable.
- [x] Open questions are resolved or captured as explicit assumptions.
- [x] All dependency tasks are `Done` (T001, T002, and T003 must be completed first).

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
| 1 | `NX_INTERACTIVE=false pnpm nx e2e admin-backend-tests --testPathPattern='produtos\|planos'` | 9 suites passed, 97 tests passed, 0 failures | 2026-04-17 16:24 |
| 2 | `npx biome check apps/admin/backend-tests/src/backend/produtos.spec.ts apps/admin/backend-tests/src/backend/planos.spec.ts` | Checked 2 files, no fixes applied | 2026-04-17 16:24 |

**Rules**:
- Every test suite relevant to the task must have a row in this table.
- "Result" must include pass/fail/skip counts copied from actual terminal output.
- If any test fails, the task stays `In Progress` — do not fill this section with failing results and mark Done.
- This section is never pre-filled during Step 3 (Task Breakdown) — it is populated only during Step 4 (Implementation).

## 11. Jira Mapping (Optional)

- **Issue Type**: Task
- **Summary**: Create integration tests for subscription domain
- **Description**: End-to-end API tests for products, permissions, and plans endpoints covering all acceptance scenarios with test data seeding.
- **Priority**: High
- **Labels**: Backend, QA
- **Custom field (Tipo)**: Feature
- **Custom field (Tamanho)**: 1 Dia

## 12. Status Log

| Date       | Status      | Notes |
| ---------- | ----------- | ----- |
| 2026-04-17 | Draft       | Task created from approved feature plan |
| 2026-04-17 | Awaiting Dependency | Task approved; waiting for T001, T002, and T003 to complete |
| 2026-04-17 | Ready | All dependencies (T001, T002, T003) are Done |
| 2026-04-17 | In Progress | Implementation started — created produtos.spec.ts and planos.spec.ts |
| 2026-04-17 | Done | All 97 tests pass (9 suites), Biome clean, all ACs validated |

## 13. Observations

- Test data seeding uses direct SQL via `pg.Client` (same pattern as `usuarios-admin.spec.ts`) rather than repository/API calls — products and permissions have no create API.
- Fixed UUIDs for test products/permissions allow deterministic assertions without querying for IDs.
- Integration tests depend on a running backend, PostgreSQL, and Redis — same as existing tests via `jest.config.cts` with global setup/teardown.
- Plans created during tests should be cleaned up in `afterAll` to avoid polluting the database for other test runs.
- The test data cleanup order must respect FK constraints: delete `plano_permissao` → `plano` → `permissao` → `produto` (or use hard deletes with CASCADE).
