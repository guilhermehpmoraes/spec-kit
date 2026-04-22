# Task Spec: Install Scalar and replace Swagger UI

- **Feature ID**: 002-scalar-api-docs
- **Task ID**: T001
- **Status**: Done
- **Type**: Task
- **Parallelizable**: No
- **Parallelization Notes**: This is the foundational task — all other tasks depend on Scalar being installed and configured.
- **Date**: 2026-04-16
- **Owner**: Giovanni
- **Feature Folder**: docs/specs/features/002-scalar-api-docs/
- **Feature Spec**: docs/specs/features/002-scalar-api-docs/feature.spec.md
- **Feature Plan**: docs/specs/features/002-scalar-api-docs/plan.spec.md
- **Task File**: docs/specs/features/002-scalar-api-docs/tasks/T001-install-scalar-replace-swagger-ui.task.spec.md
- **Related ADRs**: N/A (ADR-010 will be created in T004)
- **Dependencies**: N/A

## 1. Purpose

Install `@scalar/nestjs-api-reference` and replace the default Swagger UI with Scalar API Reference in the Admin backend. This task delivers the core UI replacement: Scalar serves the API docs at `/api/docs` with Bearer Auth pre-selected, persistent authentication enabled, and a step-by-step auth guide in the OpenAPI description.

This is the foundational task for the entire feature — all subsequent tasks depend on it.

## 2. Scope

### In Scope

- Install `@scalar/nestjs-api-reference` as a `devDependency` in root `package.json`
- Run `pnpm install` to update lockfile
- Replace `SwaggerModule.setup("api/docs", app, document)` with `app.use("/api/docs", apiReference({ ... }))` in `main.ts`
- Configure Scalar with `authentication.preferredSecurityScheme: "bearer"`, `authentication.securitySchemes.bearer.token: ""`, and `persistAuth: true`
- Add `DocumentBuilder.setDescription()` with a markdown auth guide
- Verify the app starts and `/api/docs` serves the Scalar UI (manual smoke test)

### Out of Scope

- Swagger decorator changes on DTOs or controllers (T002)
- E2e test (T003)
- ADR or architecture doc updates (T004)
- Custom Scalar theming or branding

## 3. Context from Feature Plan

- **Plan slice**: Slice 1 — Install Scalar and replace Swagger UI
- **Requirement refs**: FR-001, FR-002, FR-003, FR-004, FR-006, FR-007, FR-008, AC-001, AC-002, AC-003
- **Affected Paths**:
  - `package.json` (root)
  - `pnpm-lock.yaml` (root)
  - `apps/admin/backend/src/main.ts`
- **Why this task exists**: Core replacement — everything else builds on top of a working Scalar setup.

## 4. Technical Specification (Required)

### 4.1 Backend and API Contracts (when applicable)

N/A — No API contract changes. This task only changes how existing API documentation is rendered.

### 4.2 Database Specification (mandatory when data impact exists)

N/A — No data impact.

### 4.3 Frontend and UX Contracts (when applicable)

N/A — Backend-only change (documentation UI is served by the backend).

### 4.4 Cross-Cutting Constraints

- **Security/authorization**: No change to auth logic. The Scalar UI config only pre-selects the Bearer Auth scheme and persists the token in `localStorage` for DX convenience.
- **Performance expectations**: N/A — documentation UI has no performance requirements.
- **Observability**: N/A.
- **Compatibility constraints**: `@scalar/nestjs-api-reference` must be compatible with NestJS v11 + Express adapter. Verified via Scalar docs.

### 4.5 Detailed Code Changes

**Current code** (`apps/admin/backend/src/main.ts`):
```typescript
import { Logger, ValidationPipe } from "@nestjs/common";
import { NestFactory } from "@nestjs/core";
import { DocumentBuilder, SwaggerModule } from "@nestjs/swagger";
import { AppModule } from "./app/app.module";

async function bootstrap() {
    const app = await NestFactory.create(AppModule);
    const globalPrefix = "api";
    app.setGlobalPrefix(globalPrefix);

    app.useGlobalPipes(
        new ValidationPipe({
            whitelist: true,
            forbidNonWhitelisted: true,
            transform: true,
        }),
    );

    const swaggerConfig = new DocumentBuilder().setTitle("Admin API").setVersion("1.0").addBearerAuth().build();
    const document = SwaggerModule.createDocument(app, swaggerConfig);
    SwaggerModule.setup("api/docs", app, document);

    const port = process.env.PORT || 3000;
    await app.listen(port);
    Logger.log(`🚀 Application is running on: http://localhost:${port}/${globalPrefix}`);
}

bootstrap();
```

**Target code** (`apps/admin/backend/src/main.ts`):
```typescript
import { Logger, ValidationPipe } from "@nestjs/common";
import { NestFactory } from "@nestjs/core";
import { DocumentBuilder, SwaggerModule } from "@nestjs/swagger";
import { apiReference } from "@scalar/nestjs-api-reference";
import { AppModule } from "./app/app.module";

async function bootstrap() {
    const app = await NestFactory.create(AppModule);
    const globalPrefix = "api";
    app.setGlobalPrefix(globalPrefix);

    app.useGlobalPipes(
        new ValidationPipe({
            whitelist: true,
            forbidNonWhitelisted: true,
            transform: true,
        }),
    );

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

    const port = process.env.PORT || 3000;
    await app.listen(port);
    Logger.log(`🚀 Application is running on: http://localhost:${port}/${globalPrefix}`);
}

bootstrap();
```

**Key changes:**
1. Add `import { apiReference } from "@scalar/nestjs-api-reference";`
2. Expand `DocumentBuilder` chain to include `.setDescription(...)` with auth guide markdown
3. Remove `SwaggerModule.setup("api/docs", app, document);`
4. Add `app.use("/api/docs", apiReference({ ... }));` with auth persistence config

## 5. Implementation Steps

1. Add `@scalar/nestjs-api-reference` to `devDependencies` in root `package.json`.
2. Run `pnpm install` to update `pnpm-lock.yaml`.
3. In `apps/admin/backend/src/main.ts`:
   - Add import: `import { apiReference } from "@scalar/nestjs-api-reference";`
   - Replace the single-line `DocumentBuilder` chain with the expanded version including `.setDescription(...)`.
   - Remove the line `SwaggerModule.setup("api/docs", app, document);`.
   - Add the `app.use("/api/docs", apiReference({ ... }));` block with auth config.
4. Run `pnpm nx run admin-backend:serve` and verify `/api/docs` loads the Scalar UI.
5. Run `pnpm nx run-many -t check` (Biome compliance).

## 6. Acceptance Criteria

- AC-001: Navigating to `http://localhost:3000/api/docs` renders the Scalar API reference UI (not Swagger UI).
- AC-002: Bearer Auth is pre-selected in the authentication section of the Scalar UI.
- AC-003: The old Swagger UI is no longer rendered — `SwaggerModule.setup` is not called.
- AC-004: The auth guide (4 steps) is visible in the API description section at the top of the Scalar docs.
- AC-005: After entering a JWT token and reloading the page, the token persists (localStorage).
- AC-006: All existing endpoints remain visible and correctly documented in Scalar.
- AC-007: Biome reports zero warnings and zero errors.

## 7. Test Scenarios

- Scenario 1: Start the Admin backend, navigate to `/api/docs` — Scalar UI loads with all endpoints.
- Scenario 2: Open the auth section — Bearer Auth is pre-selected, token input is visible.
- Scenario 3: Enter a token, reload the page — token is still present.
- Scenario 4: Verify auth guide text is rendered in the description area.

Note: Formal e2e test is in T003. This task uses manual smoke testing.

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

### Changed Files

- `package.json` — added `@scalar/nestjs-api-reference` to `devDependencies`
- `pnpm-lock.yaml` — lockfile updated
- `apps/admin/backend/src/main.ts` — replaced Swagger UI with Scalar API Reference

## 10. Jira Mapping (Optional)

- **Issue Type**: Task
- **Summary**: Install Scalar and replace Swagger UI in Admin backend
- **Description**: Install @scalar/nestjs-api-reference, replace SwaggerModule.setup with Scalar apiReference middleware, configure auth persistence and auth guide.
- **Priority**: Highest
- **Labels**: Backend
- **Custom field (Tipo)**: Feature
- **Custom field (Tamanho)**: 1 Dia

## 11. Status Log

| Date       | Status | Notes |
| ---------- | ------ | ----- |
| 2026-04-16 | Draft  | Task created from approved feature plan |
| 2026-04-16 | Ready  | Task approved for implementation |
| 2026-04-16 | In Progress | Implementation started |
| 2026-04-16 | Done   | Implementation complete — Scalar installed, main.ts updated, Biome clean |
