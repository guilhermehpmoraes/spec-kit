# Domain Spec: Educational Group

## Overview

- **Domain Name**: Educational Group
- **Bounded Context**: Educational group (tenant/client) lifecycle management, master user provisioning, database parameter generation, plan subscription management, and common user management
- **Application**: Admin

## Responsibilities

- Educational group lifecycle management (CRUD with soft delete)
- Automatic provisioning of master user credentials (username and hashed password) based on group description and environment
- Automatic generation of database name parameters based on group description and environment
- Plan subscription management with 1-plan-per-product constraint enforcement
- Common user management within educational groups
- Atomic creation of educational group with all related entities in a single transaction

## Entities

| Entity | Table | Inherits | Description |
|--------|-------|----------|-------------|
| `GrupoEducacional` | `grupo_educacional` | `EntidadeBase` | Core tenant entity representing an educational group/school |
| `UsuarioMestre` | `usuario_mestre` | `EntidadeBase` | Auto-provisioned master user credentials per group (1:1) |
| `ParametrosGrupoEducacional` | `parametros_grupo_educacional` | `EntidadeBase` | Configuration parameters (database name) for a group (1:1) |
| `UsuarioComum` | `usuario_comum` | `EntidadeBase` | Common user accounts within a group (N:1) |
| `Assinatura` | `assinatura` | `EntidadeBase` | Link between a group and subscription plans (N:1, cross-domain FK to `plano`) |

## Key Modules

| Module | Role |
|--------|------|
| `EducationalGroupModule` | Domain boundary module — imports Groups, Subscriptions, CommonUsers |
| `GroupsModule` | Educational group CRUD with atomic creation of related entities |
| `SubscriptionsModule` | Independent plan subscription management per group |
| `CommonUsersModule` | Independent common user management per group |

## API Surface

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/grupos-educacionais` | POST | Create educational group with master user, database params, first common user, and subscriptions |
| `/grupos-educacionais` | GET | List all educational groups |
| `/grupos-educacionais/:id` | GET | Get educational group with full details (master user, params, subscriptions, common users) |
| `/grupos-educacionais/:id` | PATCH | Update educational group (description only) |
| `/grupos-educacionais/:id` | DELETE | Soft-delete educational group and cascade to related records |
| `/assinaturas?idGrupoEducacional=:id` | GET | List subscriptions for a group |
| `/assinaturas/:idGrupoEducacional` | PATCH | Replace plan subscriptions for a group |
| `/usuarios-comuns` | POST | Create a common user within a group |
| `/usuarios-comuns?idGrupoEducacional=:id` | GET | List common users for a group |
| `/usuarios-comuns/:id` | GET | Get a common user by ID |
| `/usuarios-comuns/:id` | PATCH | Update a common user (email, password) |
| `/usuarios-comuns/:id` | DELETE | Soft-delete a common user |

## Business Rules

- **Auto-provisioning naming**: Master username format `mestre-{env}-{slug}`, database name format `db_{env}_{slug}`, where `{slug}` is the Unicode-normalized, diacritics-stripped, kebab-case version of the group description and `{env}` comes from `APP_ENVIRONMENT` env var (values: `dev` | `prod`, default `dev`).
- **Password policy**: Auto-generated passwords must contain at least one uppercase letter, one lowercase letter, one digit, and one special character.
- **Immutable fields**: `usuario_mestre.usuario` and `parametros_grupo_educacional.nome_banco` are set at creation and cannot be updated.
- **1-plan-per-product constraint**: A group may not have more than one `assinatura` referencing plans from the same product (`plano.idProduto`). Enforced at application level on creation and on subscription replacement.
- **Atomic creation**: `grupo_educacional`, `usuario_mestre`, `parametros_grupo_educacional`, first `usuario_comum`, and all `assinatura` records are created in a single database transaction.
- **Cascade soft-delete**: Deleting a group soft-deletes all related `assinatura`, `usuario_comum`, `parametros_grupo_educacional`, and `usuario_mestre` records in a transaction.
- **`verificado` field**: `usuario_comum.verificado` is always `false` at creation. Email verification flow is out of scope for this domain.

## Infrastructure Dependencies

- **PostgreSQL** (via `@satie/database`) — primary data store for all educational group tables
- **`@satie/utils`** — slug generation, random password generation, environment-based naming utilities
- **Identity domain** — JWT authentication guard reuse
- **Subscription domain** — cross-domain FK from `assinatura` to `plano` table (read-only dependency on plan data)
- **`APP_ENVIRONMENT`** — environment variable (`dev` | `prod`) consumed via `ConfigService` to derive master username and database name prefixes

## Related Decisions

- [ADR-004 — Domain-Driven Design Baseline](../../decisions/004-domain-driven-design-baseline.md)
- [ADR-005 — Database Conventions and Shared Package](../../decisions/005-database-conventions-shared-package.md)
- [ADR-010 — Scalar API Documentation](../../decisions/010-scalar-api-documentation.md)

## Related Features

- [004-admin-educational-group-domain](../features/004-admin-educational-group-domain/feature.spec.md)
