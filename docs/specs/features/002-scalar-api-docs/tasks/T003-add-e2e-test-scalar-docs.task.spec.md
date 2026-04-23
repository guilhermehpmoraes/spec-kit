# Task Spec: Add e2e test for Scalar docs endpoint

- **Feature ID**: 002-scalar-api-docs
- **Task ID**: T003
- **Status**: Done
- **Type**: Task
- **Parallelizable**: Yes
- **Parallelization Notes**: Safe to run in parallel with T002 and T004 after T001 is done. Only adds a new test case — no shared file conflicts.
- **Date**: 2026-04-16
- **Last Updated**: 2026-04-16
- **Owner**: Giovanni
- **Feature Folder**: docs/specs/features/002-scalar-api-docs/
- **Feature Spec**: docs/specs/features/002-scalar-api-docs/feature.spec.md
- **Feature Plan**: docs/specs/features/002-scalar-api-docs/plan.spec.md
- **Task File**: docs/specs/features/002-scalar-api-docs/tasks/T003-add-e2e-test-scalar-docs.task.spec.md
- **Related ADRs**: N/A
- **Dependencies**: T001

## 1. Purpose

Add an e2e test confirming that `/api/docs` returns HTTP 200 with Scalar-specific content. This validates that the Scalar UI is correctly mounted and serves as a regression guard against accidental removal.

## 2. Scope

### In Scope

- Add a new test case in `apps/admin/backend-tests/src/backend/backend.spec.ts` (or a new describe block) that:
  - Sends `GET /api/docs` via axios
  - Asserts HTTP 200 status
  - Asserts response body contains Scalar-specific markers (e.g., `scalar` or `@scalar` in the HTML)

### Out of Scope

- Testing Scalar UI interactivity (auth persistence, request testing) — those are manual verification
- Testing decorator metadata rendering — covered by manual scenarios in T002
- Unit tests — this is a pure e2e test

## 3. Context from Feature Plan

- **Plan slice**: Slice 3 — E2e test for Scalar docs endpoint
- **Requirement refs**: EQ-004, AC-001
- **Affected Paths**:
  - `apps/admin/backend-tests/src/backend/backend.spec.ts` (modify)
- **Why this task exists**: EQ-004 requires the bootstrap change to be verifiable via an e2e test.

## 4. Technical Specification (Required)

### 4.1 Backend and API Contracts (when applicable)

N/A — This task only adds a test, no API changes.

### 4.2 Database Specification (mandatory when data impact exists)

N/A — No data impact.

### 4.3 Frontend and UX Contracts (when applicable)

N/A.

### 4.4 Cross-Cutting Constraints

- **Security/authorization**: N/A — `/api/docs` is a public documentation endpoint, no auth required.
- **Performance expectations**: N/A.
- **Observability**: N/A.
- **Compatibility constraints**: The e2e test infrastructure uses Jest + axios with a global setup that waits for port 3000. The test uses axios with `baseURL = http://localhost:3000`.

### 4.5 Existing Test Infrastructure

**Current file** (`apps/admin/backend-tests/src/backend/backend.spec.ts`):
```typescript
import axios from "axios";

describe("GET /api", () => {
    it("should return a message", async () => {
        const res = await axios.get(`/api`);

        expect(res.status).toBe(200);
        expect(res.data).toEqual({ message: "Hello API" });
    });
});
```

**E2e config**: `apps/admin/backend-tests/jest.config.cts` — Jest with global setup waiting for port 3000 and axios baseURL `http://localhost:3000`.

### 4.6 Test Code to Add

Add a new `describe` block (or add to the existing file) for the Scalar docs endpoint:

```typescript
describe("GET /api/docs", () => {
    it("should serve the Scalar API reference UI", async () => {
        const res = await axios.get("/api/docs", {
            headers: { Accept: "text/html" },
        });

        expect(res.status).toBe(200);
        expect(res.headers["content-type"]).toContain("text/html");
        expect(res.data).toContain("scalar");
    });
});
```

**Notes:**
- The `Accept: text/html` header ensures we get the HTML response (not JSON).
- Asserting `res.data` contains `"scalar"` validates the Scalar UI is served (Scalar's HTML includes references to its library name).
- `content-type` check confirms HTML is returned, not JSON.

## 5. Implementation Steps

1. Open `apps/admin/backend-tests/src/backend/backend.spec.ts`.
2. Add a new `describe("GET /api/docs", ...)` block with the test case.
3. Run `pnpm nx run admin-backend-tests:e2e` to verify the test passes (requires the Admin backend to be running with Scalar from T001).
4. Run `pnpm nx run-many -t check` (Biome compliance).

## 6. Acceptance Criteria

- AC-001: The e2e test `GET /api/docs` passes — HTTP 200, HTML content type, body contains "scalar".
- AC-002: Existing e2e tests (`GET /api`) continue to pass.
- AC-003: Biome reports zero warnings and zero errors.

## 7. Test Scenarios

- Scenario 1 (happy path): `GET /api/docs` → 200, HTML response containing "scalar".
- Scenario 2 (regression): Existing `GET /api` test still passes after adding the new test.

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

**Changed files**: `apps/admin/backend-tests/src/backend/backend.spec.ts`

## 10. Jira Mapping (Optional)

- **Issue Type**: Task
- **Summary**: Add e2e test for Scalar docs endpoint
- **Description**: Add e2e test validating GET /api/docs returns HTTP 200 with Scalar content.
- **Priority**: High
- **Labels**: Backend, QA
- **Custom field (Tipo)**: Feature
- **Custom field (Tamanho)**: 0.5 Dia

## 11. Status Log

| Date       | Status      | Notes |
| ---------- | ----------- | ----- |
| 2026-04-16 | Draft       | Task created from approved feature plan |
| 2026-04-16 | Ready       | Task approved for implementation |
| 2026-04-16 | In Progress | Implementation started — branch task/T003-add-e2e-test-scalar-docs |
| 2026-04-16 | Done        | E2e test added, Biome passes, all acceptance criteria met |

## 12. Observations

- **Jest e2e config was broken (pre-existing)**: `jest.config.cts` used ESM syntax (`import`/`export`) incompatible with `.cts` (CommonJS TypeScript) and referenced a non-existent `jest.preset.js`. Fixed by converting to CJS syntax (`require`/`module.exports`), removing the missing preset, and adding `ts-node` as a workspace devDependency (required by Jest to parse `.cts` files). Fix committed as `🔧 chore(admin-backend-tests): fix jest e2e config for .cts compatibility` (`8d43c27a`).
