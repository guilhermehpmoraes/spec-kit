# ADR-004: Domain-Driven Design Baseline and Multi-Platform Organization

## Status

Accepted

## Context

The repository hosts multiple platforms (applications), starting with Admin and Satie. The initial documentation and architecture framed everything around Satie as the primary product. As the project grows, we need:

1. A repo-wide architecture that is platform-agnostic — not centered on a single product.
2. A clear distinction between **applications** (platforms) and **domains** (DDD bounded contexts).
3. A consistent way to document, spec, and implement domain-scoped work across the codebase.

## Decision

### Applications vs. Domains

- **Applications** are the deployable platforms: `apps/admin/`, `apps/satie/`, etc. Each has its own `backend/` and `frontend/`.
- **Domains** follow Domain-Driven Design principles. Each domain is a bounded context that encapsulates a coherent problem area within an application.
- Domains are **scoped to a single application**. A domain does not span multiple apps. If two apps share concepts, they do so through shared contracts in `packages/`, not by sharing a domain.

### Code Organization

- Each application decides its own internal code organization for domains. There is no enforced folder convention across apps.
- Shared abstractions live in `packages/` only when they are truly cross-application.

### Documentation Organization

- **Repo-wide architecture**: `docs/architecture.md` — covers the entire monorepo, layers, and cross-cutting concerns.
- **Per-app architecture**: `docs/specs/apps/<app>/architecture.md` — detailed architecture for each application.
- **Domain specs**: `docs/specs/domains/<domain-name>.md` — one spec per domain, describing its bounded context, responsibilities, entities, and rules.
- **Feature specs**: reference their parent domain via the `Domain` field in the feature spec header.

### Domain Spec Lifecycle

- Domains are documented in `docs/specs/domains/` before or alongside the first feature that touches them.
- Feature specs declare which domain they belong to (`Feature -> Domain` linking).
- Domain specs are living documents that evolve as features are delivered.

## Consequences

- Architecture documentation becomes platform-agnostic and scales to N applications.
- Each team or application can organize domain code internally without a repo-wide convention.
- Domain boundaries are explicit and traceable from specs to code.
- Feature specs gain clearer context by linking to their parent domain.
- Per-app architecture specs allow depth without cluttering the repo-wide overview.
