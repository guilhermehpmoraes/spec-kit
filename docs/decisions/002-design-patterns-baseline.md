# ADR-002: Design Patterns Baseline

## Status

Accepted

## Context

The team needs practical guidance for design patterns that fit the active stack profile and experience level. The goal is to use a small set of patterns consistently, avoiding over-engineering while keeping code extensible.

## Decision

We adopt a small baseline of patterns and delay advanced patterns until real need appears.

### Backend (Framework-Agnostic)

- **Domain/module boundaries** as primary organization strategy.
- **Request/Application/Persistence separation** (for example Controller-Service-Repository where applicable).
- **Explicit contracts + validation** for input/output boundaries.
- **Adapter pattern** for external providers/integrations.

### Frontend (Framework-Agnostic)

- **Data/Presentation split (lightweight)**:
    - container components handle data orchestration,
    - presentational components focus on rendering and interaction.
- **Reusable state/effect abstractions** aligned with the frontend framework.
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
