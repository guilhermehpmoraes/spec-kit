---
description: "Initialize the Spec Kit in a new repository by capturing stack, architecture, and frontend planning defaults"
name: "init"
argument-hint: "Describe the project, current repo context, or what should be kept/adapted from the default kit"
agent: "agent"
---
You are responsible for executing the **Project Init (Step 0)** flow of this repository's Spec Kit.

This prompt is used when a repository is adopting the kit for the first time or when the baseline project context must be reset.

## Input

{{input}}

## Goal

Capture the project's baseline decisions and materialize them into the repository documentation so future SDD steps can run without guessing the stack, architecture mode, frontend conventions, or design workflow.

## Required reading

Before asking questions or writing files, read and follow:

1. `docs/project.spec.md`
2. `docs/architecture.md`
3. `.github/copilot-instructions.md`
4. `docs/specs/templates/project-init.spec.md`
5. Relevant ADRs in `docs/decisions/`

## Required questions

Ask the user a focused batch of questions that covers at least:

1. Project summary, goals, and primary applications/products
2. Stack by layer:
   - backend
   - frontend
   - mobile
   - data
   - testing
   - CI/CD
3. Repository shape:
   - keep Nx monorepo baseline
   - adapt Nx monorepo baseline
   - use another repo structure
4. Architecture mode:
   - keep baseline
   - adapt baseline
   - replace baseline
5. Shared package/shared layer expectations
6. Naming conventions that should become project defaults
7. Frontend baseline when UI exists:
   - mobile-first rules
   - shared component library
   - reusable/configurable component expectations
   - global CSS/theme token expectations
   - Pencil component catalog location
   - Pencil planning rule for UI features

If the initial user input already answers some of these, do not re-ask them. Only ask what is still needed to make the init deterministic.

## Materialization rules

After the critical questions are answered:

1. Use `docs/specs/templates/project-init.spec.md` as the structure for the init result.
2. Update `docs/project.spec.md` with the finalized project baseline.
3. Update `docs/architecture.md` with the finalized architecture mode and repository shape.
4. If the project has one or more explicit applications, create or update `docs/specs/apps/<app>/architecture.md` when app-level details are already known.
5. If the project has frontend scope, make sure the resulting docs explicitly state:
   - Pencil is used during Step 2 for UI planning
   - approved `.pen` artifacts become the implementation source of truth
   - the shared component library and global theme layer are part of the baseline

Do not write production code in this step.

## Exit criteria

- The repository baseline docs are updated to reflect the chosen stack and architecture.
- The frontend design workflow is explicit when UI work exists.
- The user receives a concise summary of:
  - chosen stack profile
  - chosen architecture mode
  - whether Nx baseline was kept or adapted
  - frontend/Pencil defaults
- Ask the user whether they want any further baseline docs or app architecture docs materialized now.