# Project Initialization Instructions

This file exists only while this spec kit is being adapted to a new project.

After initialization is complete, delete this file.

## Purpose

Use this bootstrap step to turn the kit into a project-specific workspace without losing the SDD process, ADR discipline, or validation gates.

## When To Run Initialization

Run initialization before the first real feature spec when any of these are still undefined or inherited from another project:

- project or product identity
- monorepo vs single-repo shape
- apps, services, packages, or modules in scope
- languages, frameworks, runtimes, and data stores
- code quality tools and test tools
- naming conventions
- branching and integration branch defaults
- which ADRs are core vs optional vs app-specific

## Required Inputs

Collect enough detail to update the foundational docs in one pass.

### Project Identity

- Project or workspace name
- Short description of the product or platform
- Intended users or operators
- Single product vs multi-app platform

### Repository Shape

- Single repo or monorepo
- If monorepo: workspace tool (`Nx`, `Turborepo`, plain workspaces, Gradle, Maven, other)
- Package or build tool (`pnpm`, `npm`, `yarn`, `bun`, `gradle`, `maven`, other)

### Delivery Surfaces

- Backend services or APIs
- Frontend apps
- Mobile apps
- Shared packages or libraries
- Infra or data projects

### Technology Choices

- Languages used per surface
- Frameworks used per surface
- Databases, queues, caches, and external integrations
- Test tooling per surface
- Code quality tools per surface

### Conventions

- Source code language
- Database naming language, if applicable
- API style or contract conventions
- Project naming rules
- Branching integration branch name

## Files To Update During Initialization

Initialization should update, create, or clean up the following as needed:

- `README.md`
- `docs/project.spec.md`
- `docs/architecture.md`
- `.github/copilot-instructions.md`
- `docs/decisions/*.md`
- `docs/specs/apps/<app>/architecture.md`
- `docs/specs/apps/_template/architecture.md`

Also review whether old project examples should be archived or removed:

- `docs/specs/features/*`
- `docs/specs/domains/*`
- `docs/specs/apps/*`

## Initialization Rules

1. Preserve the SDD workflow, status gates, and retrospective loop.
2. Keep stack-specific guidance out of core docs unless it is a deliberate default.
3. Treat Biome as a preferred default, not a universal mandate.
4. If the project mixes languages, document validation per surface instead of forcing one tool everywhere.
5. Move highly stack-specific decisions to app-level decisions instead of core ADRs.
6. Do not leave template placeholders in core project files once initialization is complete.

## Expected Outputs

By the end of initialization:

- the root docs describe the actual project instead of the kit template
- optional ADRs are either generalized, scoped, or deprecated
- app or service architecture docs match the chosen stack
- quality tooling expectations are explicit
- stale project-specific examples are removed or clearly marked
- this file is deleted