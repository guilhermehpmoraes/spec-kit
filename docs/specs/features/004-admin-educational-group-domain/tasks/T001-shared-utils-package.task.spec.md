# Task Spec: Shared Utilities Package (@satie/utils)

- **Feature ID**: 004-admin-educational-group-domain
- **Task ID**: T001
- **Status**: Done
- **Type**: Task
- **Parallelizable**: Yes
- **Parallelization Notes**: No dependency on T002 (database entities). Both are foundational tasks that can be developed in parallel.
- **Date**: 2026-04-20
- **Owner**: Guilherme Moraes
- **Feature Folder**: docs/specs/features/004-admin-educational-group-domain/
- **Feature Spec**: docs/specs/features/004-admin-educational-group-domain/feature.spec.md
- **Feature Plan**: docs/specs/features/004-admin-educational-group-domain/plan.spec.md
- **Task File**: docs/specs/features/004-admin-educational-group-domain/tasks/T001-shared-utils-package.task.spec.md
- **Related ADRs**: ADR-005, ADR-008
- **Dependencies**: N/A

## 1. Purpose

Create a new shared workspace package `@satie/utils` at `packages/utils/` containing reusable utility functions needed by the educational group domain: slug generation, random password generation, and environment-based naming. This package will also include a companion test project `packages/utils-tests/` and unit tests for all exported functions.

These utilities are extracted into a shared package because they are general-purpose functions that may be reused by other domains and applications in the future.

## 2. Scope

### In Scope

- Create `packages/utils/` Nx library package with proper `package.json`, `project.json`, `tsconfig.json`, `tsconfig.lib.json`
- Implement `generateSlug(descricao: string): string` — Unicode normalization, diacritics stripping, kebab-case conversion
- Implement `generateRandomPassword(length?: number): string` — cryptographically secure random password with character class guarantees
- Implement `generateMasterUsername(slug: string, env: string): string` — format `mestre-{env}-{slug}`
- Implement `generateDatabaseName(slug: string, env: string): string` — format `db_{env}_{slug}`
- Add `@satie/utils` path mapping to `tsconfig.base.json`
- Create `packages/utils-tests/` test project with unit tests for all functions
- Export all functions from `packages/utils/src/index.ts`

### Out of Scope

- Integration with NestJS modules or services (consumed in T003)
- Any backend or frontend code changes
- Environment variable reading (consumers pass `env` as a string parameter)

## 3. Context from Feature Plan

- **Plan slice**: Slice 0 — Shared Utilities Package
- **Requirement refs**: FR-002 (master username format), FR-003 (database name format), FR-004 (random password policy)
- **Affected Paths**:
  - `packages/utils/` (NEW)
  - `packages/utils-tests/` (NEW)
  - `tsconfig.base.json` (MODIFIED — add path mapping)
- **Why this task exists**: The utility functions (slug, password, naming) are needed by the educational group service (T003) for atomic group creation. Extracting them into a shared package follows the monorepo pattern established by `@satie/database`.

## 4. Technical Specification (Required)

### 4.1 Backend and API Contracts (when applicable)

N/A — This task creates a shared utility package with pure functions, no API endpoints.

### 4.2 Database Specification (mandatory when data impact exists)

N/A — No database changes in this task.

### 4.3 Frontend and UX Contracts (when applicable)

N/A — Backend-only feature.

### 4.4 Cross-Cutting Constraints

- **Security/authorization**: `generateRandomPassword` MUST use `crypto.randomBytes` or `crypto.randomInt` for randomness. `Math.random()` is forbidden.
- **Performance expectations**: All functions are synchronous and O(n) where n is the input string length. No performance concerns.
- **Observability**: N/A — pure functions, no logging needed.
- **Compatibility constraints**: Package must be compatible with Node.js 18+ and TypeScript 5.x.

## 5. Implementation Steps

1. **Create package structure** at `packages/utils/`:
   - `packages/utils/package.json` — name `@satie/utils`, main entry `./src/index.ts`
   - `packages/utils/project.json` — Nx project config (follow `packages/database/project.json` pattern)
   - `packages/utils/tsconfig.json` — extend `../../tsconfig.base.json`
   - `packages/utils/tsconfig.lib.json` — lib-specific config (follow `packages/database/tsconfig.lib.json` pattern)
   - `packages/utils/src/index.ts` — re-export all public functions

2. **Implement `packages/utils/src/slug.ts`**:
   ```
   export function generateSlug(descricao: string): string
   ```
   - Normalize Unicode (NFD)
   - Strip combining diacritical marks (regex: `/[\u0300-\u036f]/g`)
   - Lowercase
   - Replace spaces and non-alphanumeric chars with hyphens
   - Collapse consecutive hyphens
   - Trim leading/trailing hyphens
   - Example: `"Colégio São Paulo CCCL"` → `"colegio-sao-paulo-cccl"`

3. **Implement `packages/utils/src/password.ts`**:
   ```
   export function generateRandomPassword(length: number = 16): string
   ```
   - Character pools: uppercase `A-Z`, lowercase `a-z`, digits `0-9`, symbols `!@#$%^&*()-_=+[]{}|;:,.<>?`
   - Guarantee at least 1 character from each pool
   - Fill remaining positions with random characters from all pools combined
   - Shuffle the final result using Fisher-Yates with `crypto.randomInt`
   - Use `crypto.randomInt` for all random selections

4. **Implement `packages/utils/src/naming.ts`**:
   ```
   export function generateMasterUsername(slug: string, env: string): string
   ```
   - Returns `mestre-${env}-${slug}`

   ```
   export function generateDatabaseName(slug: string, env: string): string
   ```
   - Returns `db_${env}_${slug}`

5. **Update `tsconfig.base.json`** — add path mapping:
   ```json
   "@satie/utils": ["packages/utils/src/index.ts"]
   ```

6. **Create test project** at `packages/utils-tests/`:
   - `packages/utils-tests/package.json` — test project config
   - `packages/utils-tests/project.json` — Nx config with `implicitDependencies: ["utils"]`
   - `packages/utils-tests/tsconfig.json` — extend base, follow `packages/database-tests/` pattern
   - `packages/utils-tests/jest.config.js` — Jest config following existing test project patterns

7. **Write unit tests** at `packages/utils-tests/src/`:
   - `slug.spec.ts` — test `generateSlug`
   - `password.spec.ts` — test `generateRandomPassword`
   - `naming.spec.ts` — test `generateMasterUsername` and `generateDatabaseName`

## 6. Acceptance Criteria

- AC1: `generateSlug("Colégio São Paulo CCCL")` returns `"colegio-sao-paulo-cccl"`.
- AC2: `generateSlug` strips accents, handles multiple spaces, leading/trailing whitespace, and special characters.
- AC3: `generateRandomPassword()` returns a 16-character string containing at least 1 uppercase, 1 lowercase, 1 digit, and 1 symbol.
- AC4: `generateRandomPassword(length)` respects custom length parameter (minimum 4 to guarantee one from each pool).
- AC5: `generateMasterUsername("colegio-cccl", "dev")` returns `"mestre-dev-colegio-cccl"`.
- AC6: `generateMasterUsername("colegio-cccl", "prod")` returns `"mestre-prod-colegio-cccl"`.
- AC7: `generateDatabaseName("colegio-cccl", "dev")` returns `"db_dev_colegio-cccl"`.
- AC8: `generateDatabaseName("colegio-cccl", "prod")` returns `"db_prod_colegio-cccl"`.
- AC9: `@satie/utils` is importable from other workspace packages via the `tsconfig.base.json` path mapping.
- AC10: All unit tests pass.

## 7. Test Scenarios

### Slug Generation

- **Happy path**: `"Colégio São Paulo CCCL"` → `"colegio-sao-paulo-cccl"`
- **Accents**: `"São José dos Campos"` → `"sao-jose-dos-campos"`
- **Multiple spaces**: `"Escola   Municipal"` → `"escola-municipal"`
- **Special characters**: `"Escola @#$% Teste!"` → `"escola-teste"`
- **Leading/trailing spaces**: `"  Escola Teste  "` → `"escola-teste"`
- **Already kebab-case**: `"escola-teste"` → `"escola-teste"`
- **Single word**: `"Escola"` → `"escola"`
- **Empty string**: `""` → `""`

### Password Generation

- **Default length**: Returns exactly 16 characters
- **Custom length**: `generateRandomPassword(20)` returns exactly 20 characters
- **Character class guarantees**: Password contains at least 1 uppercase, 1 lowercase, 1 digit, 1 symbol (run 100 iterations to verify)
- **Randomness**: Two consecutive calls return different passwords
- **Minimum length**: `generateRandomPassword(4)` still satisfies all character class requirements

### Naming Functions

- **Master username dev**: `generateMasterUsername("colegio-cccl", "dev")` → `"mestre-dev-colegio-cccl"`
- **Master username prod**: `generateMasterUsername("colegio-cccl", "prod")` → `"mestre-prod-colegio-cccl"`
- **Database name dev**: `generateDatabaseName("colegio-cccl", "dev")` → `"db_dev_colegio-cccl"`
- **Database name prod**: `generateDatabaseName("colegio-cccl", "prod")` → `"db_prod_colegio-cccl"`

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
| 1 | `pnpm nx test utils-tests` | 3 suites passed, 17 tests passed, 0 failures | 2026-04-20 |
| 2 | `pnpm biome check packages/utils/ packages/utils-tests/` | Checked 15 files. No fixes applied. 0 warnings, 0 errors | 2026-04-20 |

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
| 2026-04-20 | Ready  | No dependencies; ready for implementation |
| 2026-04-20 | In Progress | Implementation started — task branch task/T001-shared-utils-package created |
| 2026-04-20 | Done | All code implemented, 17/17 tests pass, Biome clean |

## 12. Observations

- Follow `packages/database/` and `packages/database-tests/` as reference patterns for package structure, Nx config, and test project setup.
- The `@satie/utils` package has no runtime dependencies — only Node.js built-in `crypto` module.
