# Architecture Overview

## Purpose

This document describes the default architectural baseline for repositories initialized with this Spec Kit. It is intentionally **stack-agnostic** and should be refined during project init to match the selected technologies and repository shape.

Detailed tradeoffs and long-lived decisions belong in ADRs under `docs/decisions/`.

## Architecture Modes

Each initialized project should choose one of these modes:

| Mode | Meaning |
| ---- | ------- |
| Keep baseline | Use the default Spec Kit structure with minimal changes |
| Adapt baseline | Keep the overall flow and layout, but customize stack- or repo-specific areas |
| Replace baseline | Keep SDD and templates, but define a different project structure |

## Repository Baseline

The default repository bias is an **Nx monorepo**, especially when the project contains multiple apps or shared packages. That said, the kit is not limited to JavaScript or TypeScript stacks.

Default layout:

```text
apps/
  <application>/
    backend/
    backend-tests/
    frontend/
    frontend-tests/
    mobile/
packages/
  <shared-lib>/
docs/
  architecture.md
  project.spec.md
  decisions/
  specs/
    apps/
    domains/
    features/
    templates/
design/
  pencil/
    components.pen
    features/
```

Projects can adapt this layout when needed. For example, a Java backend may live under `apps/<application>/backend/` while an Angular frontend lives under `apps/<application>/frontend/`, or the repo may keep a different folder organization while still following the same SDD flow.

## System Boundaries

### Applications / Products

Applications are deployable products or major services. A project may contain one or many applications.

### Domains / Modules

When the problem benefits from DDD, domains should be scoped as bounded contexts and documented in `docs/specs/domains/`. If DDD is not needed, the same documentation structure can be used for modules, services, or functional areas.

### Shared Packages

Cross-application assets should live in shared packages or shared modules. Typical examples:

- UI/component libraries
- design tokens and theme contracts
- API contracts and DTOs
- utility libraries
- infrastructure helpers

## Frontend Architecture Baseline

If the project has frontend scope, this kit assumes the following defaults unless init overrides them:

- **Mobile-first** layout and interaction design
- A **shared component library** for reuse across all apps
- **Reusable and configurable components** instead of many nearly identical variants
- A **global CSS/theme layer** with shared tokens such as primary, secondary, tertiary colors
- A **Pencil component catalog** so shared components can be reviewed visually
- **Pencil-first UI planning**: feature plans that affect UI must create or update `.pen` artifacts before implementation starts

The planning output for UI features should identify:

- the `.pen` file to use
- the screens/routes/components covered by the design
- shared components to reuse or extend
- responsive behavior, with mobile as the starting breakpoint
- theme/token changes required for implementation

## Backend and Integration Baseline

Backend architecture is project-specific and may use different languages or frameworks. The baseline expectation is only that it remains explicit enough to support planning and task generation.

Document in project-specific architecture specs:

- main runtime/framework choices
- service/module boundaries
- persistence strategy
- integration contracts
- test strategy by layer

## Documentation Hierarchy

| Level | Location | Purpose |
| ----- | -------- | ------- |
| Repo-wide | `docs/project.spec.md` | stack profile, project-level constraints, and init baseline |
| Repo-wide | `docs/architecture.md` | architectural shape, boundaries, and cross-cutting rules |
| Per-app | `docs/specs/apps/<app>/architecture.md` | app-specific architecture and conventions |
| Domain/module | `docs/specs/domains/<domain>.md` | bounded context or module documentation |
| Feature | `docs/specs/features/<feature-id>/` | feature spec, plan, and tasks |

## Decision Process

Use ADRs for decisions that change the long-term shape of the system, including:

- architecture structure
- repository conventions
- branching workflow
- API documentation standards
- stack-specific patterns that should become defaults

Current baseline references in this kit:

- `docs/decisions/000-use-adrs.md`
- `docs/decisions/001-engineering-principles.md`
- `docs/decisions/002-design-patterns-baseline.md`
- `docs/decisions/004-domain-driven-design-baseline.md`
- `docs/decisions/008-project-naming-conventions.md`
- `docs/decisions/009-sdd-branching-strategy.md`
- `docs/decisions/010-scalar-api-documentation.md`

## Evolution Rules

- Update this document when init changes the baseline architecture.
- Update per-app architecture docs when an application needs more specific guidance.
- Keep feature specs focused on feature intent and acceptance.
- Keep plans and tasks detailed enough that implementation does not depend on hidden design decisions.
