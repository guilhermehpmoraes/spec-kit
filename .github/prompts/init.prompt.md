---
description: "Bootstrap a repository to use this Spec Kit by collecting project context and materializing foundational docs"
name: "init"
argument-hint: "Optional note about the project or current repository state"
agent: "agent"
---
You are responsible for executing **Step 0 — Project Bootstrap / Init** for a repository adopting this Spec Kit.

## Goal

Turn the reusable kit into a project-specific baseline by collecting the real repository context and materializing the foundational docs and instructions.

This step happens **once per repository adoption** before the normal SDD feature flow starts.

## Required reading

Before asking questions or editing files, read and internalize:

1. `README.md`
2. `docs/project.spec.md`
3. `docs/architecture.md`
4. `.github/copilot-instructions.md`
5. `docs/decisions/`
6. `AGENTS.md`
7. Any existing repository docs that clarify the actual stack, topology, or conventions

If `.github/bootstrap-instructions.md` exists, read it too.

## Discovery process

### Phase 1 — Inspect the repository

Inspect the current repository to determine:

- whether it is a monorepo, single repo, or multi-service repo
- which languages, frameworks, or build tools are already present
- whether code already exists or the repository is mostly documentation/scaffolding
- which foundational docs already reflect real project truth versus sample kit placeholders

### Phase 2 — Ask one organized batch of questions

Ask the user for the real project context in one grouped batch. Cover at least:

- product/platform name and purpose
- repository topology
- main deployable surfaces (apps, services, packages, libraries, workers, etc.)
- languages, runtimes, frameworks, and key libraries
- database and persistence modeling conventions when the project stores business data, including:
	- whether there is a reusable base entity or equivalent shared persistence abstraction
	- the standard six lifecycle fields for created by/at, modified by/at, and deleted by/at
	- how those fields are named according to the project's naming language
	- whether soft delete is the default deletion strategy and any known exceptions
- dependency/package/build tooling
- test, lint, format, and static analysis tools
- frontend design workflow when the project has UI work, including:
	- where design artifacts and tokens live
	- where reusable UI implementation code lives
	- how global theme values are defined
	- whether mobile-first is the default responsive baseline
	- whether Pencil is required for visually relevant UI changes
- if the project uses Nx, whether applications are grouped as `apps/<app>/backend` and `apps/<app>/frontend`
- if the project uses Nx, where unit, integration, and e2e tests live for backend and frontend stacks
- naming and language conventions
- branching and release conventions
- whether the project uses domains, modules, bounded contexts, or another boundary model
- whether optional per-app architecture docs should exist

If answers are incomplete, ask only the minimum follow-up questions needed to materialize the baseline correctly.

## Materialization requirements

After the answers are complete, update at least:

1. `README.md`
2. `docs/project.spec.md`
3. `docs/architecture.md`
4. `.github/copilot-instructions.md`
5. Relevant ADRs in `docs/decisions/`

Also create or update additional docs when needed, for example:

- `docs/specs/apps/<surface>/architecture.md`
- `docs/specs/domains/<domain>.md`
- `design/` references or supporting docs when the project has a frontend design system baseline
- setup or operations docs referenced by the README

## Bootstrap self-removal

If a bootstrap-only instruction file exists solely for setup, remove it after the bootstrap is fully materialized.

The default bootstrap-only file for this kit is:

- `.github/bootstrap-instructions.md`

Delete it only after all required files have been successfully updated.

## Exit criteria

- Foundational docs reflect the actual project rather than kit placeholders.
- Project conventions are explicit.
- The repository is ready to begin normal SDD work with `/feature`.
- Any bootstrap-only instruction file has been removed or clearly retired.

## Final response

Report:

- what project context was captured
- which files were updated
- any assumptions that still need confirmation
- whether bootstrap-only instructions were removed