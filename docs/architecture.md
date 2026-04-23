# Architecture Overview

## Purpose

This document gives a stable, high-level view of how a repository using this Spec Kit is organized. It should describe system boundaries, major components, repository topology, and the cross-cutting rules that shape implementation.

Detailed tradeoffs and long-lived technical choices belong in ADRs under `docs/decisions/`. Optional per-application or per-surface architecture specs may live under `docs/specs/apps/<app>/architecture.md` when the repository structure needs that level of detail.

## System Context

This kit is designed to support multiple repository shapes:

- single application repositories
- monorepos with multiple deployable apps or services
- backend-only or frontend-only repositories
- libraries, platforms, and internal tools

The concrete project context is established during the bootstrap/init flow and then documented here.

## Guiding Principles

- Keep system boundaries explicit.
- Prefer repository conventions that are easy to explain and enforce.
- Record major architectural decisions in ADRs.
- Keep spec, plan, and task artifacts aligned with the actual repository structure.
- Treat domains, modules, services, apps, and packages as project-defined concepts rather than forcing one universal layout.

## Repository Layout

The consuming project should document its actual layout here after bootstrap. Typical examples include:

### Monorepo Example

```text
design/
  <artifacts-and-tokens>/
apps/
  <app-or-service>/
packages/
  <shared-package>/
libs/
  <shared-library>/
docs/
  architecture.md
  project.spec.md
  decisions/
  specs/
```

### Single-Repository Example

```text
src/
  <modules>
tests/
docs/
  architecture.md
  project.spec.md
  decisions/
  specs/
```

If the project uses a different structure, document it explicitly instead of trying to force it into one of these examples.

## Documentation Hierarchy

| Level | Location | Purpose |
|-------|----------|---------|
| Repo-wide | `docs/architecture.md` | Repository/system boundaries, layers, cross-cutting concerns |
| Project-wide | `docs/project.spec.md` | Project context, goals, bootstrap baseline |
| Optional per-app | `docs/specs/apps/<app>/architecture.md` | Internal architecture for a specific app, service, or deployable surface |
| Optional domain | `docs/specs/domains/<domain>.md` | Bounded context, module, or capability area |
| Feature | `docs/specs/features/<feature-id>/` | Feature spec, plan, and task artifacts |

## Architectural Building Blocks

Projects may use some or all of the following building blocks. Keep only the ones that actually apply:

- **Deployable surfaces**: applications, services, workers, packages, libraries, CLIs, or jobs
- **Capability boundaries**: domains, modules, bounded contexts, or feature areas
- **Shared assets**: contracts, schemas, utilities, UI primitives, design tokens, component libraries, SDKs, or platform packages
- **Operational surfaces**: CI, infrastructure, migrations, observability, release automation

## Logical Layers

Use the sections below only when they fit the project. Remove or simplify them when they do not.

### Presentation Layer

UI, API gateway, CLI, or externally facing delivery surface. Owns interaction flow and transport concerns. When the project has a frontend, this layer should also reflect the documented design system, responsive behavior, and approved design artifacts.

### Application Layer

Coordinates use cases, orchestration, validation, and integration boundaries.

### Domain or Capability Layer

Owns business rules, invariants, and core concepts where the project has meaningful domain boundaries.

### Data and Integration Layer

Owns persistence, external integrations, eventing, synchronization, and migration concerns.

## Cross-Cutting Concerns

Document the project's actual choices here after bootstrap:

- dependency management policy
- naming and language conventions
- test organization
- build output conventions
- API and contract ownership
- frontend design artifact ownership and location
- theme and token ownership
- shared component and form library strategy
- responsive design baseline
- shared package or module strategy
- observability requirements
- security and authorization boundaries

Use ADRs for the reasoning behind these choices.

## Architectural Decision Process

Any significant choice that changes system shape, introduces a long-lived dependency, or establishes an enduring implementation rule should be captured in an ADR.

The base kit provides a small generic ADR baseline. Consuming projects should update, replace, or extend those ADRs during bootstrap.

## Non-Functional Expectations

The architecture should leave room for the expectations that matter to the project, for example:

- authorization and access boundaries
- observability through logs, metrics, and traces
- performance and scalability expectations
- accessibility and usability goals
- operational safety for data changes and releases

## Evolution Rules

- Update this document when the repository structure or system boundaries change materially.
- Add per-app or per-surface architecture specs only when repo-wide documentation becomes too vague.
- Use ADRs for the why.
- Use feature specs for the what.
- Use plans and tasks for the how.