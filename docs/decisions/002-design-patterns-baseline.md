# ADR-002: Design Patterns Baseline

## Status

Accepted

## Context

The team needs practical guidance for design patterns that fit the current stack and experience level. The goal is to use a small set of patterns consistently, avoiding over-engineering while keeping code extensible.

## Decision

We adopt a small baseline of patterns and delay advanced patterns until real need appears.

### Backend (NestJS)

- **Module pattern** as primary boundary by domain.
- **Controller-Service-Repository** for request handling, business orchestration, and persistence separation.
- **DTO + validation** for explicit API contracts.
- **Adapter pattern** for external providers/integrations.

### Frontend (React)

- **Container/Presentational split (lightweight)**:
    - container components handle data orchestration,
    - presentational components focus on rendering and interaction.
- **Custom hooks** for reusable state and side-effect logic.
- **Composition over inheritance** for UI reuse.

### Shared Packages

- **Utility modules** for pure reusable logic.
- **Contract modules** for shared types/schemas used by multiple apps.

### Explicit Non-Goals (for now)

- No mandatory use of complex patterns (Factory hierarchies, CQRS/Event Sourcing, Mediator pipelines) unless required by measurable complexity.
- No pattern adoption only for theoretical purity.

## Consequences

- Faster onboarding and more consistent code organization.
- Lower risk of accidental architecture complexity.
- Some future features may require introducing additional patterns via new ADRs.
- Pattern choices remain aligned with KISS-first development.
