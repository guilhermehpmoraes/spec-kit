# Task Spec: Standardize existing swagger decorators

- **Feature ID**: 002-scalar-api-docs
- **Task ID**: T002
- **Status**: Done
- **Type**: Task
- **Parallelizable**: No
- **Parallelization Notes**: Depends on T001 (Scalar must be installed and working so decorator changes are visible in the new UI). Cannot run before T001 is done.
- **Date**: 2026-04-16
- **Last Updated**: 2026-04-16
- **Owner**: Giovanni
- **Feature Folder**: docs/specs/features/002-scalar-api-docs/
- **Feature Spec**: docs/specs/features/002-scalar-api-docs/feature.spec.md
- **Feature Plan**: docs/specs/features/002-scalar-api-docs/plan.spec.md
- **Task File**: docs/specs/features/002-scalar-api-docs/tasks/T002-standardize-swagger-decorators.task.spec.md
- **Related ADRs**: N/A (decorator standard documented in T004)
- **Dependencies**: T001

## 1. Purpose

Bring all existing DTOs and controllers in the Admin backend up to a full OpenAPI documentation standard. Every `@ApiProperty` must have `description` and `example`, every non-void endpoint must have a typed response DTO in `@ApiResponse`, and missing swagger decorators must be added to `AppController`. This ensures the Scalar UI (installed in T001) displays complete, self-documenting API schemas.

## 2. Scope

### In Scope

- Create `LoginResponseDto` with `accessToken` and `refreshToken` fields
- Create `RefreshResponseDto` with `accessToken` field
- Add `description` to `@ApiProperty` in `login.dto.ts` (already has `example`)
- Add `description` and `example` to `@ApiProperty` in `refresh-token.dto.ts`
- Add `description` and `example` to `@ApiProperty` in `logout.dto.ts`
- Add `description` to `@ApiProperty` in `create-user.dto.ts` (already has `example`)
- Add `description` and `example` to all `@ApiProperty` in `user-response.dto.ts`
- Add `type: LoginResponseDto` to the 200 `@ApiResponse` on `auth.controller.ts` login endpoint
- Add `type: RefreshResponseDto` to the 200 `@ApiResponse` on `auth.controller.ts` refresh endpoint
- Add `@ApiTags("Health")`, `@ApiOperation`, and `@ApiResponse` to `AppController`

### Out of Scope

- Changing any business logic, validation rules, or API behavior
- Modifying the Scalar UI configuration (done in T001)
- E2e testing (T003)
- Architecture docs (T004)

## 3. Context from Feature Plan

- **Plan slice**: Slice 2 — Standardize existing swagger decorators
- **Requirement refs**: FR-009, FR-010, FR-011, FR-012, SC-002, SC-003
- **Affected Paths**:
  - `apps/admin/backend/src/identity/auth/dto/login.dto.ts` (modify)
  - `apps/admin/backend/src/identity/auth/dto/refresh-token.dto.ts` (modify)
  - `apps/admin/backend/src/identity/auth/dto/logout.dto.ts` (modify)
  - `apps/admin/backend/src/identity/auth/dto/login-response.dto.ts` (create)
  - `apps/admin/backend/src/identity/auth/dto/refresh-response.dto.ts` (create)
  - `apps/admin/backend/src/identity/auth/auth.controller.ts` (modify)
  - `apps/admin/backend/src/identity/users/dto/create-user.dto.ts` (modify)
  - `apps/admin/backend/src/identity/users/dto/user-response.dto.ts` (modify)
  - `apps/admin/backend/src/app/app.controller.ts` (modify)
- **Why this task exists**: The Scalar UI is only as good as the underlying OpenAPI decorators. Standardizing all existing decorators ensures the full API schema is visible and consistent.

## 4. Technical Specification (Required)

### 4.1 Backend and API Contracts (when applicable)

No API behavior changes. Only documentation metadata is added/enhanced.

### 4.2 Database Specification (mandatory when data impact exists)

N/A — No data impact.

### 4.3 Frontend and UX Contracts (when applicable)

N/A — Backend-only change.

### 4.4 Cross-Cutting Constraints

- **Security/authorization**: No change.
- **Performance expectations**: N/A.
- **Observability**: N/A.
- **Compatibility constraints**: N/A.

### 4.5 Decorator Standard

Every DTO field must follow this pattern:

```typescript
@ApiProperty({
    description: "Brief description of the field purpose",
    example: "concrete example value",
})
fieldName!: string;
```

### 4.6 New Response DTOs

**`login-response.dto.ts`** — Create at `apps/admin/backend/src/identity/auth/dto/login-response.dto.ts`:

```typescript
import { ApiProperty } from "@nestjs/swagger";

export class LoginResponseDto {
    @ApiProperty({
        description: "JWT access token for authenticating API requests",
        example: "eyJhbGciOiJIUzI1NiIs...",
    })
    accessToken!: string;

    @ApiProperty({
        description: "JWT refresh token for obtaining new access tokens",
        example: "eyJhbGciOiJIUzI1NiIs...",
    })
    refreshToken!: string;
}
```

**`refresh-response.dto.ts`** — Create at `apps/admin/backend/src/identity/auth/dto/refresh-response.dto.ts`:

```typescript
import { ApiProperty } from "@nestjs/swagger";

export class RefreshResponseDto {
    @ApiProperty({
        description: "New JWT access token",
        example: "eyJhbGciOiJIUzI1NiIs...",
    })
    accessToken!: string;
}
```

### 4.7 Existing DTO Enhancements

**`login.dto.ts`** — Current vs target:

| Field | Current | Target |
|-------|---------|--------|
| `email` | `@ApiProperty({ example: "admin@satie.local" })` | `@ApiProperty({ description: "Admin user email address", example: "admin@satie.local" })` |
| `senha` | `@ApiProperty({ example: "P@ssw0rd" })` | `@ApiProperty({ description: "Admin user password", example: "P@ssw0rd" })` |

**`refresh-token.dto.ts`** — Current vs target:

| Field | Current | Target |
|-------|---------|--------|
| `refreshToken` | `@ApiProperty()` | `@ApiProperty({ description: "JWT refresh token received from login", example: "eyJhbGciOiJIUzI1NiIs..." })` |

**`logout.dto.ts`** — Current vs target:

| Field | Current | Target |
|-------|---------|--------|
| `refreshToken` | `@ApiProperty()` | `@ApiProperty({ description: "JWT refresh token to revoke", example: "eyJhbGciOiJIUzI1NiIs..." })` |

**`create-user.dto.ts`** — Current vs target:

| Field | Current | Target |
|-------|---------|--------|
| `email` | `@ApiProperty({ example: "user@example.com" })` | `@ApiProperty({ description: "Email address for the new admin user", example: "user@example.com" })` |
| `senha` | `@ApiProperty({ example: "P@ssw0rd" })` | `@ApiProperty({ description: "Password (min 8 chars, 1 uppercase, 1 number, 1 special char)", example: "P@ssw0rd" })` |

**`user-response.dto.ts`** — Current vs target:

| Field | Current | Target |
|-------|---------|--------|
| `idUsuarioAdmin` | `@ApiProperty()` | `@ApiProperty({ description: "Unique identifier of the admin user", example: "a1b2c3d4-e5f6-7890-abcd-ef1234567890" })` |
| `email` | `@ApiProperty()` | `@ApiProperty({ description: "Admin user email address", example: "admin@satie.local" })` |
| `criadoAs` | `@ApiProperty()` | `@ApiProperty({ description: "Timestamp when the user was created", example: "2026-01-15T10:30:00.000Z" })` |
| `modificadoAs` | `@ApiProperty()` | `@ApiProperty({ description: "Timestamp when the user was last modified", example: "2026-01-15T10:30:00.000Z" })` |

### 4.8 Controller Changes

**`auth.controller.ts`** — Add `type` to the 200 `@ApiResponse`:

- Login endpoint: `@ApiResponse({ status: 200, description: "Login successful — returns access and refresh tokens" })` → `@ApiResponse({ status: 200, description: "Login successful — returns access and refresh tokens", type: LoginResponseDto })`
- Refresh endpoint: `@ApiResponse({ status: 200, description: "Token refreshed — returns new access token" })` → `@ApiResponse({ status: 200, description: "Token refreshed — returns new access token", type: RefreshResponseDto })`
- Add imports for `LoginResponseDto` and `RefreshResponseDto`.

**`app.controller.ts`** — Add swagger decorators:

```typescript
import { Controller, Get } from "@nestjs/common";
import { ApiOperation, ApiResponse, ApiTags } from "@nestjs/swagger";
import { AppService } from "./app.service";

@ApiTags("Health")
@Controller()
export class AppController {
    constructor(private readonly appService: AppService) {}

    @Get()
    @ApiOperation({ summary: "Health check" })
    @ApiResponse({ status: 200, description: "Application is running" })
    getData() {
        return this.appService.getData();
    }
}
```

## 5. Implementation Steps

1. Create `apps/admin/backend/src/identity/auth/dto/login-response.dto.ts` with `LoginResponseDto`.
2. Create `apps/admin/backend/src/identity/auth/dto/refresh-response.dto.ts` with `RefreshResponseDto`.
3. Update `login.dto.ts` — add `description` to both `@ApiProperty` decorators.
4. Update `refresh-token.dto.ts` — add `description` and `example` to `@ApiProperty`.
5. Update `logout.dto.ts` — add `description` and `example` to `@ApiProperty`.
6. Update `create-user.dto.ts` — add `description` to both `@ApiProperty` decorators.
7. Update `user-response.dto.ts` — add `description` and `example` to all 4 `@ApiProperty` decorators.
8. Update `auth.controller.ts` — import response DTOs, add `type` to both 200 `@ApiResponse`.
9. Update `app.controller.ts` — add `@ApiTags("Health")`, `@ApiOperation`, `@ApiResponse` imports and decorators.
10. Run `pnpm nx run-many -t check` (Biome compliance).
11. Start backend and verify all endpoints show full schemas in Scalar UI.

## 6. Acceptance Criteria

- AC-001: Every `@ApiProperty` in the Admin backend has both `description` and `example`.
- AC-002: Login endpoint's 200 response shows `LoginResponseDto` schema (with `accessToken` and `refreshToken`) in Scalar.
- AC-003: Refresh endpoint's 200 response shows `RefreshResponseDto` schema (with `accessToken`) in Scalar.
- AC-004: `AppController` health check appears under "Health" tag in Scalar with operation summary and response description.
- AC-005: No `@ApiProperty()` with empty config remains in any DTO.
- AC-006: Biome reports zero warnings and zero errors.

## 7. Test Scenarios

- Scenario 1: Open Scalar UI → login endpoint → verify request body schema shows field descriptions and examples.
- Scenario 2: Open Scalar UI → login endpoint → verify 200 response schema shows `accessToken` and `refreshToken` with descriptions.
- Scenario 3: Open Scalar UI → refresh endpoint → verify 200 response schema shows `accessToken` with description.
- Scenario 4: Open Scalar UI → verify "Health" tag exists with the health check endpoint.
- Scenario 5: Open Scalar UI → user endpoints → verify `UserResponseDto` fields have descriptions and examples.

Note: These are manual verification scenarios. No automated tests needed for decorator metadata.

## 8. Definition of Ready (to start Step 4)

A task is ready for implementation only if:

- [ ] Status is `Ready`.
- [ ] Scope is explicit and bounded.
- [ ] Required contracts are defined (API, DTO, event, UI state, schema).
- [ ] Technical specification is detailed enough for independent implementation.
- [ ] For data-impact tasks, table/field/type/constraint/index/migration details are fully documented.
- [ ] Acceptance criteria are testable.
- [ ] Open questions are resolved or captured as explicit assumptions.
- [ ] Dependencies are completed or clearly sequenced.

## 9. Definition of Done

- [x] Status moved to `Done`.
- [x] All related tests (unit and integration) pass — no test failures allowed.
- [x] Biome reports zero warnings and zero errors on all changed files.
- [x] Acceptance criteria are validated by tests or clear verification evidence.
- [x] Edge cases listed in this task are covered.
- [x] Links to changed files/PR/tests are registered.
- [x] Feature spec and plan traceability remains intact.

## 10. Changed Files

- `apps/admin/backend/src/identity/auth/dto/login-response.dto.ts` (created)
- `apps/admin/backend/src/identity/auth/dto/refresh-response.dto.ts` (created)
- `apps/admin/backend/src/identity/auth/dto/login.dto.ts` (modified)
- `apps/admin/backend/src/identity/auth/dto/refresh-token.dto.ts` (modified)
- `apps/admin/backend/src/identity/auth/dto/logout.dto.ts` (modified)
- `apps/admin/backend/src/identity/auth/auth.controller.ts` (modified)
- `apps/admin/backend/src/identity/users/dto/create-user.dto.ts` (modified)
- `apps/admin/backend/src/identity/users/dto/user-response.dto.ts` (modified)
- `apps/admin/backend/src/app/app.controller.ts` (modified)

## 11. Status Log

| Date | From | To | Notes |
|------|------|----|-------|
| 2026-04-16 | Ready | In Progress | Implementation started |
| 2026-04-16 | In Progress | Done | All decorator enhancements applied, Biome clean |

## 10. Jira Mapping (Optional)

- **Issue Type**: Task
- **Summary**: Standardize existing swagger decorators across Admin backend
- **Description**: Enhance all @ApiProperty decorators with description/example, create LoginResponseDto and RefreshResponseDto, add swagger decorators to AppController.
- **Priority**: High
- **Labels**: Backend
- **Custom field (Tipo)**: Feature
- **Custom field (Tamanho)**: 1 Dia

## 11. Status Log

| Date       | Status | Notes |
| ---------- | ------ | ----- |
| 2026-04-16 | Draft  | Task created from approved feature plan |
