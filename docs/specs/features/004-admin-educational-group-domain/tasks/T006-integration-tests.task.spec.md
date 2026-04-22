# Task Spec: Integration Tests

- **Feature ID**: 004-admin-educational-group-domain
- **Task ID**: T006
- **Status**: Done
- **Type**: Task
- **Parallelizable**: No
- **Parallelization Notes**: Depends on all CRUD tasks (T003, T004, T005) being done. Integration tests exercise the full stack and require all endpoints to exist.
- **Date**: 2026-04-20
- **Owner**: Guilherme Moraes
- **Feature Folder**: docs/specs/features/004-admin-educational-group-domain/
- **Feature Spec**: docs/specs/features/004-admin-educational-group-domain/feature.spec.md
- **Feature Plan**: docs/specs/features/004-admin-educational-group-domain/plan.spec.md
- **Task File**: docs/specs/features/004-admin-educational-group-domain/tasks/T006-integration-tests.task.spec.md
- **Related ADRs**: ADR-004, ADR-005, ADR-007
- **Dependencies**: T003, T004, T005

## 1. Purpose

Write integration tests (Supertest) for all 12 endpoints across the 3 endpoint groups (groups, subscriptions, common users). These tests exercise the full NestJS stack — controllers, services, DTOs, entities, and database — to verify that the educational group domain works end-to-end. This task also validates all acceptance scenarios from the feature spec.

## 2. Scope

### In Scope

- Integration test file for groups endpoints: `apps/admin/backend-tests/src/backend/educational-group/groups.spec.ts`
- Integration test file for subscriptions endpoints: `apps/admin/backend-tests/src/backend/educational-group/subscriptions.spec.ts`
- Integration test file for common users endpoints: `apps/admin/backend-tests/src/backend/educational-group/common-users.spec.ts`
- Test scenarios covering all 6 acceptance scenarios from the feature spec
- Auth rejection tests (unauthenticated requests)
- Error path tests (404, 400, 409)
- Edge case tests (slug collision, empty arrays, cascade soft-delete verification)

### Out of Scope

- Unit tests (covered in T003, T004, T005)
- E2E/Playwright tests (N/A — backend-only)
- Performance/load testing
- Test infrastructure changes (use existing `backend-tests` setup)

## 3. Context from Feature Plan

- **Plan slice**: Slice 5 — Tests
- **Requirement refs**: All FRs (FR-001 through FR-011)
- **Acceptance covered**: All scenarios (S1–S6)
- **Affected Paths**:
  - `apps/admin/backend-tests/src/backend/educational-group/groups.spec.ts` (NEW)
  - `apps/admin/backend-tests/src/backend/educational-group/subscriptions.spec.ts` (NEW)
  - `apps/admin/backend-tests/src/backend/educational-group/common-users.spec.ts` (NEW)
- **Why this task exists**: Integration tests validate the full stack behavior and all acceptance criteria from the feature spec. Unit tests (in T003–T005) validate service logic in isolation; these tests validate the HTTP layer, validation pipes, guard behavior, database persistence, and transaction integrity.

## 4. Technical Specification (Required)

### 4.1 Backend and API Contracts (when applicable)

N/A — This task tests existing endpoints, not creating new ones. All endpoint contracts are defined in T003, T004, and T005.

### 4.2 Database Specification (mandatory when data impact exists)

N/A — No schema changes. Tests use the existing database and migration from T002.

### 4.3 Frontend and UX Contracts (when applicable)

N/A — Backend-only feature.

### 4.4 Cross-Cutting Constraints

- **Security/authorization**: Tests must verify that all endpoints reject unauthenticated requests (401).
- **Performance expectations**: N/A — test execution time is not a concern.
- **Observability**: N/A.
- **Compatibility constraints**: Tests follow the existing `backend-tests` project patterns (ADR-007). Use `jest.config.cts` for integration tests, same setup/teardown as existing tests.

## 5. Implementation Steps

1. **Create directory** `apps/admin/backend-tests/src/backend/educational-group/`

2. **Create `groups.spec.ts`** — integration tests for `/grupos-educacionais`:
   - Setup: authenticate to get JWT token (reuse existing test auth helper pattern from `usuarios-admin.spec.ts`)
   - Seed: ensure at least 1 product with plans exists (via direct DB insert or existing seed)
   - **Test cases**:
     - POST — happy path: create group, verify response shape, verify all related entities created
     - POST — verify slug-based naming (`descricao` → `nome_banco`, `usuario`)
     - POST — verify password meets character class requirements
     - POST — duplicate product in plans → 400
     - POST — invalid plan ID → 404
     - POST — missing required fields → 400
     - POST — unauthenticated → 401
     - GET list — returns all non-deleted groups
     - GET list — unauthenticated → 401
     - GET by ID — returns full details with relations
     - GET by ID — usuario_mestre excludes `senha`
     - GET by ID — not found → 404
     - PATCH — update descricao only
     - PATCH — verify immutable fields (usuario, nome_banco) unchanged after update
     - PATCH — not found → 404
     - DELETE — soft-delete group, verify related records also soft-deleted
     - DELETE — not found → 404
     - DELETE — verify soft-deleted group doesn't appear in GET list

3. **Create `subscriptions.spec.ts`** — integration tests for `/assinaturas`:
   - Setup: authenticate, create a group with plans
   - **Test cases**:
     - GET — returns subscriptions for a group with plan details
     - GET — group not found → 404
     - GET — unauthenticated → 401
     - PATCH — replace subscriptions: verify old removed, new created
     - PATCH — duplicate product in new plans → 400
     - PATCH — invalid plan ID → 404
     - PATCH — group not found → 404
     - PATCH — unauthenticated → 401

4. **Create `common-users.spec.ts`** — integration tests for `/usuarios-comuns`:
   - Setup: authenticate, create a group
   - **Test cases**:
     - POST — create user, verify response, `verificado = false`
     - POST — group not found → 404
     - POST — unauthenticated → 401
     - GET list — returns users for a group
     - GET list — group not found → 404
     - GET by ID — returns user
     - GET by ID — not found → 404
     - PATCH — update email
     - PATCH — update password (verify it's hashed, not returned)
     - PATCH — not found → 404
     - DELETE — soft-delete user
     - DELETE — verify soft-deleted user doesn't appear in GET list
     - DELETE — not found → 404

## 6. Acceptance Criteria

- AC-S1-1: Integration test verifies atomic creation of grupo_educacional + all related entities.
- AC-S1-2: Integration test verifies rejection of duplicate-product plans.
- AC-S1-3: Integration test verifies environment-based naming (if env can be controlled in test).
- AC-S1-4: Integration test verifies password character class requirements on created user.
- AC-S2-1: Integration test verifies only `descricao` changes on PATCH.
- AC-S3-1: Integration test verifies subscription replacement behavior.
- AC-S3-2: Integration test verifies 1-plan-per-product on subscription update.
- AC-S4-1: Integration test verifies common user creation with `verificado = false`.
- AC-S4-2: Integration test verifies email/password update.
- AC-S4-3: Integration test verifies soft-delete.
- AC-S5-1: Integration test verifies GET list returns non-deleted groups.
- AC-S5-2: Integration test verifies GET by ID includes all relations.
- AC-S6-1: Integration test verifies cascade soft-delete.
- AC-ALL-AUTH: All endpoint test suites include at least one unauthenticated request test.
- AC-PASS: All integration tests pass.

## 7. Test Scenarios

All test scenarios are listed in Section 5, Implementation Steps (grouped by endpoint file). The complete list:

### groups.spec.ts (17 test cases)
1. POST — happy path (atomic creation)
2. POST — slug-based naming
3. POST — password character classes
4. POST — duplicate product → 400
5. POST — invalid plan ID → 404
6. POST — missing fields → 400
7. POST — unauthenticated → 401
8. GET list — returns groups
9. GET list — unauthenticated → 401
10. GET by ID — full details with relations
11. GET by ID — no password in response
12. GET by ID — not found → 404
13. PATCH — update descricao
14. PATCH — immutable fields unchanged
15. PATCH — not found → 404
16. DELETE — cascade soft-delete
17. DELETE — not found → 404

### subscriptions.spec.ts (8 test cases)
1. GET — subscriptions with plan details
2. GET — group not found → 404
3. GET — unauthenticated → 401
4. PATCH — replace subscriptions
5. PATCH — duplicate product → 400
6. PATCH — invalid plan → 404
7. PATCH — group not found → 404
8. PATCH — unauthenticated → 401

### common-users.spec.ts (13 test cases)
1. POST — create with `verificado = false`
2. POST — group not found → 404
3. POST — unauthenticated → 401
4. GET list — returns users
5. GET list — group not found → 404
6. GET by ID — returns user
7. GET by ID — not found → 404
8. PATCH — update email
9. PATCH — update password
10. PATCH — not found → 404
11. DELETE — soft-delete
12. DELETE — not in list after delete
13. DELETE — not found → 404

**Total: 38 integration test cases**

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
| 1 | `pnpm nx test admin-backend-tests` | 86 passed, 0 failed (unit tests) | 2026-04-20 |
| 2 | `npx jest --config apps/admin/backend-tests/jest.config.cts --verbose` | 147 passed, 0 failed (full e2e suite: 42 educational-group + 105 existing) | 2026-04-20 |
| 3 | `pnpm nx run-many -t check` | 0 warnings, 0 errors (Biome compliance) | 2026-04-20 |

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
| 2026-04-20 | Awaiting Dependency | Waiting for T003, T004, and T005 to complete |
| 2026-04-20 | Ready | All dependencies (T003, T004, T005) Done |
| 2026-04-20 | In Progress | Implementation started |
| 2026-04-20 | Done | All 42 integration tests passing, 2 production bugs fixed (CommonUsersService audit fields, SubscriptionsService hard delete) |

## 12. Observations

- Follow existing integration test patterns from `usuarios-admin.spec.ts`, `planos.spec.ts`, and `produtos.spec.ts` for test structure, auth setup, and Supertest usage.
- Test data setup (products and plans) can use direct database inserts or rely on existing seed migrations. Check what the existing tests use and follow the same pattern.
- Database cleanup between test suites: follow existing `beforeAll`/`afterAll` patterns to avoid test pollution.
- The `APP_ENVIRONMENT` variable may need to be set in the test environment to verify naming conventions. Check if the test setup already handles this.
