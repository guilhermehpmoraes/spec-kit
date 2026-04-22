# Feature Spec: Admin Identity Domain

- **Feature ID**: 001-admin-identity-domain
- **Status**: Archived
- **Created**: 2026-04-14
- **Last Updated**: 2026-04-16
- **Owner**: Guilherme Moraes
- **Domain**: [identity](../../domains/identity.md)
- **Application**: admin
- **Source Inputs**: User request — admin user management domain (CRUD, JWT auth, no permission-based access)
- **Feature Folder**: docs/specs/features/001-admin-identity-domain/
- **Feature File**: docs/specs/features/001-admin-identity-domain/feature.spec.md
- **Plan File**: docs/specs/features/001-admin-identity-domain/plan.spec.md
- **Task Folder**: docs/specs/features/001-admin-identity-domain/tasks/
- **Related ADRs**: ADR-001, ADR-002, ADR-003, ADR-004, ADR-005

## 1. Context

The Admin platform currently has no user management or authentication system. Before any operational feature can be built, the platform needs a way to identify who is using it, control access, and manage the lifecycle of administrative users.

This feature establishes the **identity** domain for the Admin application — covering CRUD operations for admin users and JWT-based authentication (login, token refresh, logout). All users of this platform have equal access to all operations (no role or permission model).

This is a foundational capability that unblocks every other Admin feature.

## 2. Scope

### In Scope

- CRUD of admin users (create, read, update, delete)
- User entity with fields: `id_usuario_admin`, `email`, `senha` (hashed), plus inherited audit and soft-delete fields
- All entities inherit from a shared auditable base model (`EntidadeBase`) providing: `criado_por`, `modificado_por`, `criado_as`, `modificado_as`, `deletado_as`, `deletado_por` (soft delete)
- Shared database package (`@satie/database`) in `packages/database/` containing base entity, TypeORM config, and naming strategy — shared across all applications
- `typeorm-naming-strategies` (SnakeNamingStrategy) to enforce snake_case for all table and column names automatically
- Entity classes named in Portuguese for consistency with database column names (e.g., `UsuarioAdmin`, `TokenRevogado`)
- Password hashing with bcrypt and strong policy enforcement (8+ chars, uppercase, number, special character)
- JWT authentication: login endpoint returning access token + refresh token
- Token refresh endpoint
- Logout endpoint with server-side token invalidation via Redis
- Redis-backed token revocation (blacklist) with TTL-based auto-cleanup
- Initial super-admin user seeded via database migration
- Only authenticated admin users can create new users (no self-registration)
- All endpoints documented with Swagger/Scalar
- TypeORM as the ORM for NestJS
- Tests following project-defined tooling (Jest + Supertest for backend)
- Docker Compose for local development: PostgreSQL 17.4 + Redis (latest) with named volumes for data persistence

### Out of Scope

- Role-based or permission-based access control
- Password recovery / reset flow
- Email verification
- Multi-factor authentication (MFA)
- Frontend implementation (will be a separate feature)
- Integration with external identity providers (OAuth, SAML, etc.)

## 3. User Scenarios (Prioritized)

### Scenario 1 — Admin Login (P1)

An admin user opens the Admin platform and authenticates using email and password. Upon successful login, they receive an access token and a refresh token to maintain their session.

#### Acceptance Scenarios

1. **Given** valid credentials exist in the database, **When** the user submits email and password to the login endpoint, **Then** the system returns a 200 response with an access token and a refresh token.
2. **Given** invalid credentials are submitted, **When** the user attempts to log in, **Then** the system returns a 401 Unauthorized response with a generic error message (no credential-specific hints).
3. **Given** a valid refresh token, **When** the user calls the refresh endpoint, **Then** the system returns a new access token.
4. **Given** an expired or invalid refresh token, **When** the user calls the refresh endpoint, **Then** the system returns a 401 Unauthorized response.

### Scenario 2 — Create Admin User (P1)

An authenticated admin creates a new admin user by providing email and password.

#### Acceptance Scenarios

1. **Given** an authenticated admin, **When** they submit a valid email and password to the create user endpoint, **Then** the system creates the user with a bcrypt-hashed password and returns the user data (without the password).
2. **Given** an authenticated admin, **When** they submit an email that already exists, **Then** the system returns a 409 Conflict response.
3. **Given** an authenticated admin, **When** they submit a password that does not meet the strong policy (8+ chars, uppercase, number, special), **Then** the system returns a 400 Bad Request with validation details.
4. **Given** an unauthenticated request, **When** attempting to create a user, **Then** the system returns a 401 Unauthorized response.

### Scenario 3 — List and Read Admin Users (P1)

An authenticated admin lists all admin users or reads a single user's details.

#### Acceptance Scenarios

1. **Given** an authenticated admin, **When** they call the list users endpoint, **Then** the system returns a list of all users (without password fields).
2. **Given** an authenticated admin and a valid user ID, **When** they call the get user endpoint, **Then** the system returns that user's data (without password).
3. **Given** an authenticated admin and an invalid user ID, **When** they call the get user endpoint, **Then** the system returns a 404 Not Found.

### Scenario 4 — Update Admin User (P2)

An authenticated admin updates an existing user's email or password.

#### Acceptance Scenarios

1. **Given** an authenticated admin and a valid user ID, **When** they submit updated fields, **Then** the system updates the user and returns the updated data (without password).
2. **Given** an update with a new email that already exists for another user, **When** submitted, **Then** the system returns a 409 Conflict.
3. **Given** an update with a new password that does not meet policy, **When** submitted, **Then** the system returns a 400 Bad Request.

### Scenario 5 — Delete Admin User (P2)

An authenticated admin deletes an existing admin user.

#### Acceptance Scenarios

1. **Given** an authenticated admin and a valid user ID, **When** they call the delete endpoint, **Then** the system removes the user and returns a 204 No Content.
2. **Given** a valid user ID that is the last remaining admin, **When** the delete endpoint is called, **Then** the system returns a 400 Bad Request (prevent deleting the last admin).
3. **Given** an invalid user ID, **When** the delete endpoint is called, **Then** the system returns a 404 Not Found.

### Scenario 6 — Admin Logout (P2)

An authenticated admin logs out, invalidating their current tokens.

#### Acceptance Scenarios

1. **Given** an authenticated admin with a valid access token, **When** they call the logout endpoint, **Then** the system invalidates the refresh token and returns 204 No Content.
2. **Given** a previously invalidated token, **When** used to access protected endpoints, **Then** the system returns a 401 Unauthorized.

## 4. Functional Requirements

- **FR-001**: System MUST authenticate admin users via email + password and return a JWT access token and refresh token.
- **FR-002**: System MUST hash all passwords using bcrypt before storing them.
- **FR-003**: System MUST enforce password policy: minimum 8 characters, at least one uppercase letter, one number, and one special character.
- **FR-004**: System MUST support token refresh via a dedicated endpoint.
- **FR-005**: System MUST support server-side logout by invalidating refresh tokens via Redis.
- **FR-006**: System MUST expose CRUD endpoints for admin users, all protected by JWT authentication.
- **FR-007**: System MUST NOT return password fields in any user response.
- **FR-008**: System MUST prevent deletion of the last remaining admin user.
- **FR-009**: System MUST enforce unique email constraint for admin users.
- **FR-010**: System MUST seed an initial super-admin user via database migration.
- **FR-011**: All API endpoints MUST be documented with Swagger/Scalar decorators.
- **FR-012**: System MUST use TypeORM as the ORM for all database operations in this domain.
- **FR-013**: System MUST return generic error messages on authentication failure (no credential-specific hints).
- **FR-014**: All database entities MUST inherit from `EntidadeBase` providing audit fields (`criado_por`, `modificado_por`, `criado_as`, `modificado_as`) and soft-delete fields (`deletado_as`, `deletado_por`).
- **FR-015**: Database MUST use `typeorm-naming-strategies` (SnakeNamingStrategy) to enforce snake_case for all table and column names.
- **FR-016**: Entity classes MUST be named in Portuguese to be consistent with database column names.
- **FR-017**: The shared database package (`@satie/database`) MUST live in `packages/database/` and be reusable across all applications.
- **FR-018**: Token revocation MUST use Redis with TTL matching the token's remaining lifetime for auto-cleanup.
- **FR-019**: All deletes MUST be soft deletes (setting `deletado_as` and `deletado_por`) unless explicitly stated otherwise.
- **FR-020**: A `docker-compose.yml` at the repository root MUST provide PostgreSQL 17.4 and Redis (latest) services for local development, with named volumes for persistent data.

## 5. Engineering Quality Constraints

- **EQ-001 (KISS)**: The implementation MUST prefer the simplest design that satisfies all acceptance scenarios. No premature abstractions for future auth features (MFA, roles, etc.).
- **EQ-002 (Self-descriptive code)**: New code MUST be understandable through naming and structure without relying on comments.
- **EQ-003 (Comments policy)**: Comments MUST be added only for non-obvious intent, tradeoffs, or constraints.
- **EQ-004 (Testability)**: Boundaries and responsibilities MUST enable focused unit and integration tests. Service, repository, and controller layers must be independently testable.
- **EQ-005 (Patterns baseline)**: Design MUST follow Controller-Service-Repository pattern (ADR-002). Module pattern as domain boundary. DTO + validation for explicit API contracts.

## 6. Dependencies and Risks

### Dependencies

- PostgreSQL database (must be available for TypeORM connection)
- Redis instance (for token revocation blacklist)
- NestJS packages: `@nestjs/jwt`, `@nestjs/passport`, `passport-jwt`
- TypeORM packages: `@nestjs/typeorm`, `typeorm`, `pg`
- Snake case naming: `typeorm-naming-strategies`
- Redis packages: `ioredis` (or `@nestjs/cache-manager` with Redis store)
- bcrypt package for password hashing
- Swagger packages: `@nestjs/swagger`
- Shared package: `@satie/database` in `packages/database/`
- Existing Admin backend application scaffold (`apps/admin/backend/`)

### Risks

- **Redis availability**: Token revocation depends on Redis. If Redis is down, token validation fails open or closed. -> Mitigation: Fail closed (reject requests if Redis is unreachable) for security. Document Redis as a required infrastructure dependency.
- **Migration seed stability**: The initial super-admin seed must not fail on re-runs. -> Mitigation: Use idempotent seed logic (insert only if no admin exists).
- **TypeORM configuration**: First domain to introduce TypeORM; configuration must be done at the app module level via the shared `@satie/database` package. -> Mitigation: Follow NestJS + TypeORM official documentation.
- **Shared package bootstrap**: `packages/database/` is the first shared package in the monorepo. Need to ensure Nx project configuration, TypeScript paths, and pnpm workspace linking are correct. -> Mitigation: Follow Nx library generation patterns.

## 7. Success Criteria

- **SC-001**: Admin users can successfully log in and receive JWT tokens.
- **SC-002**: All CRUD operations for admin users are functional and tested.
- **SC-003**: Password policy is enforced on creation and update.
- **SC-004**: Token refresh and logout work correctly with server-side invalidation.
- **SC-005**: All endpoints are documented and visible in Swagger/Scalar UI.
- **SC-006**: The initial super-admin user is seeded and functional after migration.
- **SC-007**: Integration tests cover all acceptance scenarios using Jest + Supertest.

## 8. Validation Mapping

- **Scenario 1 (Login) ACs** -> integration test -> AuthController + AuthService
- **Scenario 2 (Create User) ACs** -> integration test -> UsersController + UsersService
- **Scenario 3 (List/Read) ACs** -> integration test -> UsersController + UsersService
- **Scenario 4 (Update) ACs** -> integration test -> UsersController + UsersService
- **Scenario 5 (Delete) ACs** -> integration test -> UsersController + UsersService
- **Scenario 6 (Logout) ACs** -> integration test -> AuthController + AuthService
- **FR-002 (bcrypt)** -> unit test -> UsersService (password hashing)
- **FR-003 (password policy)** -> unit test -> password validation DTO/pipe
- **FR-007 (no password in response)** -> integration test -> all user endpoints
- **FR-008 (last admin guard)** -> integration test -> delete endpoint
- **FR-010 (seed)** -> migration verification test
- **EQ-001/EQ-005** -> Design review against ADR-001, ADR-002
- **EQ-002/EQ-003** -> Code review checklist
- **EQ-004** -> Unit + integration test coverage

## 9. Planning Inputs for Step 2

### Affected Areas

- **Shared package**: `packages/database/` — `@satie/database` with `EntidadeBase`, TypeORM config, SnakeNamingStrategy setup
- **Backend**: `apps/admin/backend/` — new NestJS modules (identity domain), TypeORM configuration via `@satie/database`, JWT/Passport setup, Redis integration
- **Database**: New PostgreSQL table for admin users (with audit + soft-delete columns)
- **Redis**: Token revocation blacklist store
- **Infra**: Environment variables for JWT secret, token expiry, database connection, Redis connection
- **Docker**: `docker-compose.yml` at repo root for local PostgreSQL 17.4 + Redis (latest) with persistent volumes
- **QA**: Integration tests with Jest + Supertest

### Proposed Implementation Slices

1. **Slice 1 — Docker Compose for Local Dev**: Create `docker-compose.yml` at repo root with PostgreSQL 17.4 and Redis (latest) services. Named volumes for data persistence. Default environment variables for local connections.
2. **Slice 2 — Shared Database Package**: Create `packages/database/` with `EntidadeBase` (audit + soft-delete fields), SnakeNamingStrategy config, TypeORM shared configuration. Nx project setup and pnpm workspace linking.
3. **Slice 3 — TypeORM + Database Setup**: Configure TypeORM in Admin backend via `@satie/database`, create `UsuarioAdmin` entity inheriting `EntidadeBase`, run initial migration with super-admin seed.
4. **Slice 4 — User CRUD**: Implement UsersModule with controller, service, repository. DTOs with validation. Swagger decorators. Soft-delete behavior. Unit + integration tests.
5. **Slice 5 — Redis + JWT Authentication**: Set up Redis connection, implement AuthModule with login, refresh, logout endpoints. JWT + Passport strategy. Redis-backed token blacklist. Guards for protected routes. Unit + integration tests.
6. **Slice 6 — Integration & Polish**: Apply JWT guard globally to identity endpoints, verify all Swagger documentation, run full test suite, validate seed migration.

### Contracts and Constraints

#### Docker Compose — Local Development

| Service | Image | Port | Volume |
|---------|-------|------|--------|
| `postgres` | `postgres:17.4` | `5432:5432` | `satie_pgdata:/var/lib/postgresql/data` |
| `redis` | `redis:latest` | `6379:6379` | `satie_redisdata:/data` |

Default environment variables for PostgreSQL:
- `POSTGRES_USER=satie`
- `POSTGRES_PASSWORD=satie`
- `POSTGRES_DB=satie_dev`

#### Shared Base Entity: `EntidadeBase`

All entities inherit these fields automatically:

| Coluna | Tipo | Constraints | Notes |
|--------|------|-------------|-------|
| `criado_por` | VARCHAR(255) | NOT NULL | User ID or 'system' |
| `modificado_por` | VARCHAR(255) | NOT NULL | User ID or 'system' |
| `criado_as` | TIMESTAMP | NOT NULL, DEFAULT NOW() | Immutable after creation |
| `modificado_as` | TIMESTAMP | NOT NULL, DEFAULT NOW() | Updated on every change |
| `deletado_as` | TIMESTAMP | NULL | Soft delete marker (TypeORM @DeleteDateColumn) |
| `deletado_por` | VARCHAR(255) | NULL | Who performed the soft delete |

#### Database — Table: `usuario_admin`

| Coluna | Tipo | Constraints |
|--------|------|-------------|
| `id_usuario_admin` | UUID | PK, generated |
| `email` | VARCHAR(255) | NOT NULL, UNIQUE |
| `senha` | VARCHAR(255) | NOT NULL |
| _(inherited from EntidadeBase)_ | | |

#### Token Revocation — Redis

| Key Pattern | Value | TTL |
|-------------|-------|-----|
| `revoked_token:<jti>` | `1` | Remaining token lifetime |

#### API Contracts

| Endpoint | Method | Auth | Request Body | Success Response |
|----------|--------|------|-------------|-----------------|
| `POST /auth/login` | POST | No | `{ email, senha }` | `200 { accessToken, refreshToken }` |
| `POST /auth/refresh` | POST | No | `{ refreshToken }` | `200 { accessToken }` |
| `POST /auth/logout` | POST | Yes | `{ refreshToken }` | `204 No Content` |
| `POST /usuarios-admin` | POST | Yes | `{ email, senha }` | `201 { id_usuario_admin, email, criado_em }` |
| `GET /usuarios-admin` | GET | Yes | — | `200 [{ id_usuario_admin, email, criado_em, atualizado_em }]` |
| `GET /usuarios-admin/:id` | GET | Yes | — | `200 { id_usuario_admin, email, criado_em, atualizado_em }` |
| `PATCH /usuarios-admin/:id` | PATCH | Yes | `{ email?, senha? }` | `200 { id_usuario_admin, email, criado_em, atualizado_em }` |
| `DELETE /usuarios-admin/:id` | DELETE | Yes | — | `204 No Content` |

#### Password Policy

- Minimum 8 characters
- At least one uppercase letter
- At least one number
- At least one special character

#### JWT Configuration

- Access token: short-lived (e.g., 15 min)
- Refresh token: longer-lived (e.g., 7 days)
- Each token includes a `jti` (JWT ID) claim for revocation tracking
- Secret key loaded from environment variable

### Known Dependencies

- PostgreSQL 17.4 running and accessible (via Docker Compose for local dev)
- Redis (latest) running and accessible (via Docker Compose for local dev)
- Docker and Docker Compose installed for local development
- `@satie/database` shared package created and linked
- NestJS packages for JWT, Passport, TypeORM, Swagger
- `typeorm-naming-strategies` for SnakeNamingStrategy
- `ioredis` for Redis connectivity
- bcrypt for password hashing

### Open Questions to Resolve in Planning

1. What should the exact JWT expiry times be (access token and refresh token)?
2. Should the super-admin seed password come from an environment variable or be hardcoded for dev only?
3. Should list users support pagination from the start, or add it later?
4. Should the `@satie/database` package also export a shared Redis module, or keep Redis config per-app for now?

## 10. Workflow and Status Gates

### Status Transition Rules

- On creation in Step 1, set **Status** to `Draft` and save the feature spec file.
- After user approval, update **Status** to `Approved`.
- When Step 2 starts, update **Status** to `In Planning`.
- After feature plan approval and task file generation, update **Status** to `Planned`.
- When implementation of the first task starts, update **Status** to `In Implementation`.
- After all scoped tasks are done and validated, update **Status** to `Done`.

### Gate to move Step 1 -> Step 2

- All acceptance scenarios reviewed and confirmed
- Open questions acknowledged (can be resolved in Step 2)
- Scope agreed — no unresolved scope creep

## Retrospective

- **Date**: 2026-04-16
- **Feature**: 001-admin-identity-domain — Admin Identity Domain

### Key Findings

- The full SDD lifecycle (spec → plan → tasks → implementation) flowed smoothly; task specs were self-sufficient for implementation
- Templates captured everything needed — no gaps or unnecessary sections identified
- Service/module/controller patterns established in the identity domain are solid and reusable
- **Friction 1**: Biome warnings were produced during implementation but not resolved, leaving code out of the project's quality standard
- **Friction 2**: T005 was marked `Done` with failing integration tests; the user had to manually test and fix errors afterward
- **Deviation**: `tsconfig.migrations.json` was created to decouple SWC compilation from TypeORM CLI path resolution (not in original plan)
- **Bookkeeping gaps**: Plan spec status and most task Owner fields were not updated during implementation
- Two new ADRs were created during the feature: ADR-006 (Single Version Dependency Policy) and ADR-007 (Test Project TypeScript Configuration)

### Changes Applied

- `.github/copilot-instructions.md` — Added **Biome Compliance (Mandatory)** rule: run Biome after every code change, resolve all warnings/errors before marking a task Done
- `.github/copilot-instructions.md` — Added **Test Pass Gate (Mandatory)** rule: all related tests must pass before a task can transition to Done
- `docs/specs/templates/task.spec.md` — Updated Definition of Done with two new checkboxes: test pass verification and Biome compliance
- `README.md` — Updated with project setup, prerequisites, environment variables, Docker Compose usage, running the app, running migrations, and running tests
