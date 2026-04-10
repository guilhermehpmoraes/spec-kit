# Satie Architecture Overview

## Purpose

This document gives a stable, high-level view of how Satie is organized. It is meant to explain the system boundaries, major components, and how the main layers relate to each other.

Detailed tradeoffs and significant technical choices belong in ADRs under `docs/decisions/`.

## System Context

Satie is a centralized data platform for schools. The platform consolidates school data from operational sources and makes it available through organized visualizations, dashboards, and reports.

## Guiding Principles

- Keep the source code in English, database names in Portuguese, and documentation in English.
- Prefer clear boundaries between domains instead of a single shared model for the entire system.
- Keep backend APIs explicit and stable so frontend and integrations can evolve independently.
- Record significant architectural decisions in ADRs before or alongside implementation.

## Repository Layout

```text
apps/
  <domain>/
    backend/   NestJS API for the domain
    frontend/  Vite + React UI for the domain
packages/
  shared libs, UI components, utilities, contracts
docs/
  specs/      feature, plan, and task specs
  decisions/  ADRs
```

## Logical Layers

### Presentation Layer

The frontend is responsible for user interaction, routing, page composition, and query orchestration. It should not contain domain rules that belong on the server.

### Application Layer

The backend exposes application use cases through NestJS modules, controllers, services, and repositories. This layer coordinates validation, orchestration, and persistence.

### Domain Layer

Each domain should own its rules, entities, and invariants. Shared abstractions should only live in `packages/` when they are truly cross-domain.

### Persistence Layer

PostgreSQL is the primary system of record. Database schema changes should be reviewed as part of the feature spec and tracked explicitly when they alter domain data.

## Domain Boundaries

Core domains are intentionally not finalized yet. As the product grows, each domain should get its own spec under `docs/specs/domains/` and, when needed, its own backend/frontend apps under `apps/<domain>/`.

Examples of likely domain areas include:

- School structure and hierarchy
- Students and enrollment
- Classes and schedules
- Teachers and staff
- Reports and dashboards

## Cross-Cutting Concerns

### Shared Contracts

Use shared packages for types, validation schemas, UI primitives, and utility functions that are reused across apps.

### API Contracts

Backend and frontend should agree on request and response shapes through shared contracts where it reduces duplication and mismatch risk.

### Testing

Testing should follow the stack already defined in the project:

- Backend: Jest and Supertest
- Frontend: Vitest and React Testing Library
- End-to-end: Playwright

## Data and Integration

The platform should treat PostgreSQL as the authoritative store for operational data modeled inside Satie. External systems, imports, and synchronization flows should be documented per feature or domain as they are introduced.

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

Typical examples:

- introducing a new framework or package
- changing module boundaries
- adopting a new data synchronization strategy
- selecting a caching, messaging, or reporting approach

## Non-Functional Expectations

The architecture should continue to make room for:

- clear authorization boundaries
- observable backend behavior through logs and metrics
- predictable performance for dashboard and reporting flows
- accessible frontend experiences
- maintainable migrations and rollback paths

## Evolution Rules

- Update this document when the system structure changes materially.
- Use ADRs for the why behind major decisions.
- Use feature specs for the what and acceptance criteria.
- Use implementation plans and tasks for the how.

## Open Questions

- Which initial domains should be formalized first?
- Which data ingestion strategy will become the default?
- What should be shared across all domains versus kept local to each app?
