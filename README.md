# Spec Kit

Reusable **Spec Driven Development (SDD)** kit for bootstrapping new or existing software projects.

This repository is meant to be reused as a foundation, not as a fixed application. It provides:

- foundational docs (`docs/project.spec.md`, `docs/architecture.md`, ADRs)
- reusable spec templates (`feature`, `plan`, `task`)
- step-oriented prompts for the SDD workflow
- skills and instructions for branch, commit, planning, and implementation flows

The kit is intentionally flexible. It should work for monorepos or single repositories, and for different stacks such as Java + Angular, Next.js, NestJS + React, or others.

## Core Idea

Use the kit in two phases:

1. **Bootstrap the project** with `/init` so the repository captures its own context, stack, topology, conventions, and validation rules.
2. **Run normal SDD delivery** with `/feature`, `/plan`, `/tasks`, `/implement`, and `/finish`.

The bootstrap step is one-time setup. After it materializes the project-specific instructions and docs, the bootstrap-only instruction can be removed from the consuming repository.

## Bootstrap Flow

The first action in a new repository should be `/init`.

That flow should ask for the project's actual context, including:

- product or platform name
- repository topology
- languages, frameworks, and key libraries
- package manager and task runner
- testing, linting, and quality gates
- naming conventions
- branching and release rules
- optional domain/application architecture structure

The result of `/init` should update at least:

- `README.md`
- `docs/project.spec.md`
- `docs/architecture.md`
- `.github/copilot-instructions.md`
- relevant ADRs in `docs/decisions/`

## SDD Workflow

After bootstrap, the standard five-step SDD flow applies.

| Step | Prompt | Input | Output |
|------|--------|-------|--------|
| 0 — Bootstrap | `/init` | Project context and setup answers | Project-specific foundational docs and instructions |
| 1 — Feature Spec | `/feature` | Feature description or existing draft feature | `feature.spec.md` in `Draft` |
| 2 — Feature Plan | `/plan <feature-id>` | Approved feature spec | `plan.spec.md` with implementation detail |
| 3 — Task Breakdown | `/tasks <feature-id>` | Approved plan | One task spec per implementation slice |
| 4 — Implementation | `/implement <feature-id> <task-id>` | Ready task spec | Code, tests, validation, status updates |
| 5 — Retrospective | `/finish <feature-id>` | Completed feature | Retrospective and process improvements |

## Expected Repository Structure

The kit supports different repository shapes. At minimum, it expects:

```text
docs/
  architecture.md
  project.spec.md
  decisions/
  specs/
    features/
    templates/
.github/
  copilot-instructions.md
  prompts/
  skills/
```

Everything else depends on the consuming project.

## Artifact Structure

```text
docs/specs/features/<feature-id>/
  feature.spec.md
  plan.spec.md
  tasks/
    T001-<short-title>.task.spec.md
    T002-<short-title>.task.spec.md
```

## Rules

- No production code in Steps 0–3 unless the workflow explicitly says otherwise.
- Specs, plans, and tasks should be self-sufficient enough to avoid hidden context.
- Tasks must not be marked `Done` without real validation evidence.
- Foundational docs and ADRs should reflect the actual project after bootstrap, not sample data from the kit.

## Adapting the Kit

When reusing this kit in another project:

1. Copy the repository contents you want to adopt.
2. Run `/init` and answer the bootstrap questions accurately.
3. Review the generated foundational docs and ADRs.
4. Remove or archive the bootstrap-only instruction if the init flow created one.
5. Start normal feature delivery.