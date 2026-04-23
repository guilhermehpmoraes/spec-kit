# Feature Spec: Admin Subscription Domain

- **Feature ID**: 003-admin-subscription-domain
- **Status**: Archived
- **Created**: 2026-04-17
- **Last Updated**: 2026-04-17
- **Owner**: Guilherme Moraes
- **Domain**: subscription
- **Application**: admin
- **Source Inputs**: User requirement — plan management domain with products, permissions, and plans
- **Feature Folder**: docs/specs/features/003-admin-subscription-domain/
- **Feature File**: docs/specs/features/003-admin-subscription-domain/feature.spec.md
- **Plan File**: docs/specs/features/003-admin-subscription-domain/plan.spec.md
- **Task Folder**: docs/specs/features/003-admin-subscription-domain/tasks/
- **Related ADRs**: ADR-004, ADR-005, ADR-010

## 1. Context

The Admin platform needs a way to manage **subscription plans** for the products it administers. Each product (e.g., Satie) exposes a set of **permissions** (capabilities). A **plan** groups a custom selection of those permissions per product, allowing different tiers (basic, intermediate, advanced) to be offered to clients.

Products and permissions are foundational catalog data — they are inserted via migrations and only exposed through read endpoints. Plans, on the other hand, are fully managed through the Admin UI: operators create a plan for a specific product, selecting which of that product's permissions the plan grants. A plan always belongs to exactly one product (1:N relationship), keeping the schema simple and matching the real-world model where each tier (basic, intermediate, advanced) is scoped to a single product.

The create and update operations use a single rich payload that includes the plan metadata and its permission selection, matching the single-form UI approach. This provides atomic operations (all-or-nothing in a single transaction) and minimizes round trips.

This domain is a prerequisite for future client subscription management, where a client account will be linked to a specific plan that determines which product features they can access.

## 2. Scope

### In Scope

- Database schema for `produto`, `permissao`, `plano`, and `plano_permissao` tables
- TypeORM entities for all four tables, inheriting from `EntidadeBase`
- Migrations to create all tables (structure only — seed data deferred to future tasks)
- Read-only endpoints for products and their permissions (`GET` only)
- Full CRUD endpoints for plans with rich payloads (create and update include product and permissions in a single request)
- Strict validation: permissions assigned to a plan must belong to the plan's product
- JWT authentication on all endpoints (reusing Identity domain's auth)
- Swagger/Scalar API documentation for all endpoints
- Unit and integration tests

### Out of Scope

- Frontend UI for plan management (future feature)
- Client-plan assignment (future feature — links a client account to a plan)
- Seed migration with actual product/permission data (will be a separate task when data is defined)
- Permission enforcement at runtime (checking if a user has a specific permission based on their plan)
- Billing or payment integration

## 3. User Scenarios (Prioritized)

### Scenario 1 - List Products and Permissions (P1)

An admin operator opens the plan management area and needs to see which products exist and what permissions each product offers, so they can make informed decisions when building plans.

#### Acceptance Scenarios

1. **Given** products and permissions exist in the database, **When** the operator requests the list of products, **Then** all active products are returned with their codes and descriptions.
2. **Given** a product with permissions exists, **When** the operator requests the product's permissions, **Then** all permissions for that product are returned.

### Scenario 2 - Create a Plan with Product and Permissions (P1)

An admin operator creates a new plan in a single form: they provide the plan name, description, color, active status, select the target product, and pick which permissions to include. Everything is submitted as one atomic operation.

#### Acceptance Scenarios

1. **Given** the operator provides valid plan data (nome, descricao, cor, ativo, idProduto, permissoes), **When** they submit the creation request, **Then** a new plan is created with the selected permissions and returned with its generated ID.
2. **Given** the operator provides an invalid color format (not a 7-char hex string), **When** they submit the creation request, **Then** the request is rejected with a validation error.
3. **Given** a product with permissions A, B, C, **When** the operator tries to create a plan with permission D (which belongs to a different product), **Then** the request is rejected with a validation error.
4. **Given** the operator provides a non-existent product ID, **When** they submit the creation request, **Then** the request is rejected with a not-found error.

### Scenario 3 - Edit a Plan (P1)

An admin operator updates an existing plan's name, description, color, active status, or permissions. The product cannot be changed after creation (if the operator needs a different product, they create a new plan).

#### Acceptance Scenarios

1. **Given** a plan exists, **When** the operator updates its name, color, and permissions, **Then** the plan is updated, permissions are replaced with the new selection, and the updated data is returned.
2. **Given** a plan exists, **When** the operator deactivates the plan (ativo = false), **Then** the plan is updated but its permissions remain intact.
3. **Given** a plan for product X with permissions A, B, **When** the operator tries to update with permission C (which belongs to a different product), **Then** the request is rejected with a validation error.

### Scenario 4 - View Plan Details (P2)

An admin operator views a plan's full details including its product and selected permissions.

#### Acceptance Scenarios

1. **Given** a plan with permissions, **When** the operator requests the plan details, **Then** the response includes the plan data, the associated product, and the selected permissions.

### Scenario 5 - Delete a Plan (P3)

An admin operator soft-deletes a plan that is no longer needed.

#### Acceptance Scenarios

1. **Given** a plan exists with permissions, **When** the operator deletes the plan, **Then** the plan is soft-deleted and no longer appears in listing queries.

## 4. Functional Requirements

- **FR-001**: The system MUST expose read-only endpoints for products (`GET /produtos`, `GET /produtos/:id`).
- **FR-002**: The system MUST expose read-only endpoints for permissions scoped to a product (`GET /produtos/:idProduto/permissoes`).
- **FR-003**: The system MUST support full CRUD for plans (`POST /planos`, `GET /planos`, `GET /planos/:id`, `PATCH /planos/:id`, `DELETE /planos/:id`).
- **FR-004**: `POST /planos` MUST accept a rich payload including `idProduto` and `permissoes` (array of permission UUIDs), creating the plan and its permission associations atomically in a single transaction.
- **FR-005**: `PATCH /planos/:id` MUST accept optional `permissoes` array; when provided, it replaces all existing permission associations for the plan.
- **FR-006**: The `idProduto` of a plan MUST NOT be changeable after creation.
- **FR-007**: `GET /planos/:id` MUST return the plan with its product and selected permissions in a nested response.
- **FR-008**: The system MUST validate that all permissions in the request belong to the plan's product.
- **FR-009**: (Removed — N:N product association no longer applies; a plan has exactly one product.)
- **FR-010**: The system MUST require JWT authentication on all endpoints.
- **FR-011**: All entities MUST inherit from `EntidadeBase` for audit fields and soft deletes.
- **FR-012**: The `cor` field MUST be validated as a 7-character hex color string (e.g., `#FF5733`).
- **FR-013**: The `codigo_produto` field MUST be unique across all products.
- **FR-014**: The `codigo_permissao` field MUST be unique across all permissions.
- **FR-015**: All primary keys MUST be UUIDs (v4).

## 5. Engineering Quality Constraints

- **EQ-001 (KISS)**: The implementation MUST prefer the simplest design that satisfies all acceptance scenarios.
- **EQ-002 (Self-descriptive code)**: New code MUST be understandable through naming and structure without relying on comments.
- **EQ-003 (Comments policy)**: Comments MUST be added only for non-obvious intent, tradeoffs, or constraints.
- **EQ-004 (Testability)**: Boundaries and responsibilities MUST enable focused unit and integration tests.
- **EQ-005 (Patterns baseline)**: Design choices MUST follow current ADR baseline unless explicitly justified.

## 6. Dependencies and Risks

### Dependencies

- `@satie/database` — `EntidadeBase`, `SnakeNamingStrategy`, TypeORM config
- Identity domain — JWT auth guard and strategy (reuse `JwtAuthGuard` from `identity/auth`)
- PostgreSQL and Redis (existing Docker Compose setup)

### Risks

- **Schema evolution**: Future client-plan linking may require additional relationship tables or changes to the plan model. → Mitigated by keeping the plan schema focused on its current purpose and deferring client assignment.
- **Permission validation complexity**: Validating that permissions belong to the correct product adds query overhead on write operations. → Mitigated by the fact that permission catalogs are small and can be cached if needed.

## 7. Success Criteria

- **SC-001**: All four database tables are created via migrations and match the specified schema.
- **SC-002**: Products and permissions can be read through GET endpoints.
- **SC-003**: Plans can be created (with product and permissions), listed, viewed, updated, and soft-deleted via single-endpoint operations.
- **SC-004**: Permission associations are managed atomically as part of plan create/update.
- **SC-005**: Permission validation prevents cross-product permission assignment.
- **SC-006**: All endpoints are protected by JWT authentication.
- **SC-007**: All endpoints are documented in Scalar API docs.
- **SC-008**: Unit and integration tests cover all acceptance scenarios.

## 8. Validation Mapping

- **AC S1.1, S1.2** → integration test → Products/Permissions controllers
- **AC S2.1** → integration test → Plans controller (create with product + permissions)
- **AC S2.2** → unit test → Color validation DTO
- **AC S2.3** → integration test → Permission validation service (cross-product rejection)
- **AC S2.4** → integration test → Plans controller (non-existent product rejection)
- **AC S3.1** → integration test → Plans controller (update with permission replacement)
- **AC S3.2** → integration test → Plans controller (deactivate preserves permissions)
- **AC S3.3** → integration test → Permission validation service (cross-product rejection on update)
- **AC S4.1** → integration test → Plans controller (get with nested product + permissions)
- **AC S5.1** → integration test → Plans controller (soft-delete)
- **EQ-001/EQ-005** → Design review against ADRs
- **EQ-002/EQ-003** → Code review checklist
- **EQ-004** → Unit/integration test coverage

## 9. Planning Inputs for Step 2

### Affected Areas

- Backend: new `subscription` domain module under `apps/admin/backend/src/subscription/`
- Database: 4 new tables with migrations
- Shared: reuses `@satie/database` (no changes expected)
- Tests: new unit and integration tests in `apps/admin/backend-tests/`

### Proposed Implementation Slices

- Slice 1: Database entities and migrations (all 4 tables)
- Slice 2: Products and permissions read-only endpoints
- Slice 3: Plans CRUD endpoints (create/update with rich payload including permissions)
- Slice 4: Validation, tests, and API documentation

### Contracts and Constraints

#### API Endpoints

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/produtos` | GET | List all products |
| `/produtos/:id` | GET | Get product by ID (with permissions) |
| `/produtos/:idProduto/permissoes` | GET | List permissions for a product |
| `/planos` | POST | Create a new plan (payload includes idProduto + permissoes) |
| `/planos` | GET | List all plans (with product info) |
| `/planos/:id` | GET | Get plan with product and permissions (nested) |
| `/planos/:id` | PATCH | Update plan (payload may include permissoes for replacement) |
| `/planos/:id` | DELETE | Soft-delete a plan |

#### Database Schema

**produto**

| Column | Type | Constraints |
|--------|------|-------------|
| id_produto | UUID | PK, DEFAULT uuid_generate_v4() |
| descricao | VARCHAR(255) | NOT NULL |
| codigo_produto | VARCHAR(100) | NOT NULL, UNIQUE |
| + EntidadeBase fields | | |

**permissao**

| Column | Type | Constraints |
|--------|------|-------------|
| id_permissao | UUID | PK, DEFAULT uuid_generate_v4() |
| codigo_permissao | VARCHAR(100) | NOT NULL, UNIQUE |
| id_produto | UUID | NOT NULL, FK → produto(id_produto) |
| descricao | VARCHAR(255) | NOT NULL |
| + EntidadeBase fields | | |

**plano**

| Column | Type | Constraints |
|--------|------|-------------|
| id_plano | UUID | PK, DEFAULT uuid_generate_v4() |
| nome | VARCHAR(150) | NOT NULL |
| descricao | TEXT | NULL |
| cor | VARCHAR(7) | NOT NULL |
| ativo | BOOLEAN | NOT NULL, DEFAULT true |
| id_produto | UUID | NOT NULL, FK → produto(id_produto) |
| + EntidadeBase fields | | |

**plano_permissao**

| Column | Type | Constraints |
|--------|------|-------------|
| id_plano_permissao | UUID | PK, DEFAULT uuid_generate_v4() |
| id_plano | UUID | NOT NULL, FK → plano(id_plano) |
| id_permissao | UUID | NOT NULL, FK → permissao(id_permissao) |
| + EntidadeBase fields | | |
| | | UNIQUE(id_plano, id_permissao) |

### Technical Detail Baseline (Step 2 Input)

- **Database changes**: 4 new tables as specified above. All inherit `EntidadeBase` audit fields. UUIDs for all PKs. Unique constraints on codes and composite key on `plano_permissao`.
- **API contracts**: RESTful endpoints following existing patterns from the Identity domain. Create and update use rich payloads (plan data + permissions in one request). All responses via dedicated DTOs. Swagger decorators with `@ApiProperty` including `description` and `example`.
- **Migration notes**: Forward migration creates tables in dependency order (produto → permissao → plano → plano_permissao). Rollback drops in reverse order. No seed data in this feature.

### Known Dependencies

- `@satie/database` — `EntidadeBase`, TypeORM config
- Identity domain — `JwtAuthGuard` for endpoint protection
- `uuid-ossp` PostgreSQL extension (for `uuid_generate_v4()`)

### Open Questions to Resolve in Planning

- Should listing endpoints support pagination, filtering, or sorting? (Assumption: simple listing without pagination for now, given small catalog sizes)

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
- [x] Task files are generated under `docs/specs/features/003-admin-subscription-domain/tasks/`.
- [x] At least one task has status `Ready`.

## 11. Optional Data Impact

### Entities

- **Produto**: Represents a platform product (e.g., Satie). Catalog data seeded via migration.
- **Permissao**: Represents a capability within a product. Catalog data seeded via migration.
- **Plano**: Represents a subscription tier for a specific product. Fully managed via API. Has a direct FK to `produto`.
- **PlanoPermissao**: Associates specific permissions with a plan. Junction entity.

### Data Notes

- Products and permissions are read-only from the API perspective; they are managed exclusively via migrations.
- Soft deletes apply to all entities. Deleting a plan soft-deletes the plan record; associated `plano_permissao` records should also be handled (cascade soft-delete or leave orphaned — to be decided in planning).
- The `uuid-ossp` extension must be enabled in PostgreSQL for UUID generation.
- A plan's `id_produto` is immutable after creation. To change the product, create a new plan.

## 12. Fixed Learnings Input (SDD Evolution)

### Keep

- Bottom-up slice ordering (entities → read API → CRUD → integration tests) worked well for domain features
- Rich payload CRUD pattern (plan + permissions in single request) simplified both implementation and testing
- Task specs were self-sufficient — no extra discovery needed during implementation
- 5-step SDD flow worked smoothly for this feature size

### Avoid

- Nothing significant to avoid — smooth delivery

### Promote to Standard

- Standalone `UpdateDto` when create/update have different field semantics (e.g., immutable fields)
- Direct repository injection in services when cross-module service injection would cause circular dependencies
- Test data seeding via raw SQL with fixed UUIDs for catalog entities without create APIs

### Process Improvement for Next Cycle

- Keep architecture docs updated during task implementation (PlansModule was missing from the hierarchy)
- Keep README migration description current when new tables are added

## Retrospective

- **Date**: 2026-04-17
- **Key findings**:
  - Full feature delivered in a single day (4 tasks, 48 unit tests, 97 integration tests)
  - No friction reported across any SDD step
  - ADRs (004, 005, 010) remain valid — no updates needed
  - Architecture doc was missing PlansModule in module hierarchy (fixed)
  - README migration description was outdated (fixed)
- **Changes applied**:
  - [docs/specs/apps/admin/architecture.md](docs/specs/apps/admin/architecture.md) — added PlansModule subtree to module hierarchy
  - [README.md](README.md) — updated migration step to mention subscription domain tables
