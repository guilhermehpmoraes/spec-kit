# Task Spec: Integration, Guards, and Polish

- **Feature ID**: 001-admin-identity-domain
- **Task ID**: T006
- **Status**: Done
- **Type**: Task
- **Parallelizable**: No
- **Parallelization Notes**: Depends on T004 (CRUD endpoints) and T005 (JWT guard). Must be the last implementation task.
- **Date**: 2026-04-15
- **Owner**: Guilherme Moraes
- **Feature Folder**: docs/specs/features/001-admin-identity-domain/
- **Feature Spec**: docs/specs/features/001-admin-identity-domain/feature.spec.md
- **Feature Plan**: docs/specs/features/001-admin-identity-domain/plan.spec.md
- **Task File**: docs/specs/features/001-admin-identity-domain/tasks/T006-integration-guards-polish.task.spec.md
- **Related ADRs**: ADR-001, ADR-002
- **Dependencies**: T004, T005

## 1. Purpose

Apply `JwtAuthGuard` to all user CRUD endpoints, verify complete Swagger documentation across all controllers, run the full test suite to validate all acceptance scenarios end-to-end, and fix any gaps found during integration.

## 2. Scope

### In Scope

- Apply `@UseGuards(JwtAuthGuard)` to all endpoints in `UsersController` (or at controller level)
- Add `@ApiBearerAuth()` to `UsersController`
- Pass authenticated user context (`req.user`) to service methods for `criadoPor`/`modificadoPor`/`deletadoPor` audit fields
- Verify all Swagger decorators are complete (`@ApiTags`, `@ApiOperation`, `@ApiResponse`, `@ApiProperty`) across `AuthController` and `UsersController`
- Full integration test suite covering all acceptance scenarios from the feature spec
- Verify Swagger/Scalar UI at `/api/docs` renders all endpoints correctly with auth

### Out of Scope

- Implementing new CRUD logic (T004)
- Implementing auth logic (T005)
- Frontend integration
- Documentation updates (T007)

## 3. Context from Feature Plan

- **Plan slice**: Slice 6 — Integration, Guards, and Polish
- **Requirement refs**: FR-006, FR-011, FR-013
- **Affected Paths**:
  - `apps/admin/backend/src/identity/users/users.controller.ts` (modify — add guards + bearer auth)
  - `apps/admin/backend/src/identity/users/users.service.ts` (modify — accept actorId from controller for audit fields)
  - Various files for Swagger decorator completeness
- **Why this task exists**: In T004, CRUD endpoints were created without auth guards (to allow testing CRUD in isolation). This task applies the guards and validates the full end-to-end integration.

## 4. Technical Specification (Required)

### 4.1 Backend and API Contracts (when applicable)

All CRUD endpoints now require JWT Bearer authentication:

- `POST /usuarios-admin` → `@UseGuards(JwtAuthGuard)` — `criadoPor` set to authenticated user ID
- `GET /usuarios-admin` → `@UseGuards(JwtAuthGuard)`
- `GET /usuarios-admin/:id` → `@UseGuards(JwtAuthGuard)`
- `PATCH /usuarios-admin/:id` → `@UseGuards(JwtAuthGuard)` — `modificadoPor` set to authenticated user ID
- `DELETE /usuarios-admin/:id` → `@UseGuards(JwtAuthGuard)` — `deletadoPor` set to authenticated user ID

All endpoints return `401 Unauthorized` when:
- No Bearer token is provided
- The Bearer token is expired
- The Bearer token has been revoked (exists in Redis blacklist)

**Audit field mapping**:

| Audit Field | Source | Set On |
|-------------|--------|--------|
| `criadoPor` | `req.user.userId` | Create |
| `modificadoPor` | `req.user.userId` | Create, Update |
| `deletadoPor` | `req.user.userId` | Delete |

### 4.2 Database Specification (mandatory when data impact exists)

N/A — no schema changes. Audit fields already exist from T003.

### 4.3 Frontend and UX Contracts (when applicable)

N/A.

### 4.4 Cross-Cutting Constraints

- **Security**: Every CRUD endpoint MUST be protected. No endpoint should be accessible without a valid, non-revoked JWT.
- **Swagger**: All endpoints must appear in Swagger UI with correct auth requirements.
- **Test infrastructure**: Integration tests must handle authentication (login first, then use token for subsequent requests).

## 5. Implementation Steps

1. **Update `apps/admin/backend/src/identity/users/users.controller.ts`**:
   - Add `@UseGuards(JwtAuthGuard)` at the controller class level (applies to all endpoints)
   - Add `@ApiBearerAuth()` at the controller class level
   - Update method signatures to accept `@Request() req` and pass `req.user.userId` as `actorId` to service methods

2. **Update `apps/admin/backend/src/identity/users/users.service.ts`**:
   - Ensure `create()` sets `criadoPor` and `modificadoPor` from the passed `actorId`
   - Ensure `update()` sets `modificadoPor` from the passed `actorId`
   - Ensure `remove()` sets `deletadoPor` from the passed `actorId`

3. **Verify Swagger completeness**:
   - `AuthController`: Check all 3 endpoints have `@ApiOperation`, `@ApiResponse` for all status codes
   - `UsersController`: Check all 5 endpoints have `@ApiOperation`, `@ApiResponse` for all status codes
   - All DTOs: Check all fields have `@ApiProperty`
   - Visit `/api/docs` and verify all endpoints render correctly with auth lock icon

4. **Update integration tests**:
   - All CRUD tests must now authenticate first (login, obtain access token)
   - Add test: unauthenticated request to any CRUD endpoint → 401
   - Add test: request with revoked token to any CRUD endpoint → 401
   - Verify audit fields are correctly set (check `criadoPor`, `modificadoPor`, `deletadoPor` in DB after operations)

5. **Run full test suite**:
   ```bash
   pnpm nx run backend-tests:e2e
   ```
   All tests must pass.

6. **Final verification checklist**:
   - [ ] All feature spec acceptance scenarios covered by at least one integration test
   - [ ] All functional requirements (FR-001 through FR-020) are implemented
   - [ ] Swagger UI shows all endpoints with correct request/response schemas
   - [ ] No password fields leak in any response

## 6. Acceptance Criteria

- AC1: All CRUD endpoints (`POST`, `GET`, `GET/:id`, `PATCH/:id`, `DELETE/:id`) return 401 when called without a JWT Bearer token.
- AC2: All CRUD endpoints return 401 when called with a revoked JWT token.
- AC3: All CRUD endpoints function correctly when called with a valid JWT token.
- AC4: `criadoPor` is set to the authenticated user's ID on user creation.
- AC5: `modificadoPor` is set to the authenticated user's ID on user update.
- AC6: `deletadoPor` is set to the authenticated user's ID on user deletion.
- AC7: Swagger UI at `/api/docs` shows all 8 endpoints (3 auth + 5 CRUD) with correct schemas and auth requirements.
- AC8: Full integration test suite passes — covering all acceptance scenarios from the feature spec (Scenarios 1–6).

## 7. Test Scenarios

### Integration Tests (Full Flow)

- **I1**: Login → create user → list users → get user → update user → delete user → verify soft delete. All operations authenticated.
- **I2**: Attempt CRUD without token → all return 401.
- **I3**: Login → logout → attempt CRUD with revoked token → all return 401.
- **I4**: Login → create user with weak password → 400 (validation still works with auth).
- **I5**: Login → create user with duplicate email → 409.
- **I6**: Login → delete last admin → 400.
- **I7**: Login → refresh → use new access token for CRUD → works.
- **I8**: Verify `criadoPor` in DB matches the authenticated user's ID after creation.
- **I9**: Verify `deletadoPor` in DB is set after soft delete.

### Swagger Verification

- **S1**: Visit `/api/docs` → all endpoints listed under correct tags ("Auth", "Usuarios Admin").
- **S2**: Each endpoint shows request/response schemas that match the plan contracts.
- **S3**: Auth endpoints show lock icon where `@ApiBearerAuth()` is applied.

## 8. Definition of Ready (to start Step 4)

- [x] Status is `Draft` (will be `Ready` after approval).
- [x] Scope is explicit and bounded.
- [x] Required contracts are defined — guard application, audit field mapping, Swagger requirements.
- [x] Technical specification is detailed enough for independent implementation.
- [x] Acceptance criteria are testable.
- [x] Open questions are resolved.
- [x] Dependencies are completed or clearly sequenced — T004, T005 required first.

## 9. Definition of Done

- [x] Status moved to `Done`.
- [x] All acceptance criteria validated.
- [x] Full integration test suite passes.
- [x] Swagger UI verified complete.
- [x] Links to changed files/PR/tests are registered.
- [x] Feature spec and plan traceability remains intact.

## 10. Jira Mapping (Optional)

- **Issue Type**: Task
- **Summary**: Integration — apply JWT guards, Swagger polish, full test suite
- **Description**: Apply JwtAuthGuard to all CRUD endpoints, verify Swagger completeness, run full acceptance test suite.
- **Priority**: High
- **Labels**: Backend, Integration
- **Custom field (Tipo)**: Feature
- **Custom field (Tamanho)**: 2 Dias

## 11. Status Log

| Date       | Status | Notes |
| ---------- | ------ | ----- |
| 2026-04-15 | Draft  | Task created from approved feature plan |
| 2026-04-15 | Ready  | Task approved for implementation |
| 2026-04-15 | In Progress | Implementation started — applying guards, updating tests |
| 2026-04-15 | Done | Guards applied, Swagger polished, integration tests rewritten with auth |
