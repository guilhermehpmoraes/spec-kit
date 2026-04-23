# Feature Plan: Scalar API Documentation

- **Feature ID**: 002-scalar-api-docs
- **Plan ID**: PLAN-002-scalar-api-docs
- **Status**: Completed
- **Date**: 2026-04-16
- **Last Updated**: 2026-04-16
- **Owner**: Giovanni
- **Feature Folder**: docs/specs/features/002-scalar-api-docs/
- **Feature Spec**: docs/specs/features/002-scalar-api-docs/feature.spec.md
- **Plan File**: docs/specs/features/002-scalar-api-docs/plan.spec.md
- **Task Output Folder**: docs/specs/features/002-scalar-api-docs/tasks/
- **Related ADRs**: ADR-006 (single-version dependency policy), ADR-010 (to be created — Scalar as standard API docs UI)

## 1. Planning Goal

Define the implementation approach for replacing Swagger UI with Scalar API Reference across all NestJS backends in the monorepo, with enough detail to generate self-sufficient task specs.

## 2. Summary

- **Feature objective**: Replace the default Swagger UI with Scalar API Reference for all NestJS backends, improving API documentation DX and visual quality.
- **Why now**: The current Swagger UI is functional but dated; Scalar offers a modern interface with better navigation and built-in request testing. Adopting it early establishes a consistent standard before more backends are created.
- **Expected outcome**: Developers navigating to `/api/docs` on any NestJS backend see the Scalar API reference UI. All existing OpenAPI decorators and spec generation remain unchanged.
- **Implementation direction**: Install `@scalar/nestjs-api-reference`, replace `SwaggerModule.setup()` with Scalar's `apiReference` middleware in `main.ts`, keep `SwaggerModule.createDocument()` intact, add e2e test, create ADR-010, and update architecture docs.

## 3. Technical Context

### Stack Baseline (Satie)

- **Monorepo**: Nx + pnpm
- **Backend**: NestJS v11
- **Frontend**: Vite + React + TanStack Router + TanStack Query + Tailwind CSS
- **Database**: PostgreSQL
- **Tests**: Jest + Supertest (backend), Vitest + React Testing Library (frontend), Playwright (e2e)

### Feature-Specific Context

- **Touched apps/packages**: `apps/admin/backend/src/main.ts`, root `package.json`
- **New dependencies**: `@scalar/nestjs-api-reference` (devDependencies in root `package.json`)
- **Data impact**: N/A
- **Constraints**: Must keep the same `/api/docs` route. Must not modify any `@nestjs/swagger` decorators.

### Constraints and Assumptions

- **Inputs**: The OpenAPI document object created by `SwaggerModule.createDocument(app, config)` is passed directly to Scalar's `apiReference({ content: document })`.
- **Outputs**: Scalar renders an HTML page at `/api/docs` with the full API reference.
- **Boundary conditions**: If the OpenAPI document is empty or malformed, Scalar will render an empty reference (same behavior as Swagger UI — no special handling needed).
- **Assumption**: `@scalar/nestjs-api-reference` is compatible with NestJS v11 and Express. Verified via Scalar documentation — the package is Express middleware compatible with any NestJS version using Express adapter.

## 3.1 Technical Design Baseline (Required)

### Database Design

N/A — No data impact.

### API and Integration Contracts

N/A — No API changes. This feature only changes how existing API documentation is rendered.

### UI Contracts

N/A — Backend-only change (documentation UI is served by the backend, not the frontend app).

### Scalar Integration Pattern (Key Technical Detail)

**Current code** (`apps/admin/backend/src/main.ts`, lines 19–21):
```typescript
const swaggerConfig = new DocumentBuilder().setTitle("Admin API").setVersion("1.0").addBearerAuth().build();
const document = SwaggerModule.createDocument(app, swaggerConfig);
SwaggerModule.setup("api/docs", app, document);
```

**Target code** after replacement:
```typescript
import { apiReference } from "@scalar/nestjs-api-reference";

const swaggerConfig = new DocumentBuilder()
    .setTitle("Admin API")
    .setDescription(
        "## Authentication\n\n" +
        "1. Call **POST /api/auth/login** with your `email` and `senha`\n" +
        "2. Copy the `accessToken` from the response\n" +
        "3. Click the **Authorize** button (🔒) and paste the token\n" +
        "4. Done — the token is saved automatically across reloads",
    )
    .setVersion("1.0")
    .addBearerAuth()
    .build();

const swaggerConfig = new DocumentBuilder().setTitle("Admin API").setVersion("1.0").addBearerAuth().build();
const document = SwaggerModule.createDocument(app, swaggerConfig);

app.use(
    "/api/docs",
    apiReference({
        content: document,
        authentication: {
            preferredSecurityScheme: "bearer",
            securitySchemes: {
                bearer: {
                    token: "",
                },
            },
        },
        persistAuth: true,
    }),
);
```

**Key changes:**
1. Remove `SwaggerModule.setup("api/docs", app, document)` — this served the Swagger UI.
2. Add `app.use("/api/docs", apiReference({ ... }))` — this serves the Scalar UI.
3. Add import for `apiReference` from `@scalar/nestjs-api-reference`.
4. The `DocumentBuilder` configuration and `SwaggerModule.createDocument()` remain unchanged.
5. The `SwaggerModule` import can be simplified to only `{ DocumentBuilder, SwaggerModule }` (both are still needed).

**Scalar `apiReference` options used:**
- `content`: The OpenAPI document object (passed directly, no need for a separate JSON endpoint).
- `authentication.preferredSecurityScheme`: Set to `"bearer"` so the Bearer Auth input is pre-selected when the developer opens the docs — no need to hunt for it.
- `authentication.securitySchemes.bearer.token`: Empty string placeholder — the developer fills in their JWT once.
- `persistAuth: true`: Saves the entered token in the browser's `localStorage`. Once a developer authenticates, the token persists across page reloads and sessions, so they only authenticate once. This is the key DX improvement.

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

### Slice 1 — Install Scalar and replace Swagger UI

- **Goal**: Install `@scalar/nestjs-api-reference`, update `main.ts` to use Scalar instead of `SwaggerModule.setup` with Bearer Auth pre-selected, persistent authentication enabled, and an auth guide in the description.
- **Files to touch**:
  - `package.json` (root) — add `@scalar/nestjs-api-reference` to `devDependencies`
  - `apps/admin/backend/src/main.ts` — replace Swagger UI setup with Scalar `apiReference` middleware including `authentication`, `persistAuth`, and `setDescription` config
- **Requirements covered**: FR-001, FR-002, FR-003, FR-004, FR-006, FR-007, FR-008
- **Acceptance covered**: AC-001, AC-002, AC-003, AC-004 (Bearer pre-selected), AC-005 (persistent auth)
- **Expected outputs**: Modified `main.ts`, updated `package.json`, working Scalar UI at `/api/docs` with auth persistence

### Slice 2 — Standardize existing swagger decorators

- **Goal**: Bring all existing DTOs and controllers up to the full documentation standard: every `@ApiProperty` with `description` + `example`, create response DTOs for auth endpoints, add swagger decorators to `AppController`.
- **Files to touch**:
  - `apps/admin/backend/src/identity/auth/dto/login.dto.ts` — add `description` to existing `@ApiProperty`
  - `apps/admin/backend/src/identity/auth/dto/refresh-token.dto.ts` — add `description` and `example`
  - `apps/admin/backend/src/identity/auth/dto/logout.dto.ts` — add `description` and `example`
  - `apps/admin/backend/src/identity/auth/dto/login-response.dto.ts` — **create** new response DTO
  - `apps/admin/backend/src/identity/auth/dto/refresh-response.dto.ts` — **create** new response DTO
  - `apps/admin/backend/src/identity/auth/auth.controller.ts` — add `type` to `@ApiResponse` for login/refresh
  - `apps/admin/backend/src/identity/users/dto/create-user.dto.ts` — add `description`
  - `apps/admin/backend/src/identity/users/dto/user-response.dto.ts` — add `description` and `example`
  - `apps/admin/backend/src/app/app.controller.ts` — add `@ApiTags`, `@ApiOperation`, `@ApiResponse`
- **Requirements covered**: FR-009, FR-010, FR-011, FR-012
- **Acceptance covered**: SC-002, SC-003
- **Expected outputs**: Fully annotated DTOs and controllers; auth endpoints with typed response schemas visible in Scalar

#### Detailed Decorator Standard

Every DTO field must follow this pattern:

```typescript
@ApiProperty({
    description: "Brief description of the field purpose",
    example: "concrete example value",
})
fieldName!: string;
```

Response DTOs to create:

**`LoginResponseDto`**:
| Field | Type | Example | Description |
|-------|------|---------|-------------|
| `accessToken` | `string` | `"eyJhbGciOiJIUzI1NiIs..."` | JWT access token for authenticating API requests |
| `refreshToken` | `string` | `"eyJhbGciOiJIUzI1NiIs..."` | JWT refresh token for obtaining new access tokens |

**`RefreshResponseDto`**:
| Field | Type | Example | Description |
|-------|------|---------|-------------|
| `accessToken` | `string` | `"eyJhbGciOiJIUzI1NiIs..."` | New JWT access token |

**`AppController` decorators to add**:
- `@ApiTags("Health")` on class
- `@ApiOperation({ summary: "Health check" })` on `getData()`
- `@ApiResponse({ status: 200, description: "Application is running" })` on `getData()`

### Slice 3 — E2e test for Scalar docs endpoint

- **Goal**: Add an e2e test confirming `/api/docs` returns HTTP 200 with Scalar-specific content.
- **Files to touch**:
  - `apps/admin/backend-tests/src/backend/backend.spec.ts` — add test case for `/api/docs` endpoint
- **Requirements covered**: EQ-004
- **Acceptance covered**: AC-001
- **Expected outputs**: New e2e test case that validates Scalar is served at `/api/docs`

### Slice 4 — ADR and documentation updates

- **Goal**: Record the decision to adopt Scalar as the standard API docs UI (ADR-010), document the decorator standard in the Admin architecture spec, and update Swagger references to Scalar.
- **Files to touch**:
  - `docs/decisions/010-scalar-api-documentation.md` — new ADR
  - `docs/specs/apps/admin/architecture.md` — update Swagger references to Scalar, add API documentation standard section
- **Requirements covered**: FR-005, FR-013
- **Acceptance covered**: N/A (documentation)
- **Expected outputs**: ADR-010 file, updated architecture spec with decorator standard

## 6. Task Generation Matrix

| Task ID | Title | Type | Plan Slice | Parallelizable | Dependencies | Requirement Refs | Technical Scope | Output File |
| ------- | ----- | ---- | ---------- | -------------- | ------------ | ---------------- | --------------- | ----------- |
| T001 | Install Scalar and replace Swagger UI | Task | Slice 1 | No | N/A | FR-001, FR-002, FR-003, FR-004, FR-006, FR-007, FR-008, AC-001, AC-002, AC-003 | Install dependency, update `main.ts` with auth config and description | docs/specs/features/002-scalar-api-docs/tasks/T001-install-scalar-replace-swagger-ui.task.spec.md |
| T002 | Standardize existing swagger decorators | Task | Slice 2 | No | T001 | FR-009, FR-010, FR-011, FR-012 | Create response DTOs, enhance all DTOs, annotate AppController | docs/specs/features/002-scalar-api-docs/tasks/T002-standardize-swagger-decorators.task.spec.md |
| T003 | Add e2e test for Scalar docs endpoint | Task | Slice 3 | No | T001 | EQ-004, AC-001 | E2e test in backend-tests | docs/specs/features/002-scalar-api-docs/tasks/T003-add-e2e-test-scalar-docs.task.spec.md |
| T004 | Create ADR-010 and update architecture docs | Task | Slice 4 | Yes (with T002, T003) | T001 | FR-005, FR-013 | ADR file, architecture spec with decorator standard | docs/specs/features/002-scalar-api-docs/tasks/T004-adr-and-docs-update.task.spec.md |

## 7. Task File Generation Rules

- Create one file per task under `docs/specs/features/002-scalar-api-docs/tasks/`.
- Use naming pattern: `TXXX-<short-title>.task.spec.md`.
- Use `docs/specs/templates/task.spec.md` for every task file.
- Each task must reference the same feature spec and this feature plan.
- Start each task with status `Draft`; set to `Ready` after task-level approval.
- If a task cannot be delivered safely in one branch cycle, split it before approval.

## 8. Repository Impact

```text
apps/
  admin/
    backend/
      src/
        main.ts                                    (modified — Scalar replaces Swagger UI setup)
        app/
          app.controller.ts                        (modified — add swagger decorators)
        identity/
          auth/
            auth.controller.ts                     (modified — add response types to @ApiResponse)
            dto/
              login.dto.ts                         (modified — add description to @ApiProperty)
              refresh-token.dto.ts                 (modified — add description and example)
              logout.dto.ts                        (modified — add description and example)
              login-response.dto.ts                (created — response DTO for login endpoint)
              refresh-response.dto.ts              (created — response DTO for refresh endpoint)
          users/
            dto/
              create-user.dto.ts                   (modified — add description)
              user-response.dto.ts                 (modified — add description and example)
    backend-tests/
      src/
        backend/
          backend.spec.ts                          (modified — new e2e test case)
docs/
  decisions/
    010-scalar-api-documentation.md                (created — ADR for Scalar adoption + decorator standard)
  specs/
    apps/
      admin/
        architecture.md                            (modified — update Swagger references, add decorator standard)
package.json                                       (modified — add @scalar/nestjs-api-reference)
pnpm-lock.yaml                                     (modified — lockfile update from install)
```

### Planned Changes by Area

- **Backend**: `main.ts` (Scalar setup), `app.controller.ts` (swagger decorators), `auth.controller.ts` (response types), 5 DTO files (enhance decorators), 2 new response DTOs
- **Frontend**: No changes
- **Shared packages**: No changes
- **Database**: No changes
- **QA**: `backend.spec.ts` — new e2e test for `/api/docs`
- **Docs**: New ADR-010, updated Admin architecture spec with decorator standard

## 9. Validation Strategy

- **Unit tests**: N/A — no business logic changes
- **Integration tests**: N/A — no service-level changes
- **E2E tests**: Add test that `GET /api/docs` returns HTTP 200 with response body containing Scalar-specific markers (e.g., `scalar` or `@scalar` in the HTML). This validates the Scalar UI is correctly mounted.
- **Non-functional checks**: Manual verification that all existing endpoints are visible in the Scalar UI and that Bearer Auth is displayed.
- **Biome compliance**: Run `pnpm nx run-many -t check` after code changes.

## 10. Rollout and Safety

- **Feature flags**: N/A — this is a direct replacement with no gradual rollout needed.
- **Backward compatibility**: The `/api/docs` route remains the same. No external consumers are affected. No API contract changes.
- **Monitoring/observability**: N/A — documentation UI has no runtime monitoring requirements.
- **Rollback plan**: Revert `main.ts` to use `SwaggerModule.setup()` and remove `@scalar/nestjs-api-reference` dependency. One-commit revert.

## 11. Step 2 Completion Checklist

- [ ] Plan is approved.
- [ ] Technical design baseline is complete for all impacted layers.
- [ ] Scope-to-execution mapping covers all feature requirements.
- [ ] Task generation matrix is defined and consistent with scope.
- [ ] Dependencies between planned slices are explicit.
- [ ] Feature spec status is updated to `Planned`.
- [ ] Plan is ready to hand off to Step 3 (Task Breakdown).

## 12. Post-Implementation Feedback

_To be completed after feature delivery._
