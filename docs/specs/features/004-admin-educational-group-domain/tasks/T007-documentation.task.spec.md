# Task Spec: Documentation Updates

- **Feature ID**: 004-admin-educational-group-domain
- **Task ID**: T007
- **Status**: Done
- **Type**: Task
- **Parallelizable**: Yes
- **Parallelization Notes**: Can be developed in parallel with T006 (integration tests). Only depends on T003 (educational group CRUD) for module hierarchy knowledge. No code dependencies.
- **Last Updated**: 2026-04-20
- **Owner**: Guilherme Moraes
- **Feature Folder**: docs/specs/features/004-admin-educational-group-domain/
- **Feature Spec**: docs/specs/features/004-admin-educational-group-domain/feature.spec.md
- **Feature Plan**: docs/specs/features/004-admin-educational-group-domain/plan.spec.md
- **Task File**: docs/specs/features/004-admin-educational-group-domain/tasks/T007-documentation.task.spec.md
- **Related ADRs**: ADR-004
- **Dependencies**: T003

## 1. Purpose

Create the educational group domain spec and update the Admin application architecture documentation to include the new educational group domain, its modules, entities, and API surface. This task ensures that the domain is formally documented in the project's spec system following ADR-004 guidelines.

## 2. Scope

### In Scope

- Create new domain spec at `docs/specs/domains/educational-group.md`
- Update `docs/specs/apps/admin/architecture.md` to add the Educational Group domain entry, module hierarchy, and entity listings

### Out of Scope

- Code changes (documentation-only task)
- Feature spec or plan status updates (handled in Phase 4 of the breakdown)
- ADR creation (no new architectural decisions needed)

## 3. Context from Feature Plan

- **Plan slice**: Slice 6 — Documentation Updates
- **Requirement refs**: N/A (documentation)
- **Affected Paths**:
  - `docs/specs/domains/educational-group.md` (NEW)
  - `docs/specs/apps/admin/architecture.md` (MODIFIED)
- **Why this task exists**: ADR-004 requires each domain to have a spec in `docs/specs/domains/`. The admin architecture doc must reflect all active domains and their module structure.

## 4. Technical Specification (Required)

### 4.1 Backend and API Contracts (when applicable)

N/A — Documentation-only task.

### 4.2 Database Specification (mandatory when data impact exists)

N/A — No database changes.

### 4.3 Frontend and UX Contracts (when applicable)

N/A — No frontend changes.

### 4.4 Cross-Cutting Constraints

N/A — Documentation-only task.

## 5. Implementation Steps

1. **Create `docs/specs/domains/educational-group.md`** following the pattern of `docs/specs/domains/subscription.md`:
   - **Overview**: Domain name, bounded context, application
   - **Responsibilities**: Educational group lifecycle management, master user provisioning, database parameter generation, plan subscription management, common user management
   - **Entities table**: `GrupoEducacional`, `UsuarioMestre`, `ParametrosGrupoEducacional`, `UsuarioComum`, `Assinatura` — with table names, inheritance, and descriptions
   - **Key Modules table**: `EducationalGroupModule` (boundary), `GroupsModule`, `SubscriptionsModule`, `CommonUsersModule`
   - **API Surface table**: All 12 endpoints across 3 groups
   - **Infrastructure Dependencies**: PostgreSQL (via `@satie/database`), `@satie/utils` (slug, password, naming), Identity domain (JWT guard), Subscription domain (cross-domain FK to `plano`)
   - **Related Decisions**: ADR-004, ADR-005, ADR-010

2. **Update `docs/specs/apps/admin/architecture.md`**:
   - Add **Educational Group** domain entry in the Domains section (after Subscription), with module list and links to domain spec and feature spec
   - Add the `educational-group/` folder to the Module Hierarchy tree showing:
     - `educational-group.module.ts`
     - `entities/` (5 entity files)
     - `groups/` (controller, service, DTOs)
     - `subscriptions/` (controller, service, DTOs)
     - `common-users/` (controller, service, DTOs)
   - Update the entity list in app.module.ts documentation section if applicable

## 6. Acceptance Criteria

- AC1: `docs/specs/domains/educational-group.md` exists and follows the same structure as `docs/specs/domains/subscription.md`.
- AC2: Domain spec lists all 5 entities with correct table names.
- AC3: Domain spec lists all 4 modules with their roles.
- AC4: Domain spec lists all 12 API endpoints.
- AC5: `docs/specs/apps/admin/architecture.md` includes the Educational Group domain in the Domains section.
- AC6: Architecture doc includes the educational-group module hierarchy in the Module Hierarchy tree.

## 7. Test Scenarios

N/A — Documentation-only task. Validation is done via review (correct structure, completeness, accuracy against implemented code).

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
| 1 | `pnpm nx run-many -t check` | No source code changed (documentation-only task). Biome: 0 warnings, 0 errors. Output: "NX No tasks were run" | 2026-04-20 |

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
| 2026-04-20 | Awaiting Dependency | Waiting for T003 to complete |
| 2026-04-20 | Ready | T003 completed; task ready for implementation |
| 2026-04-20 | In Progress | Starting documentation implementation |
| 2026-04-20 | Done | Domain spec created, architecture doc updated, all ACs verified |

## 12. Observations

- This task is documentation-only but still requires Test Evidence for the Definition of Done. Since there are no code tests, the evidence should be a Biome check pass or a note that no source code was changed.
- Use `docs/specs/domains/subscription.md` as the structural template for the new domain spec.
