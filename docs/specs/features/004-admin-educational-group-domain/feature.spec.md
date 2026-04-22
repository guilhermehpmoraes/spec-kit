# Feature Spec: Admin Educational Group Domain

- **Feature ID**: 004-admin-educational-group-domain
- **Status**: Archived
- **Created**: 2026-04-20
- **Last Updated**: 2026-04-20
- **Owner**: Guilherme Moraes
- **Domain**: educational-group
- **Application**: admin
- **Source Inputs**: User requirement — educational group management module with master user, common users, database parameters, and plan subscriptions
- **Feature Folder**: docs/specs/features/004-admin-educational-group-domain/
- **Feature File**: docs/specs/features/004-admin-educational-group-domain/feature.spec.md
- **Plan File**: docs/specs/features/004-admin-educational-group-domain/plan.spec.md
- **Task Folder**: docs/specs/features/004-admin-educational-group-domain/tasks/
- **Related ADRs**: ADR-004, ADR-005

## 1. Context

The Admin platform needs to manage **educational groups** (grupos educacionais) — the client/tenant entities that represent schools or educational institutions using the platform. Each educational group has its own database parameters, a master user for administrative access, common users, and plan subscriptions linking it to available products.

Currently there is no module for onboarding and managing these educational groups. This feature introduces the full lifecycle for educational groups, including automatic provisioning of master user credentials, database naming, and plan association at creation time.

## 2. Scope

### In Scope

- CRUD operations for `grupo_educacional` with automatic provisioning of `usuario_mestre` and `parametros_grupo_educacional`
- Automatic generation of master user username and database name based on the group description and environment (`dev`/`prod`)
- Random secure password generation for master users (uppercase + lowercase + number + symbol, minimum one of each)
- `assinatura` management (link/unlink plans to groups, with 1 plan per product constraint)
- `usuario_comum` CRUD (common users within a group, with email)
- Three separate endpoint groups: educational groups, subscriptions, and common users
- JWT-protected endpoints (reuse Identity domain's auth guard)
- All entities inherit from `EntidadeBase` (audit fields + soft delete)

### Out of Scope

- Email verification flow for `usuario_comum` (`verificado` field stored but not used in this feature)
- Actual database provisioning (creating the physical database referenced by `nome_banco`)
- Master user authentication (master users are credentials managed by the system, not used for login in this feature)
- Frontend UI for educational group management
- Bulk import of educational groups or users

## 3. User Scenarios (Prioritized)

### Scenario 1 - Create Educational Group (P1)

An admin user creates a new educational group by providing a description, an email for the first common user, and selecting one or more plans. The system automatically generates the master user credentials and database name based on the description and environment.

#### Acceptance Scenarios

1. **Given** an authenticated admin user, **When** they POST to create a grupo educacional with description "Colegio CCCL", email "admin@cccl.com", and plan IDs, **Then** the system creates the grupo_educacional, parametros_grupo_educacional (nome_banco = "db_dev_colegio-cccl"), usuario_mestre (usuario = "mestre-dev-colegio-cccl", random password), usuario_comum (with email, verificado = false), and assinatura records, all in a single transaction.
2. **Given** an authenticated admin user, **When** they POST to create a grupo educacional with plans from the same product, **Then** the system rejects the request with a validation error (only 1 plan per product allowed).
3. **Given** environment variable set to `prod`, **When** a grupo educacional is created with description "Colegio CCCL", **Then** nome_banco = "db_prod_colegio-cccl" and usuario = "mestre-prod-colegio-cccl".
4. **Given** an authenticated admin user, **When** they POST to create a grupo educacional with a random password generated, **Then** the password contains at least one uppercase letter, one lowercase letter, one number, and one symbol.

### Scenario 2 - Edit Educational Group (P1)

An admin user edits an existing educational group. Only the description and email can be updated — master user credentials and database name are immutable after creation.

#### Acceptance Scenarios

1. **Given** an existing grupo educacional, **When** an admin PATCHes with a new description, **Then** only the description is updated; usuario_mestre and nome_banco remain unchanged.
2. **Given** an existing grupo educacional, **When** an admin attempts to update usuario_mestre or nome_banco fields, **Then** the system ignores those fields (they are not in the update DTO).

### Scenario 3 - Manage Subscriptions Independently (P1)

An admin user manages the plan associations (assinaturas) of a group without modifying the group itself.

#### Acceptance Scenarios

1. **Given** an existing grupo educacional with 2 plans, **When** an admin PATCHes the assinatura endpoint to replace with 3 different plans, **Then** old assinatura records are removed and new ones are created.
2. **Given** an admin updating assinaturas, **When** they attempt to assign two plans of the same product, **Then** the system rejects with a validation error.

### Scenario 4 - Manage Common Users Independently (P1)

An admin user manages common users (usuario_comum) within a group without modifying the group itself.

#### Acceptance Scenarios

1. **Given** an existing grupo educacional, **When** an admin creates a new usuario_comum with an email, **Then** the user is created with verificado = false and linked to the group.
2. **Given** an existing usuario_comum, **When** an admin updates the email or password, **Then** only those fields are updated.
3. **Given** an existing usuario_comum, **When** an admin deletes the user, **Then** the user is soft-deleted.

### Scenario 5 - List and View Educational Groups (P2)

An admin user lists all educational groups and views details of a specific group.

#### Acceptance Scenarios

1. **Given** multiple educational groups exist, **When** an admin GETs the list endpoint, **Then** all non-deleted groups are returned with their descriptions.
2. **Given** an existing grupo educacional, **When** an admin GETs by ID, **Then** the response includes the group details, parametros, usuario_mestre (without password), assinaturas with plan details, and usuario_comum list.

### Scenario 6 - Delete Educational Group (P2)

An admin user soft-deletes an educational group.

#### Acceptance Scenarios

1. **Given** an existing grupo educacional, **When** an admin DELETEs it, **Then** the group and its related records (parametros, usuario_mestre, assinaturas, usuario_comum) are soft-deleted.

## 4. Functional Requirements

- **FR-001**: System MUST create `grupo_educacional`, `parametros_grupo_educacional`, `usuario_mestre`, first `usuario_comum`, and `assinatura` records atomically in a single database transaction.
- **FR-002**: System MUST auto-generate `usuario_mestre.usuario` in the format `mestre-{env}-{slug}` where `{env}` comes from an environment variable and `{slug}` is the kebab-case, normalized, lowercased version of the description.
- **FR-003**: System MUST auto-generate `parametros_grupo_educacional.nome_banco` in the format `db_{env}_{slug}` where `{env}` and `{slug}` follow the same rules as FR-002.
- **FR-004**: System MUST generate a random password for `usuario_mestre.senha` with at least one uppercase letter, one lowercase letter, one digit, and one special character.
- **FR-005**: System MUST NOT allow editing `usuario_mestre.usuario` or `parametros_grupo_educacional.nome_banco` after creation.
- **FR-006**: System MUST enforce a unique constraint: only 1 plan per product per grupo educacional (validated at application level via the `plano.idProduto` relationship).
- **FR-007**: System MUST expose three separate endpoint groups: `/grupos-educacionais`, `/assinaturas`, `/usuarios-comuns`.
- **FR-008**: All endpoints MUST be protected by the existing JWT authentication guard from the Identity domain.
- **FR-009**: All entities MUST inherit from `EntidadeBase` and support soft delete.
- **FR-010**: System MUST hash `usuario_mestre.senha` before persisting (same bcrypt approach used by Identity domain).
- **FR-011**: The first `usuario_comum` MUST be created during grupo educacional creation using the provided email, with `verificado` = false.

## 5. Engineering Quality Constraints

- **EQ-001 (KISS)**: The implementation MUST prefer the simplest design that satisfies all acceptance scenarios.
- **EQ-002 (Self-descriptive code)**: New code MUST be understandable through naming and structure without relying on comments.
- **EQ-003 (Comments policy)**: Comments MUST be added only for non-obvious intent, tradeoffs, or constraints.
- **EQ-004 (Testability)**: Boundaries and responsibilities MUST enable focused unit and integration tests.
- **EQ-005 (Patterns baseline)**: Design choices MUST follow current ADR baseline unless explicitly justified.

## 6. Dependencies and Risks

### Dependencies

- **Identity domain**: Reuse `JwtAuthGuard` for endpoint protection.
- **Subscription domain**: FK reference to `plano` table for `assinatura` entity; need to query `plano.idProduto` for the 1-plan-per-product validation.
- **`@satie/database`**: `EntidadeBase`, TypeORM config, `SnakeNamingStrategy`.
- **Environment variable**: A configuration variable (e.g., `ENVIRONMENT` or `NODE_ENV`) to determine `dev`/`prod` prefix for username and database name generation.

### Risks

- **Cross-domain FK to plano**: The assinatura entity references the Subscription domain's `plano` table. This creates a domain boundary crossing at the DB level -> Mitigated by treating `assinatura` as the boundary table owned by the educational-group domain, with a read-only dependency on `plano`.
- **Slug collision**: Two groups with similar descriptions could generate the same username/database name -> Mitigate with unique constraints on `usuario_mestre.usuario` and `parametros_grupo_educacional.nome_banco`.

## 7. Success Criteria

- **SC-001**: An admin can create an educational group with all related entities provisioned in a single request.
- **SC-002**: Auto-generated master user credentials and database name follow the environment-based naming convention.
- **SC-003**: Plan subscriptions respect the 1-plan-per-product constraint.
- **SC-004**: Common users can be managed independently from the group.
- **SC-005**: All endpoints are protected and follow existing patterns (DTOs, validation, error handling, Scalar docs).

## 8. Validation Mapping

- **AC-S1-1** -> integration test -> Educational group creation endpoint (transaction + all related entities)
- **AC-S1-2** -> unit test -> Plan-per-product validation logic
- **AC-S1-3** -> unit test -> Slug/naming generation with environment variable
- **AC-S1-4** -> unit test -> Password generation policy
- **AC-S2-1** -> integration test -> Update endpoint (immutable fields)
- **AC-S3-1** -> integration test -> Assinatura replacement endpoint
- **AC-S3-2** -> unit test -> Plan-per-product validation on update
- **AC-S4-1** -> integration test -> Common user creation
- **AC-S5-1** -> integration test -> List endpoint
- **AC-S5-2** -> integration test -> Get-by-ID with relations
- **AC-S6-1** -> integration test -> Soft-delete cascading

- **EQ-001/EQ-005** -> Design review against ADRs
- **EQ-002/EQ-003** -> Code review checklist
- **EQ-004** -> Unit/integration test coverage

## 9. Planning Inputs for Step 2

### Affected Areas

- Backend: new `educational-group/` domain module under `apps/admin/backend/src/`
- Database: 5 new tables (`grupo_educacional`, `parametros_grupo_educacional`, `usuario_mestre`, `usuario_comum`, `assinatura`)
- Tests: new test files under `apps/admin/backend-tests/`
- Config: environment variable for `dev`/`prod` environment prefix

### Proposed Implementation Slices

- Slice 1: Database entities, migrations, and module setup
- Slice 2: Educational group CRUD (with atomic creation of related entities)
- Slice 3: Assinatura independent CRUD
- Slice 4: Common user independent CRUD
- Slice 5: Integration tests and validation

### Contracts and Constraints

- **API endpoints**:
  - `POST /grupos-educacionais` — create group (input: descricao, email, planoIds[])
  - `GET /grupos-educacionais` — list groups
  - `GET /grupos-educacionais/:id` — get group with details
  - `PATCH /grupos-educacionais/:id` — update group (descricao, email only)
  - `DELETE /grupos-educacionais/:id` — soft-delete group
  - `GET /assinaturas?idGrupoEducacional=X` — list subscriptions for a group
  - `PATCH /assinaturas/:idGrupoEducacional` — replace plan subscriptions
  - `POST /usuarios-comuns` — create common user (input: email, senha, idGrupoEducacional)
  - `GET /usuarios-comuns?idGrupoEducacional=X` — list common users for a group
  - `GET /usuarios-comuns/:id` — get common user
  - `PATCH /usuarios-comuns/:id` — update common user
  - `DELETE /usuarios-comuns/:id` — soft-delete common user

### Technical Detail Baseline (Step 2 Input)

- **Database changes**:
  - `usuario_mestre` — PK: id_usuario_mestre (UUID), columns: usuario (varchar 255, unique), senha (varchar 255). Inherits EntidadeBase audit fields.
  - `parametros_grupo_educacional` — PK: id_parametros_grupo_educacional (UUID), columns: nome_banco (varchar 255, unique). Inherits EntidadeBase.
  - `grupo_educacional` — PK: id_grupo_educacional (UUID), columns: descricao (varchar 255), id_usuario_mestre (UUID FK, unique), id_parametros_grupo_educacional (UUID FK, unique). Inherits EntidadeBase.
  - `usuario_comum` — PK: id_usuario_comum (UUID), columns: email (varchar 255), senha (varchar 255), verificado (boolean, default false), id_grupo_educacional (UUID FK). Inherits EntidadeBase.
  - `assinatura` — PK: id_assinatura (UUID), columns: id_grupo_educacional (UUID FK), id_plano (UUID FK). Unique constraint on (id_grupo_educacional, id_plano). Inherits EntidadeBase.
- **API contracts**: See Contracts section above.
- **Migration notes**: Forward migration creates all 5 tables. Rollback drops them in reverse FK order.

### Known Dependencies

- `plano` table from Subscription domain (FK from `assinatura`)
- `JwtAuthGuard` from Identity domain
- `@satie/database` package (EntidadeBase, TypeORM config)

### Open Questions to Resolve in Planning

- Q1: What is the exact environment variable name for dev/prod prefix? (Assumption: `APP_ENVIRONMENT` with values `dev` or `prod`)
- Q2: What is the minimum/maximum password length for `usuario_mestre`? (Assumption: 16 characters)
- Q3: Should the `usuario_mestre.senha` (plaintext) be returned in the creation response so the admin can share it, or only stored hashed? (Assumption: returned once on creation, then only stored hashed)
- Q4: Should `grupo_educacional` update also allow changing the email of the first `usuario_comum`? Or is that handled entirely via the `/usuarios-comuns` endpoint? (Assumption: handled via `/usuarios-comuns` endpoint only)
- Q5: Should slug generation strip accents/diacritics from the description? (Assumption: yes, normalize to ASCII)

## 10. Workflow and Status Gates

### Status Transition Rules

- On creation in Step 1, set **Status** to `Draft` and save the feature spec file.
- After user approval, update **Status** to `Approved`.
- When Step 2 starts, update **Status** to `In Planning`.
- After feature plan approval and task file generation, update **Status** to `Planned`.
- When implementation of the first task starts, update **Status** to `In Implementation`.
- After all scoped tasks are done and validated, update **Status** to `Done`.

### Gate to move Step 1 -> Step 2

- [x] Scope and requirements are approved.
- [x] Acceptance scenarios are testable.
- [x] Planning inputs are complete enough to derive tasks.

### Gate to move Step 2 -> Step 3

- [x] Feature plan is approved.
- [x] Task files are generated under `docs/specs/features/004-admin-educational-group-domain/tasks/`.
- [x] At least one task has status `Ready`.

## 11. Optional Data Impact

### Entities

- **GrupoEducacional**: Core tenant entity representing an educational group/school.
- **ParametrosGrupoEducacional**: Configuration parameters (database name) for a group.
- **UsuarioMestre**: Auto-provisioned master user credentials per group.
- **UsuarioComum**: Common user accounts within a group.
- **Assinatura**: Link between a group and subscription plans.

### Data Notes

- All entities use soft delete via `EntidadeBase.deletadoAs`.
- `usuario_mestre` and `parametros_grupo_educacional` have a 1:1 relationship with `grupo_educacional`.
- `usuario_comum` has a N:1 relationship with `grupo_educacional`.
- `assinatura` references `plano` from the Subscription domain (cross-domain FK).
- `nome_banco` and `usuario` (master) are unique and immutable after creation.
- Passwords for `usuario_mestre` are hashed with bcrypt before storage.

## 12. Fixed Learnings Input (SDD Evolution)

_To be completed after feature is done._

### Keep

-

### Avoid

-

### Promote to Standard

-

### Process Improvement for Next Cycle

## Retrospective

**Date**: 2026-04-20

### Key Findings

- Feature was delivered cleanly in a single day with no deviations from the plan
- All 7 tasks were self-sufficient — implementation required no extra discovery beyond what was in the specs
- 5-step flow worked as designed; no friction or rework reported
- The three-sub-module domain structure (`GroupsModule`, `SubscriptionsModule`, `CommonUsersModule` under `EducationalGroupModule`) proved clean and extensible — pattern worth repeating for future multi-entity domains
- The `@satie/utils` shared package established a reusable pattern for extracting pure utility functions into workspace packages (parallel to `@satie/database`)
- Atomic creation of 5 related entities in a single transaction worked correctly with TypeORM `DataSource.transaction()`
- 147 integration + unit tests passed before marking the feature Done

### Changes Applied

- `docs/specs/domains/educational-group.md` — Added "Business Rules" section documenting auto-provisioning naming format, password policy, immutability constraints, 1-plan-per-product rule, atomic creation, cascade soft-delete, and `APP_ENVIRONMENT` dependency
- `README.md` — Added `APP_ENVIRONMENT` env var to the environment variables table; updated migration step description to list all domain tables created (identity, subscription, educational group); updated project structure to reference all active domains and `@satie/utils` package

-
