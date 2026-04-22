# Feature Plan: Admin Identity Domain

- **Feature ID**: 001-admin-identity-domain
- **Plan ID**: PLAN-001-admin-identity-domain
- **Status**: Completed
- **Date**: 2026-04-15
- **Last Updated**: 2026-04-16
- **Owner**: TBD
- **Feature Folder**: docs/specs/features/001-admin-identity-domain/
- **Feature Spec**: docs/specs/features/001-admin-identity-domain/feature.spec.md
- **Plan File**: docs/specs/features/001-admin-identity-domain/plan.spec.md
- **Task Output Folder**: docs/specs/features/001-admin-identity-domain/tasks/
- **Related ADRs**: ADR-001, ADR-002, ADR-003, ADR-004, ADR-005

## 1. Planning Goal

Define a complete, implementation-ready plan for the Admin Identity Domain: a shared database package (`@satie/database`), Docker Compose for local dev, TypeORM integration, admin user CRUD, JWT authentication (login, refresh, logout), Redis-backed token revocation, and the initial super-admin seed — with enough detail to generate self-sufficient task specs.

## 2. Summary

- **Feature objective**: Establish the identity domain for the Admin application — CRUD operations for admin users and JWT-based authentication, enabling all future Admin features that require user identity.
- **Why now**: No authentication or user management exists. This is a foundational capability that unblocks every other Admin feature.
- **Expected outcome**: A fully functional Auth + Users API with JWT tokens, Redis-backed revocation, soft-delete users, Swagger docs, integration tests, and a seeded super-admin — all backed by a shared database package usable across all future applications.
- **Implementation direction**: Bottom-up — infrastructure first (Docker, shared package), then data layer (TypeORM + entity + migration), then domain logic (CRUD, auth), then integration (guards, tests, docs).

## 3. Technical Context

### Stack Baseline (Satie)

- **Monorepo**: Nx 22.6.4 + pnpm 10.33.0
- **Backend**: NestJS (currently a bare scaffold — empty AppModule)
- **Frontend**: Vite + React + TanStack Router + TanStack Query + Tailwind CSS (out of scope for this feature)
- **Database**: PostgreSQL 17.4 (not yet configured)
- **Cache/Ephemeral**: Redis (not yet configured)
- **Tests**: Jest + Supertest (backend), Vitest + React Testing Library (frontend), Playwright (e2e)
- **Quality**: Biome 2.4.11 (formatter + linter)
- **TypeScript**: 5.9.2, target ES2022, module nodenext, strict mode

### Feature-Specific Context

- **Touched apps/packages**: `apps/admin/backend/`, `packages/database/` (new), root `docker-compose.yml` (new)
- **New dependencies**:
  - `@nestjs/typeorm`, `typeorm`, `pg`, `typeorm-naming-strategies` — ORM and database
  - `@nestjs/jwt`, `@nestjs/passport`, `passport-jwt`, `@types/passport-jwt` — authentication
  - `bcrypt`, `@types/bcrypt` — password hashing
  - `ioredis`, `@types/ioredis` — Redis connectivity
  - `@nestjs/swagger` — API documentation
  - `@nestjs/config` — environment variable management
  - `class-validator`, `class-transformer` — DTO validation
  - `uuid`, `@types/uuid` — UUID generation (if not using TypeORM's built-in)
- **Data impact**: New PostgreSQL table `usuario_admin`, Redis keys for token revocation
- **Constraints**: Portuguese naming for entities/DB, English for all other code (ADR-005). Soft deletes mandatory (ADR-005). KISS-first (ADR-001). Controller-Service-Repository pattern (ADR-002).

### Constraints and Assumptions

- **Inputs**: HTTP requests (JSON bodies), environment variables for configuration
- **Outputs**: JSON API responses, JWT tokens, Redis keys
- **Boundary conditions**: Duplicate emails, weak passwords, expired tokens, missing Redis, last-admin deletion guard, soft-deleted user login attempts
- **Assumptions**:
  - PostgreSQL and Redis will be available via Docker Compose for local development
  - No other application currently uses `@satie/database` — this is the first consumer
  - The admin backend currently has no middleware, guards, or interceptors configured
  - TypeORM `synchronize` will NOT be used — migrations are the only schema management mechanism
  - The test project infrastructure (global setup on port 3000) may need adjustment for database-dependent tests
  - Test projects follow the naming convention `<app>-tests` (e.g., `backend-tests`, `frontend-tests`) — the existing `backend-e2e` and `frontend-e2e` directories must be renamed before or alongside this feature

## 3.1 Technical Design Baseline

### Database Design

#### Tables and Actions

| Schema | Table | Action | Purpose |
| ------ | ----- | ------ | ------- |
| public | `usuario_admin` | Create | Store admin user accounts with email, hashed password, and audit/soft-delete fields |

#### Columns Specification

| Tabela | Campo | Tipo SQL | Nullable | Default | PK | FK Ref | Unique | Index | Notes |
| ------ | ----- | -------- | -------- | ------- | -- | ------ | ------ | ----- | ----- |
| `usuario_admin` | `id_usuario_admin` | UUID | No | `gen_random_uuid()` | Yes | N/A | Yes | PK | Primary key |
| `usuario_admin` | `email` | VARCHAR(255) | No | — | No | N/A | Yes | `idx_usuario_admin_email` | Unique constraint for login lookup |
| `usuario_admin` | `senha` | VARCHAR(255) | No | — | No | N/A | No | — | bcrypt hash |
| `usuario_admin` | `criado_por` | VARCHAR(255) | No | — | No | N/A | No | — | Inherited from `EntidadeBase` |
| `usuario_admin` | `modificado_por` | VARCHAR(255) | No | — | No | N/A | No | — | Inherited from `EntidadeBase` |
| `usuario_admin` | `criado_as` | TIMESTAMP | No | `NOW()` | No | N/A | No | — | Inherited from `EntidadeBase` |
| `usuario_admin` | `modificado_as` | TIMESTAMP | No | `NOW()` | No | N/A | No | — | Inherited from `EntidadeBase` |
| `usuario_admin` | `deletado_as` | TIMESTAMP | Yes | NULL | No | N/A | No | — | Soft delete marker, inherited from `EntidadeBase` |
| `usuario_admin` | `deletado_por` | VARCHAR(255) | Yes | NULL | No | N/A | No | — | Inherited from `EntidadeBase` |

#### Constraints and Indexes

- **PK**: `pk_usuario_admin` — `id_usuario_admin`
- **UNIQUE**: `uq_usuario_admin_email` — `email`
- **INDEX**: `idx_usuario_admin_email` — `email` (B-tree, for login lookup performance)

#### Migration Strategy

- **Forward migration**: Create `usuario_admin` table with all columns, constraints, and indexes. Second statement: insert seed super-admin user (idempotent — only if no admin exists).
- **Rollback strategy**: Drop `usuario_admin` table.
- **Backfill strategy**: N/A (new table).
- **Compatibility window**: N/A (no existing data or code depends on this table).
- **Migration files location**: `apps/admin/backend/src/migrations/`
- **Migration naming**: TypeORM CLI timestamp-based naming (e.g., `1713100000000-CreateUsuarioAdmin.ts`)
- **Seed data**: Initial super-admin user with email from `ADMIN_SEED_EMAIL` env var (default: `admin@satie.local`) and password from `ADMIN_SEED_PASSWORD` env var. Seed is a separate migration that checks if any admin user exists before inserting.

### Token Revocation — Redis Design

| Key Pattern | Value | TTL | Purpose |
|-------------|-------|-----|---------|
| `revoked_token:<jti>` | `"1"` | Remaining token lifetime in seconds | Blacklist for both access and refresh tokens on logout |

- On logout, both the access token's `jti` and the refresh token's `jti` are added to the blacklist.
- TTL is calculated as `token.exp - now()` so keys auto-expire when the token would have expired anyway.
- On every authenticated request, the JWT strategy checks Redis for the access token's `jti`. If found, the request is rejected with 401.
- On refresh, the refresh token's `jti` is checked against Redis. If found, the request is rejected with 401.
- Redis failure mode: **fail closed** — if Redis is unreachable, reject the request (security over availability).

### API and Integration Contracts

#### POST /auth/login

- **Auth**: None
- **Request body**:

| Field | Type | Required | Validation | Notes |
| ----- | ---- | -------- | ---------- | ----- |
| `email` | string | Yes | Valid email format | Login identifier |
| `senha` | string | Yes | Non-empty string | Raw password to verify |

- **Success response** (200):

| Field | Type | Nullable | Source | Notes |
| ----- | ---- | -------- | ------ | ----- |
| `accessToken` | string | No | JWT sign | Short-lived (15min), contains `sub` (user ID), `email`, `jti` |
| `refreshToken` | string | No | JWT sign | Long-lived (7 days), contains `sub` (user ID), `jti`, `type: "refresh"` |

- **Error responses**:
  - `401 Unauthorized` — `{ statusCode: 401, message: "Credenciais inválidas" }` — generic, no hints about which field is wrong
  - `400 Bad Request` — validation errors (missing fields)

#### POST /auth/refresh

- **Auth**: None (refresh token in body)
- **Request body**:

| Field | Type | Required | Validation | Notes |
| ----- | ---- | -------- | ---------- | ----- |
| `refreshToken` | string | Yes | Non-empty, valid JWT | The refresh token to exchange |

- **Success response** (200):

| Field | Type | Nullable | Source | Notes |
| ----- | ---- | -------- | ------ | ----- |
| `accessToken` | string | No | JWT sign | New short-lived access token |

- **Error responses**:
  - `401 Unauthorized` — `{ statusCode: 401, message: "Token inválido ou expirado" }` — expired, invalid, or revoked refresh token

#### POST /auth/logout

- **Auth**: JWT Bearer (access token)
- **Request body**:

| Field | Type | Required | Validation | Notes |
| ----- | ---- | -------- | ---------- | ----- |
| `refreshToken` | string | Yes | Non-empty string | The refresh token to revoke |

- **Success response**: `204 No Content`
- **Behavior**: Adds both the access token's `jti` and the refresh token's `jti` to the Redis blacklist with TTL = remaining lifetime.
- **Error responses**:
  - `401 Unauthorized` — invalid or missing access token

#### POST /usuarios-admin

- **Auth**: JWT Bearer
- **Request body**:

| Field | Type | Required | Validation | Notes |
| ----- | ---- | -------- | ---------- | ----- |
| `email` | string | Yes | Valid email, max 255 chars | Must be unique |
| `senha` | string | Yes | Min 8 chars, 1 uppercase, 1 number, 1 special char | Password policy enforced |

- **Success response** (201):

| Field | Type | Nullable | Source | Notes |
| ----- | ---- | -------- | ------ | ----- |
| `idUsuarioAdmin` | string (UUID) | No | DB | User's unique ID |
| `email` | string | No | DB | User's email |
| `criadoAs` | string (ISO 8601) | No | DB | Creation timestamp |
| `modificadoAs` | string (ISO 8601) | No | DB | Last modification timestamp |

- **Error responses**:
  - `409 Conflict` — `{ statusCode: 409, message: "Email já cadastrado" }` — duplicate email
  - `400 Bad Request` — validation errors (weak password, invalid email)
  - `401 Unauthorized` — missing or invalid token

#### GET /usuarios-admin

- **Auth**: JWT Bearer
- **Success response** (200): Array of user objects (same shape as POST response, without pagination)
- **Notes**: No pagination. Returns all non-deleted admin users.

#### GET /usuarios-admin/:id

- **Auth**: JWT Bearer
- **URL params**: `id` — UUID of the admin user
- **Success response** (200): Single user object (same shape as POST response)
- **Error responses**:
  - `404 Not Found` — `{ statusCode: 404, message: "Usuário não encontrado" }` — user does not exist or is soft-deleted

#### PATCH /usuarios-admin/:id

- **Auth**: JWT Bearer
- **URL params**: `id` — UUID of the admin user
- **Request body** (all fields optional):

| Field | Type | Required | Validation | Notes |
| ----- | ---- | -------- | ---------- | ----- |
| `email` | string | No | Valid email, max 255 chars | Must be unique if changed |
| `senha` | string | No | Min 8 chars, 1 uppercase, 1 number, 1 special char | Re-hashed if provided |

- **Success response** (200): Updated user object (same shape as POST response)
- **Error responses**:
  - `404 Not Found` — user does not exist
  - `409 Conflict` — email already taken by another user
  - `400 Bad Request` — validation errors

#### DELETE /usuarios-admin/:id

- **Auth**: JWT Bearer
- **URL params**: `id` — UUID of the admin user
- **Success response**: `204 No Content`
- **Behavior**: Soft delete — sets `deletado_as` and `deletado_por` on the record.
- **Error responses**:
  - `404 Not Found` — user does not exist
  - `400 Bad Request` — `{ statusCode: 400, message: "Não é possível remover o último administrador" }` — last admin guard
  - `401 Unauthorized` — missing or invalid token

### Environment Variables

| Variable | Required | Default | Purpose |
| -------- | -------- | ------- | ------- |
| `DATABASE_HOST` | Yes | `localhost` | PostgreSQL host |
| `DATABASE_PORT` | Yes | `5432` | PostgreSQL port |
| `DATABASE_USER` | Yes | `satie` | PostgreSQL user |
| `DATABASE_PASSWORD` | Yes | `satie` | PostgreSQL password |
| `DATABASE_NAME` | Yes | `satie_dev` | PostgreSQL database name |
| `REDIS_HOST` | Yes | `localhost` | Redis host |
| `REDIS_PORT` | Yes | `6379` | Redis port |
| `JWT_SECRET` | Yes | — | Secret key for signing JWTs (no default, must be set) |
| `JWT_ACCESS_EXPIRY` | No | `15m` | Access token expiration |
| `JWT_REFRESH_EXPIRY` | No | `7d` | Refresh token expiration |
| `ADMIN_SEED_EMAIL` | No | `admin@satie.local` | Super-admin seed email |
| `ADMIN_SEED_PASSWORD` | No | — | Super-admin seed password (required for seed migration) |

### Swagger / Scalar Configuration

- Register `SwaggerModule` in `main.ts` with `DocumentBuilder`
- API title: "Admin API"
- API version: "1.0"
- Bearer auth scheme configured globally
- Serve Swagger JSON at `/api/docs-json` and Scalar UI at `/api/docs`
- All controllers and DTOs decorated with `@ApiTags`, `@ApiOperation`, `@ApiResponse`, `@ApiProperty`

### NestJS Module Structure

```
apps/admin/backend/src/
├── main.ts                          # Bootstrap + Swagger setup
├── app/
│   ├── app.module.ts                # Root module — imports all domain modules + TypeORM + Config
│   ├── app.controller.ts            # Health check (existing)
│   └── app.service.ts               # Health check (existing)
├── identity/
│   ├── identity.module.ts           # Domain boundary module — imports Auth + Users
│   ├── auth/
│   │   ├── auth.module.ts           # Auth module — JWT, Passport, Redis
│   │   ├── auth.controller.ts       # Login, refresh, logout endpoints
│   │   ├── auth.service.ts          # Auth business logic
│   │   ├── dto/
│   │   │   ├── login.dto.ts         # LoginDto (email, senha)
│   │   │   ├── refresh-token.dto.ts # RefreshTokenDto (refreshToken)
│   │   │   └── logout.dto.ts        # LogoutDto (refreshToken)
│   │   ├── strategies/
│   │   │   └── jwt.strategy.ts      # Passport JWT strategy with Redis revocation check
│   │   └── guards/
│   │       └── jwt-auth.guard.ts    # JWT authentication guard
│   ├── users/
│   │   ├── users.module.ts          # Users module
│   │   ├── users.controller.ts      # CRUD endpoints for /usuarios-admin
│   │   ├── users.service.ts         # User business logic (CRUD, password hashing, last-admin guard)
│   │   ├── dto/
│   │   │   ├── create-user.dto.ts   # CreateUserDto (email, senha with validation)
│   │   │   ├── update-user.dto.ts   # UpdateUserDto (email?, senha? with validation)
│   │   │   └── user-response.dto.ts # UserResponseDto (excludes senha)
│   │   └── entities/
│   │       └── usuario-admin.entity.ts  # UsuarioAdmin TypeORM entity
│   └── token-revocation/
│       ├── token-revocation.module.ts   # Redis token blacklist module
│       └── token-revocation.service.ts  # Revoke/check token JTI against Redis
├── migrations/
│   ├── <timestamp>-CreateUsuarioAdmin.ts    # Table creation migration
│   └── <timestamp>-SeedSuperAdmin.ts        # Super-admin seed migration
└── config/
    └── typeorm.config.ts            # TypeORM DataSource config for CLI migrations
```

### Shared Package Structure: `@satie/database`

```
packages/database/
├── package.json                # @satie/database, exports for entidade-base, config, redis
├── project.json                # Nx project config
├── tsconfig.json               # Extends root tsconfig.base.json
├── tsconfig.lib.json           # Library build config
├── src/
│   ├── index.ts                # Barrel export
│   ├── entidade-base.ts        # EntidadeBase abstract entity class
│   ├── typeorm-config.ts       # createTypeOrmConfig() helper
│   ├── naming-strategy.ts      # Re-export of SnakeNamingStrategy
│   └── redis/
│       ├── index.ts            # Barrel export for redis module
│       ├── redis.module.ts     # Global RedisModule (NestJS dynamic module)
│       └── redis.service.ts    # RedisService wrapping ioredis client
```

### `EntidadeBase` Entity Design

```typescript
// packages/database/src/entidade-base.ts
import {
  CreateDateColumn,
  UpdateDateColumn,
  DeleteDateColumn,
  Column,
} from "typeorm";

export abstract class EntidadeBase {
  @Column({ type: "varchar", length: 255 })
  criadoPor: string;

  @Column({ type: "varchar", length: 255 })
  modificadoPor: string;

  @CreateDateColumn({ type: "timestamp" })
  criadoAs: Date;

  @UpdateDateColumn({ type: "timestamp" })
  modificadoAs: Date;

  @DeleteDateColumn({ type: "timestamp", nullable: true })
  deletadoAs: Date | null;

  @Column({ type: "varchar", length: 255, nullable: true })
  deletadoPor: string | null;
}
```

### `UsuarioAdmin` Entity Design

```typescript
// apps/admin/backend/src/identity/users/entities/usuario-admin.entity.ts
import { Entity, PrimaryGeneratedColumn, Column } from "typeorm";
import { EntidadeBase } from "@satie/database";

@Entity("usuario_admin")
export class UsuarioAdmin extends EntidadeBase {
  @PrimaryGeneratedColumn("uuid")
  idUsuarioAdmin: string;

  @Column({ type: "varchar", length: 255, unique: true })
  email: string;

  @Column({ type: "varchar", length: 255 })
  senha: string;
}
```

### Docker Compose Design

```yaml
# docker-compose.yml (repo root)
services:
  postgres:
    image: postgres:17.4
    ports:
      - "5432:5432"
    environment:
      POSTGRES_USER: satie
      POSTGRES_PASSWORD: satie
      POSTGRES_DB: satie_dev
    volumes:
      - satie_pgdata:/var/lib/postgresql/data

  redis:
    image: redis:latest
    ports:
      - "6379:6379"
    volumes:
      - satie_redisdata:/data

volumes:
  satie_pgdata:
  satie_redisdata:
```

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

### Resolved Open Questions

| # | Question | Resolution |
|---|----------|------------|
| 1 | JWT expiry times | Access: 15 min, Refresh: 7 days |
| 2 | Super-admin seed password source | Environment variable `ADMIN_SEED_PASSWORD` (documented default for dev) |
| 3 | Pagination for list users | No pagination now. Admin user count will be small. Add later if needed. |
| 4 | Redis module in shared package | Yes — `@satie/database` exports a shared `RedisModule` and `RedisService` |
| 5 | Seed email | Env var `ADMIN_SEED_EMAIL` with default `admin@satie.local` |
| 6 | Token revocation scope | Revoke BOTH access and refresh tokens on logout |
| 7 | Migration strategy | TypeORM CLI migrations, files in `apps/admin/backend/src/migrations/` |
| 8 | API response field naming | camelCase Portuguese (e.g., `idUsuarioAdmin`, `criadoAs`, `modificadoAs`) |

### Verified Assumptions

| Assumption | Verified | Notes |
|-----------|----------|-------|
| `packages/database/` does not exist | ✅ Verified | Must be created from scratch |
| `docker-compose.yml` does not exist | ✅ Verified | Must be created at repo root |
| Admin backend is a bare NestJS scaffold | ✅ Verified | Empty AppModule, single GET endpoint |
| No TypeORM/JWT/Redis packages installed | ✅ Verified | Must add all dependencies |
| pnpm-workspace.yaml includes `packages/*` | ✅ Verified | Shared packages will be discovered |
| No tsconfig path aliases for packages | ✅ Verified | Must add `@satie/database` alias to tsconfig.base.json |
| `docs/specs/domains/identity.md` does not exist | ✅ Verified | Must be created |
| E2E test setup listens on port 3000 | ✅ Verified | May need database setup for integration tests |
| Biome configured with double quotes, semicolons | ✅ Verified | Code must follow established Biome config |

## 5. Scope-to-Execution Mapping

### Slice 0 — Rename Test Projects Convention

- **Goal**: Rename existing test project directories from `*-e2e` to `*-tests` to establish the standard naming convention. This applies to `apps/admin/backend-e2e/` → `apps/admin/backend-tests/` and `apps/admin/frontend-e2e/` → `apps/admin/frontend-tests/`. Update all references in `project.json`, `jest.config.cts`, `package.json`, `tsconfig.json`, `pnpm-lock.yaml`, and root `tsconfig.json`.
- **Files to rename**:
  - `apps/admin/backend-e2e/` → `apps/admin/backend-tests/`
  - `apps/admin/frontend-e2e/` → `apps/admin/frontend-tests/`
- **Files to modify**:
  - `apps/admin/backend-tests/project.json` — update name and paths
  - `apps/admin/backend-tests/jest.config.cts` — update displayName
  - `apps/admin/backend-tests/package.json` — update name
  - `apps/admin/backend-tests/tsconfig.json` — update outDir path
  - `apps/admin/frontend-tests/package.json` — update name
  - `tsconfig.json` (root) — update project references
  - `pnpm-lock.yaml` — regenerated via `pnpm install`
- **Requirements covered**: N/A (convention alignment)
- **Acceptance covered**: Infrastructure prerequisite
- **Expected outputs**: Renamed directories, updated configs, passing `pnpm install` and existing tests

### Slice 1 — Docker Compose for Local Dev

- **Goal**: Provide local PostgreSQL 17.4 and Redis services via Docker Compose so developers can run the backend with a real database and cache.
- **Files to create**:
  - `docker-compose.yml` (repo root)
  - `.env.example` (repo root — document all env vars)
- **Requirements covered**: FR-020
- **Acceptance covered**: Infrastructure prerequisite for all other slices
- **Expected outputs**: Docker Compose file, env example, documentation

### Slice 2 — Shared Database Package (`@satie/database`)

- **Goal**: Create the `@satie/database` shared package with `EntidadeBase`, TypeORM config helper, SnakeNamingStrategy re-export, and Redis module.
- **Files to create**:
  - `packages/database/package.json`
  - `packages/database/project.json`
  - `packages/database/tsconfig.json`
  - `packages/database/tsconfig.lib.json`
  - `packages/database/src/index.ts`
  - `packages/database/src/entidade-base.ts`
  - `packages/database/src/typeorm-config.ts`
  - `packages/database/src/naming-strategy.ts`
  - `packages/database/src/redis/index.ts`
  - `packages/database/src/redis/redis.module.ts`
  - `packages/database/src/redis/redis.service.ts`
- **Files to modify**:
  - `tsconfig.base.json` — add `@satie/database` path alias
- **Requirements covered**: FR-014, FR-015, FR-016, FR-017, FR-018 (Redis module)
- **Acceptance covered**: Foundation for all entity and database operations
- **Expected outputs**: Shared package with unit tests for `EntidadeBase` and Redis service

### Slice 3 — TypeORM + Entity + Migration Setup

- **Goal**: Configure TypeORM in the Admin backend via `@satie/database`, create the `UsuarioAdmin` entity, generate the initial migration, and seed the super-admin user.
- **Files to create**:
  - `apps/admin/backend/src/identity/users/entities/usuario-admin.entity.ts`
  - `apps/admin/backend/src/migrations/<timestamp>-CreateUsuarioAdmin.ts`
  - `apps/admin/backend/src/migrations/<timestamp>-SeedSuperAdmin.ts`
  - `apps/admin/backend/src/config/typeorm.config.ts` (DataSource for CLI)
- **Files to modify**:
  - `apps/admin/backend/src/app/app.module.ts` — add TypeOrmModule.forRoot, ConfigModule
  - `apps/admin/backend/src/main.ts` — add Swagger setup
  - `apps/admin/backend/package.json` — add dependencies
- **Requirements covered**: FR-002, FR-009, FR-010, FR-011, FR-012, FR-014, FR-015, FR-016
- **Acceptance covered**: Scenario 2 AC-1 (entity exists), seed migration test
- **Expected outputs**: Working TypeORM connection, entity, migrations, Swagger UI

### Slice 4 — User CRUD Module

- **Goal**: Implement the UsersModule with controller, service, DTOs, and full CRUD operations for admin users.
- **Files to create**:
  - `apps/admin/backend/src/identity/identity.module.ts`
  - `apps/admin/backend/src/identity/users/users.module.ts`
  - `apps/admin/backend/src/identity/users/users.controller.ts`
  - `apps/admin/backend/src/identity/users/users.service.ts`
  - `apps/admin/backend/src/identity/users/dto/create-user.dto.ts`
  - `apps/admin/backend/src/identity/users/dto/update-user.dto.ts`
  - `apps/admin/backend/src/identity/users/dto/user-response.dto.ts`
- **Files to modify**:
  - `apps/admin/backend/src/app/app.module.ts` — import IdentityModule
- **Requirements covered**: FR-002, FR-003, FR-006, FR-007, FR-008, FR-009, FR-019
- **Acceptance covered**: Scenario 2, Scenario 3, Scenario 4, Scenario 5
- **Expected outputs**: CRUD endpoints, DTOs with validation, password hashing, soft delete, last-admin guard, unit tests, integration tests

### Slice 5 — JWT Authentication Module

- **Goal**: Implement the AuthModule with login, token refresh, logout, JWT strategy, Redis-backed token revocation, and JWT guard.
- **Files to create**:
  - `apps/admin/backend/src/identity/auth/auth.module.ts`
  - `apps/admin/backend/src/identity/auth/auth.controller.ts`
  - `apps/admin/backend/src/identity/auth/auth.service.ts`
  - `apps/admin/backend/src/identity/auth/dto/login.dto.ts`
  - `apps/admin/backend/src/identity/auth/dto/refresh-token.dto.ts`
  - `apps/admin/backend/src/identity/auth/dto/logout.dto.ts`
  - `apps/admin/backend/src/identity/auth/strategies/jwt.strategy.ts`
  - `apps/admin/backend/src/identity/auth/guards/jwt-auth.guard.ts`
  - `apps/admin/backend/src/identity/token-revocation/token-revocation.module.ts`
  - `apps/admin/backend/src/identity/token-revocation/token-revocation.service.ts`
- **Files to modify**:
  - `apps/admin/backend/src/identity/identity.module.ts` — import AuthModule + TokenRevocationModule
- **Requirements covered**: FR-001, FR-004, FR-005, FR-013, FR-018
- **Acceptance covered**: Scenario 1, Scenario 6
- **Expected outputs**: Auth endpoints, JWT strategy with Redis check, token revocation, guards, unit tests, integration tests

### Slice 6 — Integration, Guards, and Polish

- **Goal**: Apply JWT guard to all user CRUD endpoints, verify complete Swagger documentation, run full test suite, validate all acceptance scenarios end-to-end.
- **Files to modify**:
  - `apps/admin/backend/src/identity/users/users.controller.ts` — apply `@UseGuards(JwtAuthGuard)` to all endpoints
  - Various files for Swagger decorator completeness
- **Requirements covered**: FR-006, FR-011, FR-013
- **Acceptance covered**: All scenarios — cross-cutting auth enforcement
- **Expected outputs**: Protected endpoints, complete Swagger docs, passing integration test suite

### Slice 7 — Documentation Updates

- **Goal**: Create the identity domain spec and update architecture documentation.
- **Files to create**:
  - `docs/specs/domains/identity.md`
- **Files to modify**:
  - `docs/specs/apps/admin/architecture.md` — update with implemented module structure
- **Requirements covered**: Documentation requirements from planning
- **Expected outputs**: Domain spec, updated architecture docs

## 6. Task Generation Matrix

| Task ID | Title | Type | Plan Slice | Parallelizable | Dependencies | Requirement Refs | Technical Scope | Output File |
| ------- | ----- | ---- | ---------- | -------------- | ------------ | ---------------- | --------------- | ----------- |
| T000 | Rename test projects to *-tests convention | Task | Slice 0 | Yes | N/A | N/A (convention) | Rename backend-e2e → backend-tests, frontend-e2e → frontend-tests, update configs | docs/specs/features/001-admin-identity-domain/tasks/T000-rename-test-projects.task.spec.md |
| T001 | Docker Compose for local dev | Task | Slice 1 | Yes (parallel with T000, T002) | N/A | FR-020 | docker-compose.yml, .env.example | docs/specs/features/001-admin-identity-domain/tasks/T001-docker-compose.task.spec.md |
| T002 | Shared database package @satie/database | Task | Slice 2 | Yes (parallel with T000, T001) | N/A | FR-014, FR-015, FR-016, FR-017, FR-018 | packages/database/ with EntidadeBase, TypeORM config, Redis module | docs/specs/features/001-admin-identity-domain/tasks/T002-shared-database-package.task.spec.md |
| T003 | TypeORM setup, entity, and migrations | Task | Slice 3 | No | T001, T002 | FR-002, FR-009, FR-010, FR-012, FR-014, FR-015, FR-016 | UsuarioAdmin entity, migrations, seed, TypeORM config in AppModule | docs/specs/features/001-admin-identity-domain/tasks/T003-typeorm-entity-migration.task.spec.md |
| T004 | Admin user CRUD module | Task | Slice 4 | No | T003 | FR-002, FR-003, FR-006, FR-007, FR-008, FR-009, FR-019 | UsersModule, controller, service, DTOs, tests | docs/specs/features/001-admin-identity-domain/tasks/T004-user-crud-module.task.spec.md |
| T005 | JWT authentication module | Task | Slice 5 | No | T003, T004 | FR-001, FR-004, FR-005, FR-013, FR-018 | AuthModule, JWT strategy, token revocation, guards, tests | docs/specs/features/001-admin-identity-domain/tasks/T005-jwt-auth-module.task.spec.md |
| T006 | Integration, guards, and polish | Task | Slice 6 | No | T004, T005 | FR-006, FR-011, FR-013 | JWT guard on CRUD endpoints, Swagger completeness, full test suite | docs/specs/features/001-admin-identity-domain/tasks/T006-integration-guards-polish.task.spec.md |
| T007 | Identity domain documentation | Task | Slice 7 | Yes (parallel with T004+) | T002 | N/A (documentation) | Domain spec, architecture doc updates | docs/specs/features/001-admin-identity-domain/tasks/T007-identity-domain-docs.task.spec.md |

## 7. Task File Generation Rules

- Create one file per task under `docs/specs/features/001-admin-identity-domain/tasks/`.
- Use naming pattern: `TXXX-<short-title>.task.spec.md`.
- Use `docs/specs/templates/task.spec.md` for every task file.
- Each task must reference the same feature spec and this feature plan.
- Start each task with status `Draft`; set to `Ready` after task-level approval.
- If a task cannot be delivered safely in one branch cycle, split it before approval.
- For data-impact tasks, include explicit table, field, SQL type, nullability, default, PK/FK, constraints, indexes, and migration notes.
- For API/UI-impact tasks, include explicit request/response/UI state contracts.

## 8. Repository Impact

```text
/ (repo root)
  docker-compose.yml          (NEW)
  .env.example                (NEW)
  tsconfig.json               (MODIFIED — update test project references)
  tsconfig.base.json          (MODIFIED — add @satie/database path alias)
packages/
  database/                   (NEW — entire package)
    package.json
    project.json
    tsconfig.json
    tsconfig.lib.json
    src/
      index.ts
      entidade-base.ts
      typeorm-config.ts
      naming-strategy.ts
      redis/
        index.ts
        redis.module.ts
        redis.service.ts
apps/
  admin/
    backend/
      package.json            (MODIFIED — new dependencies)
      src/
        main.ts               (MODIFIED — Swagger setup, validation pipe)
        app/
          app.module.ts        (MODIFIED — import TypeORM, Config, IdentityModule)
        identity/              (NEW — entire domain module)
          identity.module.ts
          auth/
            auth.module.ts
            auth.controller.ts
            auth.service.ts
            dto/
              login.dto.ts
              refresh-token.dto.ts
              logout.dto.ts
            strategies/
              jwt.strategy.ts
            guards/
              jwt-auth.guard.ts
          users/
            users.module.ts
            users.controller.ts
            users.service.ts
            dto/
              create-user.dto.ts
              update-user.dto.ts
              user-response.dto.ts
            entities/
              usuario-admin.entity.ts
          token-revocation/
            token-revocation.module.ts
            token-revocation.service.ts
        migrations/
          <timestamp>-CreateUsuarioAdmin.ts
          <timestamp>-SeedSuperAdmin.ts
        config/
          typeorm.config.ts
    backend-e2e/              (RENAMED → backend-tests/)
    frontend-e2e/             (RENAMED → frontend-tests/)
docs/
  specs/
    domains/
      identity.md             (NEW)
    apps/
      admin/
        architecture.md       (MODIFIED)
    features/
      001-admin-identity-domain/
        feature.spec.md        (MODIFIED — status update)
        plan.spec.md           (NEW — this file)
        tasks/                 (Step 3 output — not created yet)
```

### Planned Changes by Area

- **Backend**: New identity domain module (auth + users + token-revocation), TypeORM configuration, migrations, Swagger setup, validation pipe, environment config
- **Shared packages**: New `@satie/database` with `EntidadeBase`, TypeORM config helper, SnakeNamingStrategy, Redis module
- **Database**: New `usuario_admin` table with audit fields, super-admin seed
- **Infrastructure**: Docker Compose (PostgreSQL 17.4 + Redis), env example file, test project renaming (`*-e2e` → `*-tests`)
- **QA**: Unit tests for services and DTOs, integration tests for all endpoints (Jest + Supertest)
- **Documentation**: Identity domain spec, updated admin architecture, updated feature spec status

## 9. Validation Strategy

- **Unit tests**:
  - `UsersService` — password hashing, last-admin guard logic, CRUD operations (mocked repository)
  - `AuthService` — login validation, token generation, refresh logic, logout revocation (mocked dependencies)
  - `TokenRevocationService` — revoke and check operations (mocked Redis)
  - DTO validation — password policy enforcement, email format
  - `EntidadeBase` — field presence and decorator verification
  - `RedisService` — connection and operations (mocked ioredis)

- **Integration tests** (Jest + Supertest, against real database and Redis via Docker Compose):
  - **Auth flow**: login with valid/invalid creds, refresh with valid/expired/revoked token, logout and verify revocation
  - **User CRUD**: create user, duplicate email conflict, weak password rejection, list users, get user by ID, update user (email + password), delete user, last-admin deletion guard
  - **Cross-cutting**: all CRUD endpoints require auth, password never returned in responses, soft-deleted users not visible in queries

- **E2E tests**: Covered by integration tests in this feature (no separate Playwright tests — backend-only feature)

- **Non-functional checks**:
  - Swagger UI accessible and documenting all endpoints
  - Redis fail-closed behavior (if Redis is down, requests are rejected)
  - Migration idempotency (seed migration safe to re-run)

## 10. Rollout and Safety

- **Feature flags**: N/A — foundational feature, no partial rollout needed.
- **Backward compatibility**: N/A — no existing functionality to break (greenfield domain).
- **Monitoring/observability**: NestJS default logging for now. No custom metrics or tracing in initial implementation (KISS — ADR-001).
- **Rollback plan**: Revert code changes and drop `usuario_admin` table via migration rollback. Remove Docker Compose services. No data to preserve (greenfield).

## 11. Step 2 Completion Checklist

- [x] Plan is drafted.
- [x] Technical design baseline is complete for all impacted layers.
- [x] Scope-to-execution mapping covers all feature requirements.
- [x] Task generation matrix is defined and consistent with scope.
- [x] Dependencies between planned slices are explicit.
- [x] Feature spec status is updated to `In Planning`.
- [x] Plan is ready to hand off to Step 3 (Task Breakdown).

## 12. Documentation Impact Mapping

| Document | Action | Reason |
|----------|--------|--------|
| `docs/specs/domains/identity.md` | Create | First domain being implemented — must document bounded context, entities, rules |
| `docs/specs/apps/admin/architecture.md` | Update | Add implemented identity module structure, connection details |
| `docs/specs/features/001-admin-identity-domain/feature.spec.md` | Update | Status transition: `Approved` → `In Planning` → `Planned` |

No new ADRs are needed — all patterns and decisions align with existing ADR-001 through ADR-005.

## 13. Post-Implementation Feedback

_(To be filled after all tasks are completed)_
