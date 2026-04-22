# Domain Spec: Identity

## Overview

- **Domain Name**: Identity
- **Bounded Context**: Admin user management and authentication
- **Application**: Admin

## Responsibilities

- Admin user lifecycle (CRUD with soft delete)
- Password management (hashing with bcrypt, policy enforcement: 8+ chars, uppercase, number, special character)
- JWT-based authentication (login, refresh, logout)
- Token revocation via Redis blacklist (TTL-based auto-cleanup)

## Entities

| Entity | Table | Inherits | Description |
|--------|-------|----------|-------------|
| `UsuarioAdmin` | `usuario_admin` | `EntidadeBase` | Admin user account with email and hashed password |

## Key Modules

| Module | Role |
|--------|------|
| `IdentityModule` | Domain boundary module — imports Auth, Users, and TokenRevocation |
| `UsersModule` | Admin user CRUD operations |
| `AuthModule` | JWT authentication (login, refresh, logout) |
| `TokenRevocationModule` | Redis-backed token blacklist for logout/revocation |

## API Surface

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/auth/login` | POST | Authenticate admin user, return access + refresh tokens |
| `/auth/refresh` | POST | Exchange refresh token for a new access token |
| `/auth/logout` | POST | Invalidate tokens via Redis blacklist |
| `/usuarios-admin` | POST | Create a new admin user |
| `/usuarios-admin` | GET | List all admin users |
| `/usuarios-admin/:id` | GET | Get a single admin user |
| `/usuarios-admin/:id` | PATCH | Update an admin user |
| `/usuarios-admin/:id` | DELETE | Soft-delete an admin user |

## Infrastructure Dependencies

- **PostgreSQL** (via `@satie/database`) — primary data store for the `usuario_admin` table
- **Redis** (via `@satie/database` RedisModule) — token revocation blacklist with TTL-based auto-cleanup

## Related Decisions

- [ADR-002 — Design Patterns Baseline](../../decisions/002-design-patterns-baseline.md)
- [ADR-004 — Domain-Driven Design Baseline](../../decisions/004-domain-driven-design-baseline.md)
- [ADR-005 — Database Conventions and Shared Package](../../decisions/005-database-conventions-shared-package.md)

## Related Features

- [001-admin-identity-domain](../features/001-admin-identity-domain/feature.spec.md)

## Future Considerations

- Role-based access control (RBAC)
- Password recovery / reset flow
- Multi-factor authentication (MFA)
- Integration with external identity providers (OAuth, SAML)

All of the above are explicitly out of scope for the initial implementation.
