# Domain Spec: Subscription

## Overview

- **Domain Name**: Subscription
- **Bounded Context**: Product catalog, permission management, and subscription plan composition
- **Application**: Admin

## Responsibilities

- Maintain the product and permission catalog (read-only from API; managed via migrations)
- Plan lifecycle management (CRUD with soft delete)
- Plan-product association management
- Permission selection per plan-product association with cross-product validation

## Entities

| Entity | Table | Inherits | Description |
|--------|-------|----------|-------------|
| `Produto` | `produto` | `EntidadeBase` | A platform product (e.g., Satie) |
| `Permissao` | `permissao` | `EntidadeBase` | A capability/feature within a product |
| `Plano` | `plano` | `EntidadeBase` | A subscription tier for a specific product (FK to `produto`) |
| `PlanoPermissao` | `plano_permissao` | `EntidadeBase` | Associates specific permissions with a plan |

## Key Modules

| Module | Role |
|--------|------|
| `SubscriptionModule` | Domain boundary module — imports Products, Plans |
| `ProductsModule` | Product and permission read-only operations |
| `PlansModule` | Plan CRUD and plan-product-permission management |

## API Surface

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/produtos` | GET | List all products |
| `/produtos/:id` | GET | Get product by ID with permissions |
| `/produtos/:idProduto/permissoes` | GET | List permissions for a product |
| `/planos` | POST | Create a new plan (with product + permissions) |
| `/planos` | GET | List all plans |
| `/planos/:id` | GET | Get plan with product and permissions |
| `/planos/:id` | PATCH | Update plan (may include permissions replacement) |
| `/planos/:id` | DELETE | Soft-delete a plan |

## Infrastructure Dependencies

- **PostgreSQL** (via `@satie/database`) — primary data store for all subscription tables
- **Identity domain** — JWT authentication guard reuse

## Related Decisions

- ADR-004: Domain-Driven Design Baseline
- ADR-005: Database Conventions and Shared Package
- ADR-010: Scalar API Documentation
