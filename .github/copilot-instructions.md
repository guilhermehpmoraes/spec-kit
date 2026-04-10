# Copilot Instructions — Satie

## Spec Driven Development

This project follows **Spec Driven Development (SDD)**. Every feature, module, or architectural change starts from a spec before any code is written.

### Workflow

1. **Spec first** — Before writing code, check `docs/specs/` for the relevant spec. If none exists, create or update one.
2. **Decide** — Architectural decisions are recorded as ADRs in `docs/decisions/`. Reference the relevant ADR number in specs and code when applicable.
3. **Implement** — Write code that fulfills the spec. Reference the spec path in PR descriptions and commit messages when relevant.
4. **Validate** — Tests and acceptance criteria come from the spec, not invented during implementation.

### Mandatory 4-Step Delivery Flow

Follow this sequence strictly, one step per chat:

1. **Step 1 — Feature Spec only**: When asked for a feature, generate only a feature spec based on `docs/specs/templates/feature.spec.md` directly in chat for user review (do not create a spec file automatically). Do not edit or create production code.
2. **Step 2 — Task Spec only (same chat as Step 1)**: In the same chat, after feature validation, break the feature into tasks using `docs/specs/templates/task.spec.md` so the user can validate, move tasks to Jira, and organize sprint distribution. Do not edit or create production code.
3. **Step 3 — Plan Spec for one task only (new chat, plan mode)**: In a separate chat, when the user selects a specific task, generate only the task development plan using `docs/specs/templates/plan.spec.md`. Do not edit or create production code.
4. **Step 4 — Implementation**: Only after the selected task plan is reviewed and approved should implementation begin for that task. Code edits and file creation are allowed only in this step.

### Continuous SDD Sharpening Loop

After each completed feature, run a short retrospective and evolve the process:

1. **Review outcomes** — Capture what worked well and what caused friction.
2. **Promote patterns** — Convert proven practices into stable rules/templates.
3. **Fix weak spots** — Add one or two concrete process improvements for the next cycle.
4. **Apply next feature** — Use the updated rules in the following spec and implementation.

Keep this lightweight: prefer small, evidence-based changes over large process rewrites.

### Reading specs

- Start with `docs/specs/project.spec.md` for high-level project understanding.
- Templates live in `docs/specs/templates/`.
- Use `docs/specs/templates/feature.spec.md` as the default starting point for new feature specs.
- Use `docs/specs/templates/plan.spec.md` to plan implementation before writing tasks.
- Use `docs/specs/templates/task.spec.md` to derive executable tasks from the approved plan.
- Domain specs live in `docs/specs/domains/`.

### Reading decisions

- ADRs are numbered sequentially: `docs/decisions/NNN-title.md`.
- Always check existing ADRs before proposing conflicting approaches.

## Project Context

- **Satie** is a centralized data platform for schools — organized school structure visualization with dashboards and reports.
- Nx monorepo: apps in `apps/`, shared packages in `packages/`.
- Full-stack domain apps follow `apps/<domain>/backend/` + `apps/<domain>/frontend/`.

## Stack

- **Backend**: NestJS
- **Frontend**: Vite + React + TanStack Router + TanStack Query + Tailwind CSS
- **Database**: PostgreSQL
- **Testing**: Jest + Supertest (backend), Vitest + React Testing Library (frontend), Playwright (e2e)
- **Package manager**: pnpm
- **CI**: AWS-based

## Naming Conventions

- **Database** (tables, columns, indexes, constraints): Portuguese
- **Source code** (all layers): English
- **Docs, specs, ADRs**: English

## Code Guidelines

- Follow existing patterns in the codebase before introducing new ones.
- Check `docs/decisions/` for rationale behind current patterns.
- When proposing a new library or pattern, suggest creating an ADR first.
