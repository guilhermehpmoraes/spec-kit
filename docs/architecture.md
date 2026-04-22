# Architecture Overview

## Purpose

This document defines the stable, repo-wide architecture baseline for the active project. It should stay high level and durable: workspace shape, documentation layers, cross-cutting rules, and where stack-specific detail belongs.

Detailed tradeoffs belong in ADRs under `docs/decisions/`. App or service specifics belong in `docs/specs/apps/<app>/architecture.md`.

## System Context

This workspace may represent:

- a single application or service
- a monorepo with multiple apps, services, or shared packages
- a mixed-language platform where different surfaces use different toolchains

The active repository shape must be documented during initialization and kept current as the workspace evolves.

## Guiding Principles

- Keep durable decisions explicit and easy to trace.
- Push stack-specific details down to the smallest appropriate scope.
- Prefer explicit boundaries between apps, services, domains, and shared packages.
- Keep validation expectations documented per surface.
- Evolve the architecture through specs, ADRs, and retrospective learnings instead of ad hoc conventions.

## Repository Layout

The exact source layout is project-specific, but the documentation layout is stable:

```text
docs/
  architecture.md
  project.spec.md
  decisions/
  specs/
    apps/<app>/
    domains/
    features/<feature-id>/
    templates/
source/
  apps/ or services/ or packages/ or modules/
```

Common code layouts include:

- `apps/` plus `packages/` for monorepos
- `services/` plus `libs/` for service platforms
- a single top-level application directory for smaller repos

## Documentation Hierarchy

| Level | Location | Purpose |
|-------|----------|---------|
| Repo-wide | `docs/architecture.md` | Stable structure, boundaries, cross-cutting concerns |
| Repo-wide | `docs/project.spec.md` | Why this workspace exists and how delivery is governed |
| Core decisions | `docs/decisions/` | Reusable or project-wide architectural decisions |
| App or service | `docs/specs/apps/<app>/architecture.md` | Local architecture, stack, conventions, and boundaries |
| Domain | `docs/specs/domains/<domain>.md` | Optional bounded-context documentation |
| Feature | `docs/specs/features/<feature-id>/` | Feature spec, plan, and implementation tasks |

## Workspace Conventions

### Repository Shape

- If the project is a monorepo, document the orchestration tool and dependency strategy explicitly.
- If the project is not a monorepo, keep the same docs structure and scope decisions to the application or service level.

### Dependency Management

- Prefer one documented dependency strategy per workspace.
- If a monorepo uses a single-version policy, document it in ADR-006.
- If different parts of the workspace use different package or build tools, document ownership and boundaries clearly.

### Naming Conventions

- Naming rules must be documented explicitly for source code, database objects, packages, and public APIs when relevant.
- Do not assume the same naming language or casing rules across all future projects.
- If naming rules are tied to a specific tool or workspace manager, scope them accordingly.

### Test Organization

- Tests may be colocated with source or split into dedicated test projects.
- The chosen strategy must be documented in the relevant app or service architecture doc.
- Validation commands must be explicit enough that implementation tasks can prove completion.

### Build Output

- Centralized build output is recommended when it simplifies CI, packaging, or artifact collection.
- If the workspace uses per-project output folders instead, document that choice in the app or service architecture docs.

## Applications, Services, and Domains

### Applications and Services

Applications or services are deployable runtime surfaces. Each one should own its local stack details, delivery constraints, and architecture notes.

### Domains

Use domains when bounded contexts improve clarity. If the project uses DDD:

- keep domain boundaries explicit
- scope domains to one application or service unless a deliberate shared model exists
- document shared concepts through contracts or shared packages, not by blurring domain ownership

## Logical Layers

The exact technology varies by project, but most work should fit into a small set of clear layers.

### Presentation Layer

Handles user interaction, rendering, page or screen composition, and client-side orchestration when applicable.

### Application Layer

Coordinates use cases, validation, permissions, orchestration, and integration with infrastructure.

### Domain Layer

Owns the business rules, invariants, and core concepts of the problem space.

### Data and Integration Layer

Owns persistence, external APIs, queues, caches, and other infrastructure integration points.

## Cross-Cutting Concerns

### Shared Contracts

Use shared packages or modules for types, schemas, clients, utilities, or UI primitives only when shared ownership is real.

### Code Quality Tooling

- Biome is the preferred default when it fits the active stack.
- If part of the stack is unsupported by Biome, define the replacement tool and scope in ADR-003.
- Do not leave quality tooling implicit for any language surface.

### Testing

- Unit, integration, and end-to-end testing expectations should be documented per surface.
- A task is not done until the relevant validation commands are executed and recorded as evidence.

### API and Interface Contracts

When multiple surfaces interact, keep request, response, event, or schema contracts explicit and version-aware.

## Architectural Decision Process

Capture decisions in ADRs when they:

- change the shape of the system
- establish a long-lived pattern
- introduce a significant dependency or workflow
- affect more than one app or service

Stack-specific decisions that are not meant to become workspace-wide defaults should live under `docs/specs/apps/<app>/decisions/`.

## Evolution Rules

- Update this document when the workspace structure changes materially.
- Update app or service architecture docs when local implementation structure changes.
- Use ADRs for the why.
- Use feature specs for the what.
- Use plans and tasks for the how.
