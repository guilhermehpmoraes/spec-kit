# Spec Kit

Reusable personal **Spec Driven Development** kit for bootstrapping new projects and running a consistent delivery flow across different stacks. The default bias is **Nx monorepo**, but the kit is intentionally portable enough to support combinations such as **Java + Angular**, **NestJS + React**, or other backend/frontend pairings.

## What This Kit Standardizes

- A project initialization flow that captures stack, architecture direction, and repo conventions before feature work starts.
- A 5-step SDD delivery flow for features: spec, plan, tasks, implementation, retrospective.
- Frontend planning with **Pencil** so UI decisions are validated during planning instead of being corrected repeatedly during implementation.
- Shared documentation structure under `docs/` and reusable spec templates under `docs/specs/templates/`.

## Default Baseline

These are defaults, not hard constraints:

- **Repository shape**: Nx monorepo with `apps/` and `packages/`
- **Architecture mode**: keep, adapt, or replace the baseline during project init
- **Frontend process**: mobile-first, shared component library, global theme tokens, and Pencil-driven planning
- **Delivery model**: SDD with branch lifecycle managed by the `sdd-branch` skill

## Project Init

When you copy this kit into another repository, start with `/init` or a prompt such as `quero iniciar o spec kit`.

That init flow should collect at least:

- Project summary, goals, and product context
- Selected stack by layer: backend, frontend, mobile, database, testing, CI
- Whether the repo will keep the default **Nx monorepo** baseline or adapt it
- Whether the existing architecture should be kept, adapted, or rewritten
- Applications/products that will exist in the repo
- Shared package strategy and naming conventions
- Frontend expectations, if any:
  - mobile-first behavior
  - shared UI/component library in `packages/` or equivalent shared layer
  - reusable and configurable components instead of many near-duplicate variants
  - global CSS/theme tokens for primary, secondary, tertiary colors and related design rules
  - Pencil component catalog and feature/page designs in `.pen` files

The expected output of init is to create or update the project baseline docs:

- `docs/project.spec.md`
- `docs/architecture.md`
- `docs/specs/templates/project-init.spec.md` as the reusable init questionnaire/template
- `docs/specs/apps/<app>/architecture.md` when app-level architecture is needed
- Pencil files and component catalogs when the project has frontend/UI scope

## Workflow (SDD)

The feature delivery flow starts after project init and is defined in `.github/copilot-instructions.md`.

| Step | Goal | Main Output |
| ---- | ---- | ----------- |
| Init | Establish project baseline and stack profile | `docs/project.spec.md`, `docs/architecture.md`, optional app architecture docs |
| 1 — Feature Spec | Define feature scope and acceptance | `feature.spec.md` |
| 2 — Feature Plan | Produce technical plan and validate design | `plan.spec.md`, approved Pencil artifacts when UI exists |
| 3 — Task Breakdown | Derive implementation-ready task files | `TXXX-*.task.spec.md` files |
| 4 — Implementation | Deliver code against an approved task | code, tests, updated task status |
| 5 — Retrospective | Capture learnings and refine the kit | updated specs/templates/instructions/ADRs |

### Planning Rule for Frontend Features

If a feature affects UI, routing, forms, layout, or shared components:

- the design must be created or updated in **Pencil during Step 2**
- the plan must reference the `.pen` file and the relevant screens/components
- the implementation task must use the approved `.pen` artifact as the source of truth
- design changes should happen in planning first, not ad hoc during implementation

## Frontend Baseline

The default frontend baseline for projects using this kit is:

- **Mobile first** layout and interaction design
- **Own component library** maintained as a shared package for reuse across apps
- **Reusable and dynamic components** with configurable behavior instead of many small variants
- **Pencil component catalog** so shared components can be reviewed visually in one place
- **Global CSS/theme layer** defining tokens such as primary, secondary, tertiary colors and shared visual rules

## Artifact Structure

```text
docs/
  architecture.md
  project.spec.md
  decisions/
  specs/
    apps/
    domains/
    features/
    templates/
```

## Rules

- No production code in Steps 1-3.
- Plans and tasks must contain enough technical detail to implement without hidden discovery.
- If UI is involved, Pencil validation happens in Step 2 and implementation follows the approved `.pen` artifact.
- Foundational docs should be updated when the stack or architecture baseline changes.
