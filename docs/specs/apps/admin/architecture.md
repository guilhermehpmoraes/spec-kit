# Admin — Application Architecture

## Overview

Admin is the internal administrative platform for managing clients, products, features, and usage metrics across all platforms in the monorepo.

## Domains

Each domain has a corresponding spec in `docs/specs/domains/`.

- **Identity** — admin user management (CRUD) and JWT-based authentication. Modules: `IdentityModule` (boundary), `UsersModule`, `AuthModule`, `TokenRevocationModule`. See [domain spec](../../domains/identity.md) and [feature spec](../features/001-admin-identity-domain/feature.spec.md).
- **Subscription** — product/permission catalog and plan management. Modules: `SubscriptionModule` (boundary), `ProductsModule`. See [domain spec](../../domains/subscription.md) and [feature spec](../features/003-admin-subscription-domain/feature.spec.md).
- **Educational Group** — educational group (tenant/client) lifecycle management, master user provisioning, plan subscriptions, and common users. Modules: `EducationalGroupModule` (boundary), `GroupsModule`, `SubscriptionsModule`, `CommonUsersModule`. See [domain spec](../../domains/educational-group.md) and [feature spec](../features/004-admin-educational-group-domain/feature.spec.md).
- **Client Management** — onboarding, configuration, and monitoring of client accounts. _(Planned)_

## Backend Structure

### Project Layout

- **Source**: `apps/admin/backend/` — NestJS API
- **Tests**: `apps/admin/backend-tests/` — separate Nx project with `implicitDependencies` on `backend`
- **Build Output**: `dist/apps/admin/backend/` — centralized at workspace root

### Test Project TypeScript Strategy

`backend-tests/` imports backend source files directly via relative paths, following the monorepo-wide test TypeScript configuration pattern defined in ADR-007. See [ADR-007](../../../decisions/007-test-project-typescript-configuration.md) for the full configuration template, rules, and rationale.

Unit tests use a separate Jest config (`jest.unit.config.js`) with `moduleNameMapper` for `@satie/database` resolution, while e2e tests use `jest.config.cts` with global setup/teardown.

### Module Hierarchy

```
src/
├── main.ts                          # Bootstrap + Scalar API docs setup
├── app/
│   ├── app.module.ts                # Root module — imports all domain modules + TypeORM + Config
│   ├── app.controller.ts            # Health check
│   └── app.service.ts               # Health check
├── identity/
│   ├── identity.module.ts           # Domain boundary module — imports Auth + Users
│   ├── auth/
│   │   ├── auth.module.ts           # Auth module — JWT, Passport, Redis
│   │   ├── auth.controller.ts       # Login, refresh, logout endpoints
│   │   ├── auth.service.ts          # Auth business logic
│   │   ├── dto/
│   │   │   ├── login.dto.ts             # LoginDto (email, senha)
│   │   │   ├── login-response.dto.ts    # LoginResponseDto (accessToken, refreshToken)
│   │   │   ├── refresh-token.dto.ts     # RefreshTokenDto (refreshToken)
│   │   │   ├── refresh-response.dto.ts  # RefreshResponseDto (accessToken)
│   │   │   └── logout.dto.ts            # LogoutDto (refreshToken)
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
├── subscription/
│   ├── subscription.module.ts               # Domain boundary module — imports Products + Plans
│   ├── products/
│   │   ├── products.module.ts               # Products module — entities, controller, service
│   │   ├── products.controller.ts           # Read-only endpoints for /produtos
│   │   ├── products.service.ts              # Product/permission read operations
│   │   ├── dto/
│   │   │   ├── permission-response.dto.ts   # PermissionResponseDto
│   │   │   ├── product-response.dto.ts      # ProductResponseDto
│   │   │   └── product-detail-response.dto.ts # ProductDetailResponseDto
│   │   └── entities/
│   │       ├── produto.entity.ts            # Produto TypeORM entity
│   │       └── permissao.entity.ts          # Permissao TypeORM entity
│   └── plans/
│       ├── plans.module.ts                  # Plans module — CRUD, permission management
│       ├── plans.controller.ts              # CRUD endpoints for /planos
│       ├── plans.service.ts                 # Plan CRUD, cross-product validation, transactions
│       ├── dto/
│       │   ├── create-plan.dto.ts            # CreatePlanDto (rich payload with product + permissions)
│       │   ├── update-plan.dto.ts            # UpdatePlanDto (standalone, excludes idProduto)
│       │   ├── plan-list-response.dto.ts     # PlanListResponseDto (with permissions)
│       │   └── plan-detail-response.dto.ts   # PlanDetailResponseDto (with product + permissions)
│       └── entities/
│           ├── plano.entity.ts              # Plano TypeORM entity
│           └── plano-permissao.entity.ts    # PlanoPermissao TypeORM entity
├── educational-group/
│   ├── educational-group.module.ts          # Domain boundary module — imports Groups + Subscriptions + CommonUsers
│   ├── entities/
│   │   ├── grupo-educacional.entity.ts      # GrupoEducacional TypeORM entity
│   │   ├── usuario-mestre.entity.ts         # UsuarioMestre TypeORM entity (1:1 with group)
│   │   ├── parametros-grupo-educacional.entity.ts # ParametrosGrupoEducacional TypeORM entity (1:1 with group)
│   │   ├── usuario-comum.entity.ts          # UsuarioComum TypeORM entity (N:1 with group)
│   │   └── assinatura.entity.ts             # Assinatura TypeORM entity (N:1 with group, FK to plano)
│   ├── groups/
│   │   ├── groups.module.ts                 # Groups module — CRUD with atomic creation
│   │   ├── groups.controller.ts             # CRUD endpoints for /grupos-educacionais
│   │   ├── groups.service.ts                # Group business logic (atomic creation, cascade delete)
│   │   └── dto/
│   │       ├── create-group.dto.ts          # CreateGroupDto (descricao, email, planoIds)
│   │       ├── update-group.dto.ts          # UpdateGroupDto (descricao only)
│   │       ├── group-response.dto.ts        # GroupResponseDto (creation response)
│   │       ├── group-list-response.dto.ts   # GroupListResponseDto (list response)
│   │       └── group-detail-response.dto.ts # GroupDetailResponseDto (detail with relations)
│   ├── subscriptions/
│   │   ├── subscriptions.module.ts          # Subscriptions module — independent plan management
│   │   ├── subscriptions.controller.ts      # Endpoints for /assinaturas
│   │   ├── subscriptions.service.ts         # Subscription business logic (replace plans, 1-per-product validation)
│   │   └── dto/
│   │       ├── update-subscriptions.dto.ts  # UpdateSubscriptionsDto (planoIds)
│   │       └── subscription-response.dto.ts # SubscriptionResponseDto
│   └── common-users/
│       ├── common-users.module.ts           # Common users module — independent user management
│       ├── common-users.controller.ts       # CRUD endpoints for /usuarios-comuns
│       ├── common-users.service.ts          # Common user business logic
│       └── dto/
│           ├── create-common-user.dto.ts    # CreateCommonUserDto (email, idGrupoEducacional)
│           ├── update-common-user.dto.ts    # UpdateCommonUserDto (email, senha)
│           └── common-user-response.dto.ts  # CommonUserResponseDto
└── config/
    └── typeorm.config.ts            # TypeORM DataSource config for CLI migrations
```

### Database Conventions

- All entities inherit from `EntidadeBase` (`@satie/database`) — provides audit fields and soft deletes (see ADR-005).
- TypeORM with `SnakeNamingStrategy` enforces snake_case for all database names.
- Entity classes are named in Portuguese (e.g., `UsuarioAdmin`).
- Soft deletes are the default — no physical deletes in normal operations.

### Infrastructure Dependencies

- **Docker Compose** — `docker-compose.yml` at the repo root provides local development services
- **PostgreSQL 17.4** — primary data store (port 5432, user/password: `satie`, database: `satie_dev`)
- **Redis (latest)** — token revocation blacklist, future caching (port 6379)

### Shared Packages

- `@satie/database` — base entity (`EntidadeBase`), TypeORM config (`createTypeOrmConfig`), naming strategy (`SnakeNamingStrategy`), Redis module (`RedisModule`, `RedisService`)

## Frontend Structure

### Project Layout

- **Source**: `apps/admin/frontend/` — Vite + React UI
- **Tests**: `apps/admin/frontend-tests/` — separate Nx project
- **Build Output**: `dist/apps/admin/frontend/` — centralized at workspace root

> _Document routing, state management, and component patterns specific to Admin._

## Shared Packages

| Package | Test Project | Description |
|---------|-------------|-------------|
| `@satie/database` | `packages/database-tests/` | Base entity, TypeORM config, naming strategy, Redis module |

## API Documentation Standard

All NestJS backends serve API documentation via [Scalar](https://scalar.com/) at `/api/docs`, using `@scalar/nestjs-api-reference`. The OpenAPI spec is generated by `@nestjs/swagger`; Scalar renders it with a modern UI, built-in request testing, and persistent authentication.

### Configuration

- Bearer Auth is pre-selected via `preferredSecurityScheme: "bearer"`
- Token persistence is enabled via `persistAuth: true` — developers authenticate once and the token survives page reloads
- A step-by-step authentication guide is included in `DocumentBuilder.setDescription()`

### Decorator Standard

- Every `@ApiProperty` must include `description` and `example`
- Every non-void endpoint must have a typed `@ApiResponse({ type: ResponseDto })`
- Response DTOs must be dedicated classes (not inline types)

See [ADR-010](../../../decisions/010-scalar-api-documentation.md) for the full decision rationale.

## References

- Repo-wide architecture: [docs/architecture.md](../../../architecture.md)
- Identity domain: [docs/specs/domains/identity.md](../../domains/identity.md)
- Subscription domain: [docs/specs/domains/subscription.md](../../domains/subscription.md)
- Educational Group domain: [docs/specs/domains/educational-group.md](../../domains/educational-group.md)
- ADR-002: [Design Patterns Baseline](../../../decisions/002-design-patterns-baseline.md)
- ADR-004: [Domain-Driven Design Baseline](../../../decisions/004-domain-driven-design-baseline.md)
- ADR-005: [Database Conventions and Shared Package](../../../decisions/005-database-conventions-shared-package.md)
- ADR-007: [Test Project TypeScript Configuration](../../../decisions/007-test-project-typescript-configuration.md)
- ADR-010: [Scalar as Standard API Documentation UI](../../../decisions/010-scalar-api-documentation.md)
