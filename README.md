# Personal Spec Kit

Reusable Spec Driven Development kit for personal projects. The goal is to keep the workflow, templates, and decision discipline stable while allowing each new project to define its own stack, language mix, architecture, and delivery defaults.

This repository is the kit itself, not a single product codebase. Some existing docs under `docs/specs/` reflect a real reference project and should be treated as examples, not as hard requirements for future projects.

## What Stays Stable

- Spec-first delivery flow: feature -> plan -> tasks -> implementation -> retrospective
- ADR-based decision tracking
- Checklist-gated status transitions
- Feature/task branching aligned with specs
- Strong planning artifacts before implementation
- Test evidence and validation gates before marking work done

## What Is Configurable Per Project

- Monorepo or single-repo structure
- Nx, plain workspaces, or another build/orchestration tool
- Backend and frontend stacks
- Database and infrastructure choices
- Naming conventions by layer
- Code quality tooling
- CI/CD model

## Preferred Defaults

These are defaults, not hard requirements:

- Monorepo is the default mental model
- Nx is the preferred monorepo orchestration when it fits
- Biome is the preferred formatter/linter when the target language supports it well
- Mixed-tooling setups are allowed when the stack demands it

Example: in a Java + frontend project, backend quality tooling may use the Java ecosystem while frontend keeps Biome.

## Init Workflow

When using this kit in a new project, start with a one-time initialization pass.

1. Define the project context: product, repo shape, apps or services, languages, frameworks, databases, test stack, CI, naming conventions, and code quality tooling.
2. Ask Copilot to run the initialization using `docs/init.md`.
3. Update the core Markdown files with the new project defaults and constraints.
4. Create or update app-specific architecture docs under `docs/specs/apps/` as needed.
5. Keep only the ADRs that are truly baseline decisions; move stack-specific choices to app-level docs or project-specific ADRs.
6. After initialization is complete, delete `docs/init.md` so the repository returns to normal operating mode.

## Core Structure

```text
docs/
  init.md                       temporary bootstrap instructions for a new project
  architecture.md               repo-wide architecture baseline
  project.spec.md               high-level project or workspace spec
  decisions/                    reusable ADRs and project-wide decisions
  specs/
    README.md                   guidance for active specs vs reference material
    apps/<app>/                 app or service specific architecture docs
    domains/                    optional bounded-context specs
    features/<feature-id>/      feature spec, plan, and task files
    templates/                  canonical spec templates
.github/
  copilot-instructions.md       operating rules for SDD execution in this repo
```

## SDD Flow

The delivery flow remains the same across projects:

| Step | Purpose | Main Output |
|------|---------|-------------|
| 1 — Feature Spec | Define the problem and scope | `feature.spec.md` |
| 2 — Feature Plan | Define the implementation approach | `plan.spec.md` |
| 3 — Task Breakdown | Split into implementation-ready units | `tasks/TXXX-*.task.spec.md` |
| 4 — Task Implementation | Execute one approved task | Code + tests + updated task spec |
| 5 — Feature Finish | Capture learnings and improve the kit | Updated specs, templates, or ADRs |

## Rules

- No production code in Steps 1 through 3
- Plans and tasks must be implementation-ready, not vague outlines
- A task is not done until tests have been executed and recorded as evidence
- Code quality must be enforced with the tooling chosen for the target stack
- Core docs should stay generic; stack-specific instructions belong in the project or app that owns them

## Suggested Use In A New Project

1. Copy this kit into the new repository.
2. Run the init workflow described in `docs/init.md`.
3. Replace generic placeholders with the project's real stack and constraints.
4. Archive or remove reference material that does not apply.
5. Start Step 1 of SDD for the first real feature.

## Reference Material

The current repository also contains reference examples from an existing Nx-based product setup. They are useful as patterns, especially for:

- Nx monorepo organization
- Spec packages under `docs/specs/features/`
- App-specific architecture docs
- ADR evolution over time

Use them as examples when helpful, but do not treat their stack choices as mandatory for future projects.
