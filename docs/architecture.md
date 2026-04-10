# Project Architecture Overview

## Purpose

This document gives a stable, high-level view of how the target project is organized. It explains system boundaries, major components, and how the main layers relate.

Detailed tradeoffs and significant technical choices belong in ADRs under `docs/decisions/`.

## System Context

Describe the product/platform context at a high level: who uses it, what core problem it solves, and what core capabilities it must provide.

## Guiding Principles

- Keep naming/language conventions explicit and documented in the project spec.
- Prefer clear boundaries between domains instead of a single shared model for the entire system.
- Keep backend APIs explicit and stable so frontend and integrations can evolve independently.
- Record significant architectural decisions in ADRs before or alongside implementation.

## Repository Layout

```text
apps/
  <domain>/
    backend/   API/service layer for the domain
    frontend/  UI/application layer for the domain
packages/
  shared libs, UI components, utilities, contracts
docs/
  specs/      feature, plan, and task specs
  decisions/  ADRs
```

## Logical Layers

### Presentation Layer

The frontend/presentation layer is responsible for user interaction, routing, page composition, and query orchestration. It should not contain domain rules that belong in backend/domain services.

### Application Layer

The backend/application layer exposes use cases through the project's chosen framework and architecture style. This layer coordinates validation, orchestration, and persistence.

### Domain Layer

Each domain should own its rules, entities, and invariants. Shared abstractions should only live in `packages/` when they are truly cross-domain.

### Persistence Layer

The primary system of record must be defined in the project spec. Data schema changes should be reviewed as part of the feature spec and tracked explicitly when they alter domain data.

## Domain Boundaries

Core domains are intentionally not finalized yet. As the product grows, each domain should get its own spec under `docs/specs/domains/` and, when needed, its own backend/frontend apps under `apps/<domain>/`.

Examples of likely domain areas include:

- Identity and access
- Core business entities
- Operational workflows
- Reporting and analytics
- Integrations

## Cross-Cutting Concerns

### Shared Contracts

Use shared packages for types, validation schemas, UI primitives, and utility functions that are reused across apps.

### API Contracts

Backend and frontend should agree on request and response shapes through shared contracts where it reduces duplication and mismatch risk.

### Testing

Testing should follow the active stack profile defined for the target project:

- Unit tests: [framework]
- Integration tests: [framework]
- End-to-end tests: [framework]

## Data and Integration

The platform should treat the selected primary data store as the authoritative source for operational data. External systems, imports, and synchronization flows should be documented per feature or domain as they are introduced.

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
- What stack profile constraints are mandatory versus optional for this project?
