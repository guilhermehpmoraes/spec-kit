# Architecture Overview

## Purpose

This document gives a stable, high-level view of how the monorepo is organized. It covers the system boundaries, major components, and how the main layers relate to each other across all applications.

Detailed tradeoffs and significant technical choices belong in ADRs under `docs/decisions/`. Each application has its own detailed architecture spec at `docs/specs/apps/<app>/architecture.md`.

## System Context

This monorepo hosts multiple platforms (applications). Each platform is a full-stack product with its own backend and frontend. The first two applications are:

| Application | Description |
|-------------|-------------|
| **Admin** | Internal administrative platform — client, product, and feature management; usage dashboards and metrics. |
| **Satie** | Centralized data platform for schools — school structure visualization, dashboards, and reports. |

## Guiding Principles

- Keep source code in English, database names in Portuguese, and all documentation (specs, ADRs, prompts, skills) in English.
- Prefer clear boundaries between domains instead of a single shared model.
- Keep backend APIs explicit and stable so frontend and integrations evolve independently.
- Record significant architectural decisions in ADRs before or alongside implementation.
- Follow Domain-Driven Design to organize problem spaces within each application (see ADR-004).

## Repository Layout

```text
apps/
  <application>/
    backend/       NestJS API for the application
    backend-tests/ Tests for the backend
    frontend/      Vite + React UI for the application
    frontend-tests/ Tests for the frontend
packages/
  database/        @satie/database — shared base entity, TypeORM config, naming strategy
  database-tests/  Unit/integration tests for @satie/database
  shared libs, UI components, utilities, contracts
dist/
  apps/<application>/backend/    Backend build output
  apps/<application>/frontend/   Frontend build output
  packages/<package>/            Package build output
docs/
  architecture.md          repo-wide architecture (this file)
  project.spec.md          high-level project spec
  specs/
    apps/<app>/            per-app detailed architecture and specs
    domains/               domain specs (DDD bounded contexts)
    features/<feature-id>/ feature, plan, and task specs
    templates/             spec templates
  decisions/               ADRs
```

## Conventions

### Dependency Management (Single Version Policy)

All third-party dependencies are declared exclusively in the root `package.json`. Sub-project `package.json` files contain only metadata (`name`, `version`, `private`, entry points, and Nx config) — no `dependencies`, `devDependencies`, or `peerDependencies` for external packages.

This ensures:

- A single source of truth for all dependency versions across the monorepo.
- No nested `node_modules` inside sub-projects.
- Consistent runtime behavior — all projects resolve the same version of every package.

When adding a new dependency, add it to the root `package.json` and run `pnpm install`. Nx traces actual source code imports to determine the project dependency graph, so `package.json` declarations in sub-projects are not needed for Nx to work correctly.

See ADR-006 (`docs/decisions/006-single-version-dependency-policy.md`) for rationale and trade-offs.

### Project Naming (ADR-008)

All projects follow a consistent naming scheme:

- **`package.json` names** use the `@satie/` scope: `@satie/admin-backend`, `@satie/admin-frontend`, `@satie/database`, etc.
- **`project.json` names** (used by Nx) drop the scope but include the application prefix: `admin-backend`, `admin-frontend`, `database`, etc.
- **All Nx configuration** (`targets`, `implicitDependencies`, `tags`, etc.) lives exclusively in `project.json` — never in a `package.json` `nx` key.
- **Cross-references** in Nx config use the Nx project name (e.g., `admin-backend:build`, not `@satie/admin-backend:build`).

When adding a new application or package, follow these patterns. See ADR-008 (`docs/decisions/008-project-naming-conventions.md`) for the full naming table and rationale.

### Test Project Separation

Tests are kept in dedicated sibling projects rather than alongside source code:

| Source Project | Test Project |
|----------------|--------------|
| `apps/<app>/backend` | `apps/<app>/backend-tests` |
| `apps/<app>/frontend` | `apps/<app>/frontend-tests` |
| `packages/<lib>` | `packages/<lib>-tests` |

Test projects declare an `implicitDependencies` on their source project in `project.json`, ensuring Nx runs tests when the source changes.

#### Test Project TypeScript Configuration

Test projects import source code directly from their sibling source project via relative paths (e.g., `../<source>/src/**/*.ts`). This gives tests full access to internal types without an intermediate build step. To make this work, every test project's `tsconfig.json` applies a set of mandatory overrides:

| Setting | Value | Reason |
|---------|-------|--------|
| `rootDir` | `".."` | Common ancestor covering both source and test directories. |
| `composite` | `false` | Test projects are leaf nodes — not referenced by other projects. |
| `declaration`, `declarationMap`, `emitDeclarationOnly` | `false` | No declaration output needed. |
| `noEmit` | `true` | Type-checking only; SWC (Jest) or Vite (Vitest) transpiles at runtime. |
| `experimentalDecorators`, `emitDecoratorMetadata` | `true` | Required when source uses NestJS/TypeORM decorators (backend tests). |
| `strictPropertyInitialization` | `false` | Avoids false positives on decorator-initialized DTOs and entities. |
| `types` | `["jest", "node"]` or `["vitest", "node"]` | Exposes test framework globals. |

**Important:** Test files must **not** import `describe`, `it`, `beforeEach`, or `afterEach` from `node:test`. Jest and Vitest provide these as globals. Importing from `node:test` shadows them and breaks `expect`/`jest`/`vi`.

See ADR-007 (`docs/decisions/007-test-project-typescript-configuration.md`) for full rationale, per-project-type variations, and rules.

### Centralized Build Output

All build artifacts are output to the workspace root `dist/` directory, mirroring the source layout:

- Backend: `dist/apps/<application>/backend/`
- Frontend: `dist/apps/<application>/frontend/`
- Packages: `dist/packages/<package>/`

This keeps source directories clean and simplifies CI artifact collection. Configure `outDir` in `tsconfig.app.json` / `tsconfig.lib.json` and build tool configs (webpack, vite) to point to the root `dist/` path.

## Applications and Domains

### Applications

Applications are the deployable platforms. Each application has its own `backend/` and `frontend/` under `apps/<application>/`. Applications are not domains — they are products that contain multiple domains.

### Domains (DDD Bounded Contexts)

Domains follow Domain-Driven Design principles. Each domain is a bounded context that encapsulates a coherent problem area within an application. Key rules:

- **Domains are scoped to a single application.** A domain does not span multiple apps.
- **Each application organizes its own domain code internally.** There is no enforced folder convention across apps.
- **Domain specs** are documented in `docs/specs/domains/` and linked from feature specs for development context.
- If two applications share concepts, they do so through shared contracts in `packages/`, not by sharing a domain.

### Documentation Hierarchy

| Level | Location | Purpose |
|-------|----------|---------|
| Repo-wide | `docs/architecture.md` | System boundaries, layers, cross-cutting concerns |
| Per-app | `docs/specs/apps/<app>/architecture.md` | Internal architecture, domain map, conventions |
| Domain | `docs/specs/domains/<domain>.md` | Bounded context, entities, rules, invariants |
| Feature | `docs/specs/features/<feature-id>/` | Spec, plan, tasks for a specific feature |

## Logical Layers

### Presentation Layer

The frontend handles user interaction, routing, page composition, and query orchestration. It should not contain domain rules that belong on the server.

### Application Layer

The backend exposes use cases through NestJS modules, controllers, services, and repositories. This layer coordinates validation, orchestration, and persistence.

### Domain Layer

Each domain owns its rules, entities, and invariants. Shared abstractions live in `packages/` only when they are truly cross-application.

### Persistence Layer

PostgreSQL is the primary system of record. Database schema changes should be reviewed as part of the feature spec and tracked explicitly when they alter domain data.

All entities inherit from `EntidadeBase` (defined in `@satie/database`) which provides audit fields (`criado_por`, `modificado_por`, `criado_as`, `modificado_as`) and soft-delete fields (`deletado_as`, `deletado_por`). Tables are never physically deleted in normal operations.

TypeORM with `SnakeNamingStrategy` enforces snake_case for all table and column names automatically. Entity classes are named in Portuguese for consistency with the database schema (see ADR-005).

Redis is available for caching and ephemeral data (e.g., token revocation blacklists).

## Cross-Cutting Concerns

### Shared Contracts

Use shared packages for types, validation schemas, UI primitives, and utility functions reused across applications.

### Shared Database Package (`@satie/database`)

The `packages/database/` package provides the foundational database layer shared by all applications:

- `EntidadeBase` — auditable base entity with soft deletes (see ADR-005)
- TypeORM configuration utilities and `SnakeNamingStrategy` setup
- All backends import this package for database foundations; app-specific entities and migrations stay in each app

### API Contracts

Backend and frontend should agree on request and response shapes through shared contracts where it reduces duplication and mismatch risk.

### Testing

Testing follows the stack defined in the project:

- Backend: Jest and Supertest
- Frontend: Vitest and React Testing Library
- End-to-end: Playwright

## Data and Integration

PostgreSQL is the authoritative store for operational data. Redis is used for ephemeral data such as token revocation and caching. External systems, imports, and synchronization flows should be documented per feature or domain as they are introduced.

When a feature changes data shape or ownership, the corresponding spec should document:

- affected entities
- migration impact
- consistency rules
- rollback considerations

## Architectural Decision Process

Any significant choice that changes the system shape, introduces a new dependency, or establishes a long-lived pattern should be captured in an ADR.

Current baseline ADRs:

- `docs/decisions/001-engineering-principles.md`
- `docs/decisions/002-design-patterns-baseline.md`
- `docs/decisions/003-biome-quality-tooling-baseline.md`
- `docs/decisions/004-domain-driven-design-baseline.md`
- `docs/decisions/005-database-conventions-shared-package.md`
- `docs/decisions/006-single-version-dependency-policy.md`
- `docs/decisions/007-test-project-typescript-configuration.md`
- `docs/decisions/008-project-naming-conventions.md`

## Non-Functional Expectations

The architecture should continue to make room for:

- clear authorization boundaries
- observable backend behavior through logs and metrics
- predictable performance for dashboard and reporting flows
- accessible frontend experiences
- maintainable migrations and rollback paths

## Evolution Rules

- Update this document when the system structure changes materially.
- Update per-app architecture specs when application-specific structure changes.
- Use ADRs for the why behind major decisions.
- Use feature specs for the what and acceptance criteria.
- Use implementation plans and tasks for the how.

## Open Questions

- Which initial domains should be formalized first for each application?
- Which data ingestion strategy will become the default?
- What should be shared across all applications versus kept local to each app?
