# Task Spec: Rename Test Projects to *-tests Convention

- **Feature ID**: 001-admin-identity-domain
- **Task ID**: T000
- **Status**: Done
- **Type**: Task
- **Parallelizable**: Yes
- **Parallelization Notes**: No dependency on any other task. Safe to run in parallel with T001 and T002.
- **Date**: 2026-04-15
- **Owner**: TBD
- **Feature Folder**: docs/specs/features/001-admin-identity-domain/
- **Feature Spec**: docs/specs/features/001-admin-identity-domain/feature.spec.md
- **Feature Plan**: docs/specs/features/001-admin-identity-domain/plan.spec.md
- **Task File**: docs/specs/features/001-admin-identity-domain/tasks/T000-rename-test-projects.task.spec.md
- **Related ADRs**: N/A
- **Dependencies**: N/A

## 1. Purpose

Rename the existing test project directories from `*-e2e` to `*-tests` to establish the standard naming convention before any new test infrastructure is added. This prevents reference churn in later tasks.

## 2. Scope

### In Scope

- Rename `apps/admin/backend-e2e/` → `apps/admin/backend-tests/`
- Rename `apps/admin/frontend-e2e/` → `apps/admin/frontend-tests/`
- Update all internal references in project configs, Jest configs, package.json files, tsconfig files
- Update root `tsconfig.json` project references
- Regenerate `pnpm-lock.yaml` via `pnpm install`

### Out of Scope

- Changing test content or test setup logic
- Adding new test infrastructure (database, Redis)
- Modifying Playwright config beyond path references

## 3. Context from Feature Plan

- **Plan slice**: Slice 0 — Rename Test Projects Convention
- **Requirement refs**: N/A (convention alignment)
- **Affected Paths**:
  - `apps/admin/backend-e2e/` → `apps/admin/backend-tests/`
  - `apps/admin/frontend-e2e/` → `apps/admin/frontend-tests/`
  - `apps/admin/backend-tests/project.json`
  - `apps/admin/backend-tests/jest.config.cts`
  - `apps/admin/backend-tests/package.json`
  - `apps/admin/backend-tests/tsconfig.json`
  - `apps/admin/frontend-tests/package.json`
  - `apps/admin/frontend-tests/playwright.config.ts`
  - `tsconfig.json` (root)
  - `pnpm-lock.yaml`
- **Why this task exists**: Establishes the `*-tests` naming convention before new test projects are created. Avoids inconsistency and reference churn in later tasks.

## 4. Technical Specification (Required)

### 4.1 Backend and API Contracts (when applicable)

N/A — no API changes.

### 4.2 Database Specification (mandatory when data impact exists)

N/A — no data impact.

### 4.3 Frontend and UX Contracts (when applicable)

N/A — no frontend changes.

### 4.4 Cross-Cutting Constraints

- **Compatibility constraints**: All existing Nx targets (`e2e`, `build`, `serve`) must continue to work after renaming. Verify via `pnpm nx run backend-tests:e2e` and `pnpm nx run frontend-tests:e2e`.

## 5. Implementation Steps

1. **Rename directories** using `git mv`:
   - `git mv apps/admin/backend-e2e apps/admin/backend-tests`
   - `git mv apps/admin/frontend-e2e apps/admin/frontend-tests`

2. **Update `apps/admin/backend-tests/project.json`**:
   - Change `"name": "backend-e2e"` → `"name": "backend-tests"`
   - Change `"jestConfig": "apps/admin/backend-e2e/jest.config.cts"` → `"jestConfig": "apps/admin/backend-tests/jest.config.cts"`
   - Change `"outputs"` path from `backend-e2e` → `backend-tests` if present

3. **Update `apps/admin/backend-tests/jest.config.cts`**:
   - Change `displayName: "backend-e2e"` → `displayName: "backend-tests"`

4. **Update `apps/admin/backend-tests/package.json`**:
   - Change `"name": "backend-e2e"` → `"name": "backend-tests"`

5. **Update `apps/admin/backend-tests/tsconfig.json`**:
   - Change `"outDir": "out-tsc/backend-e2e"` → `"outDir": "out-tsc/backend-tests"`

6. **Update `apps/admin/frontend-tests/package.json`**:
   - Change `"name": "frontend-e2e"` → `"name": "frontend-tests"`

7. **Update `apps/admin/frontend-tests/playwright.config.ts`** (if it contains path references to `frontend-e2e`, update them to `frontend-tests`).

8. **Update root `tsconfig.json`**:
   - Change `"path": "./apps/admin/backend-e2e"` → `"path": "./apps/admin/backend-tests"`
   - Change `"path": "./apps/admin/frontend-e2e"` → `"path": "./apps/admin/frontend-tests"`

9. **Run `pnpm install`** to regenerate `pnpm-lock.yaml` with updated package names.

10. **Verify**: Run `pnpm nx show projects` to confirm renamed projects appear. Run `pnpm nx run backend-tests:e2e` (expect pass with no tests or existing tests passing).

## 6. Acceptance Criteria

- AC1: `apps/admin/backend-e2e/` no longer exists; `apps/admin/backend-tests/` exists with all original files.
- AC2: `apps/admin/frontend-e2e/` no longer exists; `apps/admin/frontend-tests/` exists with all original files.
- AC3: `pnpm nx show projects` lists `backend-tests` and `frontend-tests` (not `backend-e2e` or `frontend-e2e`).
- AC4: `pnpm install` completes without errors.
- AC5: Root `tsconfig.json` references the new paths.

## 7. Test Scenarios

- **Scenario 1 (happy path)**: After renaming, `pnpm nx run backend-tests:e2e` executes successfully (passWithNoTests or existing tests pass).
- **Scenario 2 (happy path)**: `pnpm nx graph` shows `backend-tests` depends on `backend` (implicit dependency preserved).
- **Scenario 3 (edge case)**: No stale references to `backend-e2e` or `frontend-e2e` remain in any config file. Verify with `grep -r "backend-e2e\|frontend-e2e" apps/ tsconfig.json`.

## 8. Definition of Ready (to start Step 4)

- [x] Status is `Draft` (will be `Ready` after approval).
- [x] Scope is explicit and bounded.
- [x] Required contracts are defined — N/A (infrastructure only).
- [x] Technical specification is detailed enough for independent implementation.
- [x] Acceptance criteria are testable.
- [x] Open questions are resolved.
- [x] Dependencies are completed or clearly sequenced — no dependencies.

## 9. Definition of Done

- [x] Status moved to `Done`.
- [x] Acceptance criteria validated.
- [x] Edge cases covered (no stale references).
- [x] Links to changed files/PR/tests are registered.
- [x] Feature spec and plan traceability remains intact.

## 10. Jira Mapping (Optional)

- **Issue Type**: Task
- **Summary**: Rename test projects from *-e2e to *-tests convention
- **Description**: Rename backend-e2e → backend-tests and frontend-e2e → frontend-tests with all config updates.
- **Priority**: High
- **Labels**: Infrastructure
- **Custom field (Tipo)**: Technical Debt
- **Custom field (Tamanho)**: 1 Dia

## 11. Status Log

| Date       | Status | Notes |
| ---------- | ------ | ----- |
| 2026-04-15 | Draft  | Task created from approved feature plan |
| 2026-04-15 | Ready  | Task approved for implementation |
| 2026-04-15 | In Progress | Implementation started |
| 2026-04-15 | Done | Directories renamed, all configs updated, Nx graph verified |
