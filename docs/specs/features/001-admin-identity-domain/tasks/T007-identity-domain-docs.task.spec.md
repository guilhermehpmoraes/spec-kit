# Task Spec: Identity Domain Documentation

- **Feature ID**: 001-admin-identity-domain
- **Task ID**: T007
- **Status**: Done
- **Type**: Task
- **Parallelizable**: Yes
- **Parallelization Notes**: Can be started in parallel with T004+ (after T002 is complete, as it references the shared package structure). Documentation does not depend on code being finished.
- **Date**: 2026-04-15
- **Last Updated**: 2026-04-15
- **Owner**: TBD
- **Feature Folder**: docs/specs/features/001-admin-identity-domain/
- **Feature Spec**: docs/specs/features/001-admin-identity-domain/feature.spec.md
- **Feature Plan**: docs/specs/features/001-admin-identity-domain/plan.spec.md
- **Task File**: docs/specs/features/001-admin-identity-domain/tasks/T007-identity-domain-docs.task.spec.md
- **Related ADRs**: ADR-004, ADR-005
- **Dependencies**: T002 (shared package structure must be known)

## 1. Purpose

Create the identity domain spec and update the Admin application architecture documentation to reflect the implemented module structure, dependencies, and conventions established by this feature.

## 2. Scope

### In Scope

- Create `docs/specs/domains/identity.md` — domain spec for the identity bounded context
- Update `docs/specs/apps/admin/architecture.md` — reflect implemented module structure, dependencies, infrastructure

### Out of Scope

- Production code changes
- Test changes
- Any other documentation files

## 3. Context from Feature Plan

- **Plan slice**: Slice 7 — Documentation Updates
- **Requirement refs**: N/A (documentation requirements from planning)
- **Affected Paths**:
  - `docs/specs/domains/identity.md` (new)
  - `docs/specs/apps/admin/architecture.md` (modify)
- **Why this task exists**: ADR-004 requires domain specs to be documented before or alongside the first feature that touches them. The admin architecture doc needs to reflect the actually implemented structure.

## 4. Technical Specification (Required)

### 4.1 Backend and API Contracts (when applicable)

N/A — documentation only.

### 4.2 Database Specification (mandatory when data impact exists)

N/A — documentation only.

### 4.3 Frontend and UX Contracts (when applicable)

N/A.

### 4.4 Cross-Cutting Constraints

N/A.

## 5. Implementation Steps

1. **Create `docs/specs/domains/identity.md`** with the following content:
   - **Domain name**: Identity
   - **Bounded context**: Admin user management and authentication
   - **Application**: Admin
   - **Responsibilities**:
     - Admin user lifecycle (CRUD with soft delete)
     - Password management (hashing, policy enforcement)
     - JWT-based authentication (login, refresh, logout)
     - Token revocation via Redis blacklist
   - **Entities**: `UsuarioAdmin` (inherits `EntidadeBase`)
   - **Key modules**: `IdentityModule` (boundary), `UsersModule`, `AuthModule`, `TokenRevocationModule`
   - **API surface**: `/auth/*` (login, refresh, logout), `/usuarios-admin/*` (CRUD)
   - **Infrastructure dependencies**: PostgreSQL (via `@satie/database`), Redis (via `@satie/database` RedisModule)
   - **Related decisions**: ADR-002 (patterns), ADR-004 (DDD baseline), ADR-005 (database conventions)
   - **Related features**: 001-admin-identity-domain
   - **Future considerations**: Role-based access, password recovery, MFA (all out of scope for now)

2. **Update `docs/specs/apps/admin/architecture.md`**:
   - Update the **Domains** section: flesh out the Identity domain entry with implemented details
   - Update the **Backend Structure** section with the actual module hierarchy:
     ```
     src/
     ├── main.ts
     ├── app/
     │   ├── app.module.ts
     │   ├── app.controller.ts
     │   └── app.service.ts
     ├── identity/
     │   ├── identity.module.ts
     │   ├── auth/
     │   │   ├── auth.module.ts
     │   │   ├── auth.controller.ts
     │   │   ├── auth.service.ts
     │   │   ├── dto/ (login, refresh-token, logout)
     │   │   ├── strategies/ (jwt.strategy.ts)
     │   │   └── guards/ (jwt-auth.guard.ts)
     │   ├── users/
     │   │   ├── users.module.ts
     │   │   ├── users.controller.ts
     │   │   ├── users.service.ts
     │   │   ├── dto/ (create-user, update-user, user-response)
     │   │   └── entities/ (usuario-admin.entity.ts)
     │   └── token-revocation/
     │       ├── token-revocation.module.ts
     │       └── token-revocation.service.ts
     ├── migrations/
     │   ├── <timestamp>-CreateUsuarioAdmin.ts
     │   └── <timestamp>-SeedSuperAdmin.ts
     └── config/
         └── typeorm.config.ts
     ```
   - Update **Infrastructure Dependencies**: document Docker Compose, PostgreSQL 17.4, Redis
   - Update **Shared Packages**: confirm `@satie/database` usage with specific exports used

## 6. Acceptance Criteria

- AC1: `docs/specs/domains/identity.md` exists with all content listed in step 1.
- AC2: `docs/specs/apps/admin/architecture.md` reflects the implemented module structure.
- AC3: Both documents reference the correct ADRs (ADR-002, ADR-004, ADR-005).
- AC4: The domain spec is linked from the feature spec's `Domain` field.

## 7. Test Scenarios

- **Scenario 1**: Review `docs/specs/domains/identity.md` — all sections present, no placeholders.
- **Scenario 2**: Review `docs/specs/apps/admin/architecture.md` — module tree matches actual implementation.
- **Scenario 3**: Links between domain spec ↔ feature spec ↔ architecture doc are correct and navigable.

## 8. Definition of Ready (to start Step 4)

- [x] Status is `Draft` (will be `Ready` after approval).
- [x] Scope is explicit and bounded.
- [x] Required contracts are defined — N/A (documentation).
- [x] Technical specification is detailed enough for independent implementation.
- [x] Acceptance criteria are testable.
- [x] Open questions are resolved.
- [x] Dependencies are completed or clearly sequenced — T002 provides the shared package structure to document.

## 9. Definition of Done

- [x] Status moved to `Done`.
- [x] Acceptance criteria validated.
- [x] Links to changed files registered.
- [x] Feature spec and plan traceability remains intact.

## 10. Jira Mapping (Optional)

- **Issue Type**: Task
- **Summary**: Identity domain documentation
- **Description**: Create identity domain spec and update admin architecture documentation.
- **Priority**: Medium
- **Labels**: Documentation
- **Custom field (Tipo)**: Feature
- **Custom field (Tamanho)**: 1 Dia

## 11. Status Log

| Date       | Status | Notes |
| ---------- | ------ | ----- |
| 2026-04-15 | Draft  | Task created from approved feature plan |
| 2026-04-15 | Ready  | Task approved for implementation |
| 2026-04-15 | In Progress | Implementation started |
| 2026-04-15 | Done | Identity domain spec created, admin architecture doc updated, feature spec Domain field linked |
