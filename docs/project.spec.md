# Project Spec Baseline

## Purpose

This file is the project-wide baseline created or refined during **Spec Kit init**. It records the stack profile, architectural direction, and delivery rules for the repository that receives this kit.

The goal is to make the kit portable across different technologies without losing a consistent workflow.

## Initialization Inputs

Every new project initialized from this kit should answer these questions before feature work begins:

- What problem is the project solving?
- Which products, applications, or services will live in this repository?
- Which stack is selected for each layer?
	- backend
	- frontend
	- mobile
	- database
	- testing
	- CI/CD
- Will the project keep the default **Nx monorepo** baseline, adapt it, or use another repo shape?
- Will the architecture keep the kit baseline, adapt it, or be rewritten?
- Which naming conventions, shared packages, and deployment boundaries should be treated as defaults?
- Does the project include frontend/UI work that must use Pencil during planning?

## Default Repository Baseline

Unless init explicitly says otherwise, this kit assumes:

- A monorepo-oriented structure
- Nx as the default workspace orchestrator when the project fits that model
- Shared packages/libraries for cross-app contracts, UI primitives, utilities, and design tokens
- DDD-style domain separation when the problem benefits from bounded contexts
- Feature delivery through SDD with explicit specs, plans, tasks, and retrospectives

This baseline is a default, not a hard dependency. The kit should still work for projects such as:

- Java backend + Angular frontend
- NestJS backend + React frontend
- Backend-only services
- Repositories that keep app code and shared code in different folder shapes

## Architecture Strategy

During init, the project must choose one of these modes and document it in `docs/architecture.md`:

- **Keep baseline**: use the kit architecture with minimal changes
- **Adapt baseline**: keep the overall shape but change stack-specific or repo-specific parts
- **Replace baseline**: keep the SDD workflow but define a different architecture structure

## Frontend Baseline

If the project has frontend scope, these defaults apply unless init overrides them:

- **Mobile first** is the default design direction.
- A **shared component library** should exist in a shared layer such as `packages/`.
- Components should be **reusable and configurable**, favoring parameters and composition over many near-duplicate variants.
- A **Pencil component catalog** should exist so shared components can be reviewed visually.
- Features that change UI must be designed and validated in **Pencil during Step 2 (Feature Plan)**.
- Implementation should follow the approved `.pen` artifact instead of redesigning during task execution.
- A **global CSS/theme layer** should define tokens such as primary, secondary, tertiary colors and other reusable visual rules.

## Delivery Model

This kit uses a two-level workflow:

1. **Project Init** — establish stack profile, architecture mode, and project-wide conventions.
2. **Feature Delivery** — run the standard SDD flow for each feature:
	 - Step 1: Feature Spec Draft
	 - Step 2: Feature Plan
	 - Step 3: Task Breakdown
	 - Step 4: Task Implementation
	 - Step 5: Feature Finish / Retrospective

## Success Conditions for Init

Project init is complete only when:

- `docs/project.spec.md` reflects the actual project context
- `docs/architecture.md` reflects the chosen architecture mode
- stack choices are explicit enough to plan future features without guessing
- frontend standards are explicit when UI work exists
- shared package and component library expectations are documented

## References

- Stack and workflow rules: `/.github/copilot-instructions.md`
- Architecture baseline: `docs/architecture.md`
- Feature specs: `docs/specs/features/`
- Templates: `docs/specs/templates/`
- ADRs: `docs/decisions/`