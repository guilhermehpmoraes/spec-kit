# SDD Spec-Kit

A reusable **Spec Driven Development (SDD)** kit for new software projects. Start any project with a clear, process-driven approach to features: spec → tasks → plan → implementation.

## What's Inside

- **4-step delivery flow** (mandatory): Feature Spec → Task Spec → Plan Spec → Implementation
- **Feature templates** (`docs/specs/templates/`): Ready-to-fill specs for features, tasks, and plans
- **Baseline ADRs** (`docs/decisions/`): Engineering principles, design patterns, and quality tooling decisions
- **Architecture guide** (`docs/architecture.md`): Stack-agnostic system boundaries and layers
- **Bootstrap prompt** (`.github/prompts/sdd-bootstrap-once.prompt.md`): One-shot project initialization with self-destruct

## Quick Start for a New Project

1. Import this kit into your new project.
2. Run the SDD bootstrap prompt (`.github/prompts/sdd-bootstrap-once.prompt.md`) to:
    - Define your product/domain context
    - Lock in your stack profile (backend, frontend, database, testing, CI/CD)
    - Document naming conventions
3. The prompt deletes itself after setup to keep context clean.
4. Follow the 4-step flow for every feature starting with `docs/specs/templates/feature.spec.md`.

## Stack Profile

This kit is **stack-agnostic**. Define your project's profile in `docs/specs/project.spec.md`:

- **Backend**: NestJS | Spring Boot | FastAPI | ASP.NET | etc.
- **Frontend**: React | Angular | Vue | Svelte | etc.
- **Database**: PostgreSQL | MySQL | MongoDB | Firestore | etc.
- **Testing**: Jest | Vitest | Pytest | Mocha | etc.
- **Repo**: Nx monorepo | single repo | Gradle | Maven | etc.
- **Package manager**: pnpm | npm | yarn | bun | etc.
- **CI/CD**: GitHub Actions | GitLab CI | Jenkins | Cloud Build | etc.

## Core Files

| File                                           | Purpose                                                       |
| ---------------------------------------------- | ------------------------------------------------------------- |
| `.github/copilot-instructions.md`              | Main SDD rules + 4-step flow (stack-agnostic)                 |
| `AGENTS.md`                                    | Agent guidance (replace with your stack tools if no Nx)       |
| `docs/specs/project.spec.md`                   | Project-specific template (duplicate & fill for each project) |
| `docs/specs/templates/feature.spec.md`         | Feature spec template                                         |
| `docs/specs/templates/task.spec.md`            | Task breakdown template                                       |
| `docs/specs/templates/plan.spec.md`            | Implementation plan template                                  |
| `docs/decisions/`                              | Baseline ADRs (engineering, patterns, quality tools)          |
| `.github/prompts/sdd-bootstrap-once.prompt.md` | One-shot project setup prompt                                 |
| `.vscode/extensions.json`                      | Recommended VS Code extensions                                |

## Key Principles

- **KISS first**: Simplest solution that meets requirements
- **Self-descriptive code**: Clear naming and structure over comments
- **Specs drive acceptance**: Tests and criteria come from the spec, not invented during implementation
- **Continuous evolution**: After each feature, review what worked and update your standards

## For Your Project's First Feature

1. Copy `docs/specs/templates/feature.spec.md` → `docs/specs/features/your-feature.spec.md`
2. Fill in the feature spec directly in chat for review (do NOT create a file automatically)
3. Once approved, break it into tasks using the task template
4. Select a task and generate a plan spec for implementation
5. Implement only after plan review

## Updating the Kit

After each feature delivery, run a short retrospective and update:

- Templates (add rules that reduced friction)
- ADRs (document new decisions)
- `.github/copilot-instructions.md` (sharpen guidance)
- `docs/architecture.md` (evolve system understanding)

## Multi-Stack Support

This kit works across stacks because:

- **Instructions** are process-focused, not framework-specific
- **Templates** use placeholders for stack-dependent details
- **ADRs** capture principles, not hard requirements
- **Bootstrap prompt** collects your stack upfront

When starting a new project in a different stack (Java + Spring Boot + Angular, for example), just update `docs/specs/project.spec.md` with the active profile and the kit adapts.

---

**Adopt this kit → Define your stack → Follow the 4-step flow → Ship with confidence.**
