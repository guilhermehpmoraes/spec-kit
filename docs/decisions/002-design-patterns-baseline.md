# ADR-002: Design Patterns Baseline

## Status

Accepted

## Context

The team needs practical guidance for design patterns that can survive changes in language, framework, and repo shape. The goal is to use a small set of patterns consistently, avoid over-engineering, and still leave room for stack-specific adaptation.

## Decision

We adopt a small baseline of patterns and delay advanced patterns until real need appears.

### Backend or Service Layer

- **Capability or domain boundary** as the primary unit of organization.
- **Handler or controller -> service or use-case -> persistence or gateway separation** when the stack supports those layers.
- **Explicit contracts and validation** for public inputs and outputs.
- **Adapter pattern** for external providers and integrations.

### Frontend or Client Layer

- **Lightweight separation of orchestration and rendering** when UI complexity justifies it.
- **Reusable state or side-effect abstractions** through the native composition model of the framework.
- **Composition over inheritance** for UI reuse.

### Shared Packages

- **Utility modules** for pure reusable logic.
- **Contract modules** for shared types, schemas, clients, or interfaces used by multiple runtime surfaces.

### Explicit Non-Goals (for now)

- No mandatory use of complex patterns (Factory hierarchies, CQRS/Event Sourcing, Mediator pipelines) unless required by measurable complexity.
- No pattern adoption only for theoretical purity.
- No framework-specific pattern should become a repo-wide rule unless documented in an app-level or core ADR on purpose.

## Consequences

- Faster onboarding and more consistent code organization.
- Lower risk of accidental architecture complexity.
- Some future features may require introducing additional patterns via new ADRs.
- Pattern choices remain aligned with KISS-first development.
