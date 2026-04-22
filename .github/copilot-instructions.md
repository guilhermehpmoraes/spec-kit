# Copilot Instructions

## Spec Driven Development

This project follows **Spec Driven Development (SDD)**. Every feature, module, or architectural change starts from a spec before any code is written.

### Workflow

1. **Spec first** — Before writing code, check `docs/specs/` for the relevant spec. If none exists, create or update one.
2. **Decide** — Architectural decisions are recorded as ADRs in `docs/decisions/`. Reference the relevant ADR number in specs and code when applicable.
3. **Implement** — Write code that fulfills the spec. Reference the spec path in PR descriptions and commit messages when relevant.
4. **Validate** — Tests and acceptance criteria come from the spec, not invented during implementation.

## Project Init (Step 0)

Before the 5-step feature delivery flow, this kit supports a one-time **project initialization** flow for new repositories.

When the user asks to initialize the Spec Kit (for example: `quero iniciar o spec kit`), do this before creating any feature spec:

1. Ask for the project summary, goals, and product/application context.
2. Ask for the selected stack by layer: backend, frontend, mobile, data, testing, and CI.
3. Ask whether the default **Nx monorepo** baseline should be kept, adapted, or replaced.
4. Ask whether the architecture baseline in this kit should be kept, adapted, or rewritten.
5. Ask which applications/products will exist and which shared packages or shared layers are expected.
6. If the project has frontend scope, ask for:
   - mobile-first expectations
   - shared component library expectations
   - global CSS/theme token expectations
   - Pencil usage for component catalog and feature/page designs
7. Use `docs/specs/templates/project-init.spec.md` as the default structure for capturing the init answers.
8. Materialize the initialization by creating or updating:
   - `docs/project.spec.md`
   - `docs/architecture.md`
   - `docs/specs/apps/<app>/architecture.md` when needed
   - Pencil artifacts when frontend planning/design baselines are part of the project

This init step makes the kit portable across stacks such as Java + Angular, NestJS + React, or other combinations while preserving the same SDD workflow.

### Mandatory 5-Step Delivery Flow

Follow this sequence strictly, one step per chat:

1. **Step 1 — Feature Spec Draft**: When the user provides feature input (text, notes, design doc, requirement doc), determine the next `feature-id` by checking both the existing feature folders under `docs/specs/features/` and the existing local or remote `feature/*` branches, then generate the feature spec using `docs/specs/templates/feature.spec.md` and save it as a draft file under `docs/specs/features/<feature-id>/feature.spec.md`. Set status to `Draft`. **After saving the spec, invoke the `sdd-branch` skill to create and publish `feature/<feature-id>` from the integration branch** (see ADR-009). Do not edit or create production code.
2. **Step 2 — Feature Plan**: After feature spec approval, update feature status to `Approved` and generate the feature plan using `docs/specs/templates/plan.spec.md`. Save the plan at `docs/specs/features/<feature-id>/plan.spec.md`. The plan must be highly specific and technical: list exact files to create/modify, libraries to install, database changes, API contracts, UI contracts, and any other concrete implementation detail. **If the feature affects UI, routes, forms, layouts, or shared frontend components, create or update the Pencil design artifacts during this step and record the `.pen` file and covered screens/components in the plan.** Keep this step documentation-only plus approved design artifacts (no production code changes, no task files yet).
3. **Step 3 — Task Breakdown**: After the feature plan is approved, update plan status to `Approved` and derive tasks from the plan using `docs/specs/templates/task.spec.md`. Save each task as its own file under `docs/specs/features/<feature-id>/tasks/`. Every generated task MUST include detailed technical implementation data (for example: database tables/fields/types/constraints/indexes/migrations when applicable, and API/UI contracts where applicable). **UI tasks must also include the exact `.pen` references that implementation must follow.** Keep this step documentation-only (no production code changes).
4. **Step 4 — Task Implementation**: Only after the selected task spec is approved should implementation begin for that task. **Before starting, invoke the `sdd-branch` skill to create `task/<task-id>` from the feature branch.** Update task status during execution (`In Progress` -> `Done`) and keep links to the feature spec and feature plan. **After completing the task**, ask the user if they want to commit the changes. If they accept, invoke the `sdd-commit` skill to stage and commit following Conventional Commits + Gitmoji conventions, referencing the task and feature specs. **After the commit, invoke the `sdd-branch` skill to merge the task branch back into the feature branch and clean up.** **When the last task of a feature is marked `Done`, also update the plan spec status to `Completed`** — this ensures the plan is always in sync before Step 5.
5. **Step 5 — Feature Finish**: After all tasks for the feature are `Done`, run the SDD Sharpening Loop retrospective (see section below). Ask the user targeted questions to capture outcomes: what worked well, what caused friction, any patterns worth promoting, and concrete process improvements. Update the feature spec status to `Done`. Record retrospective findings and apply approved improvements to templates, instructions, or ADRs for the next cycle. **After all retrospective changes are committed, invoke the `sdd-branch` skill to merge the feature branch into the integration branch and clean up** (see ADR-009) — this is the last action of the feature lifecycle.

### Checklist-Gated Status Transitions

A spec's **status MUST NOT change** unless every checkbox in its corresponding checklist is checked. This applies to:

- **Feature spec**: Gate checklists in section "Workflow and Status Gates" must be fully checked before moving to the next step.
- **Plan spec**: "Planning Gates" (section 4) must be fully checked before approval. "Step 2 Completion Checklist" (section 11) must be fully checked before handing off to Step 3.
- **Task spec**: "Definition of Ready" must be fully checked before setting status to `Ready`. "Definition of Done" must be fully checked before setting status to `Done`.

If any checkbox cannot be checked, the status transition is blocked. Resolve the gap first, then transition.

### Dependency-Aware Task Status Rule

A task's status MUST reflect its dependency state:

- **`Awaiting Dependency`**: The task spec is approved and complete, but one or more dependency tasks are NOT yet `Done`. The task cannot be picked up for implementation.
- **`Ready`**: The task spec is approved AND all dependency tasks are `Done` (or the task has no dependencies). Only then can it be picked up for Step 4.

During **Step 3 (Task Breakdown)**:
- Tasks with **no dependencies** start as `Ready` (if approved).
- Tasks **with unfinished dependencies** start as `Awaiting Dependency`.

During **Step 4 (Task Implementation)**:
- When a task is marked `Done`, check all tasks that depend on it. If a dependent task's **all** dependencies are now `Done`, automatically transition it from `Awaiting Dependency` to `Ready`.

### Owner Auto-Population

The **Owner** field in every spec is auto-filled using `git config user.name` — never leave it as `TBD` or a placeholder.

- **Feature spec (Step 1)**: Set Owner to the `git config user.name` of the person creating the feature spec.
- **Plan spec (Step 2)**: Set Owner to the `git config user.name` of the person creating the plan spec.
- **Task spec (Step 4)**: Set Owner to the `git config user.name` of the person who starts implementation (i.e., when they pick up the task). During task breakdown (Step 3), leave Owner as `TBD` since the implementer is not yet known.

### Plan Mode Operational Rule

When working in **plan mode** through Step 3, use this execution pattern:

1. Run Steps 1, 2, and 3 in plan mode for drafting and approvals.
2. Before Step 4, run a **Materialization chat** in normal (write-enabled) mode to persist approved artifacts exactly as approved, without changing scope/content.
3. In Materialization chat, only write/update spec files and statuses under `docs/specs/features/<feature-id>/`.
4. Start implementation only after materialization is complete and at least one task is `Ready`.

### Continuous SDD Sharpening Loop

Triggered by **Step 5 — Feature Finish**. Run a short retrospective and evolve the process:

1. **Ask questions** — Prompt the user with targeted questions:
   - What went smoothly during this feature?
   - Where did friction or rework happen?
   - Were any specs unclear or incomplete?
   - Did any implementation deviate from the plan? Why?
   - Are there patterns worth standardizing?
2. **Review outcomes** — Capture what worked well and what caused friction based on user answers.
3. **Promote patterns** — Convert proven practices into stable rules/templates.
4. **Fix weak spots** — Add one or two concrete process improvements for the next cycle.
5. **Apply changes** — Update templates, copilot instructions, or propose ADRs as needed. Use the updated rules in the following feature.

Keep this lightweight: prefer small, evidence-based changes over large process rewrites.

## MCP Usage in SDD

This workspace has five MCP surfaces that should be used deliberately during the SDD workflow: **GitKraken**, **GitHub**, **Playwright**, **Context7**, and **Pencil**.

- **Do not replace mandatory SDD skills** with MCPs when a skill is explicitly required. `sdd-branch` still governs branch lifecycle steps, and `sdd-commit` still governs commit composition and approval.
- **Step 1 — Feature Spec Draft**: If the request references a GitHub issue, pull request, review thread, or discussion, use the GitHub or GitKraken MCP to fetch the canonical remote context before drafting or refining the spec. Use Playwright MCP when the feature request depends on understanding current browser behavior, screens, or UX regressions. Use Context7 if the feature scope depends on concrete library or framework constraints.
- **Step 2 — Feature Plan**: Always use Context7 for library, framework, and tooling decisions. Use Pencil MCP whenever the feature changes UI, shared components, layouts, forms, or navigation so the design is produced and validated during planning. Use Playwright MCP to inspect current UI flows, routes, forms, and rendered states when planning frontend or e2e work. Use GitHub or GitKraken MCP to pull linked issue or PR discussions, acceptance notes, and repository state when those affect the plan.
- **Step 3 — Task Breakdown**: Carry MCP-backed findings into the task specs. If a task requires browser validation, record Playwright-driven validation in the implementation steps or test scenarios. If a task depends on approved UI design, encode the exact Pencil file and screen/component references directly in the task. If a task depends on issue, PR, or review context, encode the relevant GitHub or GitKraken findings directly in the task.
- **Step 4 — Task Implementation**: Use Context7 for library-specific implementation details, Pencil MCP as the source of truth for approved UI artifacts, Playwright MCP for browser validation and UI reproduction, GitKraken MCP for repository status, diff, branch, and PR awareness, and GitHub MCP for remote issue or pull request context when needed. Prefer MCP-backed repository and browser inspection over assumptions.
- **Step 5 — Feature Finish**: Use GitHub or GitKraken MCP to review PR feedback, outstanding follow-ups, merge context, and other remote delivery signals that matter to the retrospective. Use Playwright MCP if retrospective findings involve browser validation gaps or UI regressions.

### Reading specs

- Start with `docs/project.spec.md` for high-level project understanding.
- Templates live in `docs/specs/templates/`.
- Use `docs/specs/templates/feature.spec.md` as the default starting point for new feature specs.
- Use `docs/specs/templates/plan.spec.md` to plan the approved feature before implementation.
- Use `docs/specs/templates/task.spec.md` to create one task file per task derived from the approved feature plan.
- Store each feature package in `docs/specs/features/<feature-id>/` with `feature.spec.md`, `plan.spec.md`, and `tasks/`.
- Domain specs live in `docs/specs/domains/`. Feature specs reference their parent domain.
- Application-specific architecture specs live in `docs/specs/apps/<app>/`.

### Reading decisions

- ADRs are numbered sequentially: `docs/decisions/NNN-title.md`.
- Always check existing ADRs before proposing conflicting approaches.

### Impact Review on Foundational Changes

Whenever a **foundational document** is created or modified, automatically review all **open** specs (features, plans, and tasks not yet `Done`) for alignment. Foundational documents include:

- `docs/project.spec.md`
- `docs/architecture.md`
- `docs/specs/apps/<app>/architecture.md`
- `docs/specs/domains/*.md`
- `docs/decisions/*.md` (ADRs)

**Trigger**: Any commit or edit that adds, updates, or removes content in the files above.

**Review process**:

1. Scan `docs/specs/features/` for features with status other than `Done`.
2. For each open feature, check its `feature.spec.md`, `plan.spec.md`, and `tasks/*.task.spec.md`.
3. Flag any spec that **conflicts with**, **is outdated by**, or **should reference** the changed foundational document.
4. Present a summary to the user listing affected specs and what needs updating.
5. Only apply changes to specs after user approval.

This ensures that architectural decisions, domain boundaries, and project-level changes propagate consistently to in-flight work.

## Project Context

- This repository is a reusable **Spec Kit**, not a fixed product implementation.
- The default bias is an **Nx monorepo** with apps in `apps/` and shared packages in `packages/`, but init may adapt that baseline.
- The kit must stay usable across different stack combinations, including examples like Java + Angular and NestJS + React.
- Each initialized project should record its own application and architecture details in `docs/project.spec.md` and `docs/specs/apps/<app>/`.
- Domain specs live in `docs/specs/domains/` when the project uses DDD or equivalent module-level documentation.

## Stack

- **Default repo bias**: Nx monorepo
- **Default package manager when using Nx/JS tooling**: pnpm
- **Supported stack model**: choose the project stack during init instead of assuming one fixed backend/frontend combination
- **Frontend design workflow**: Pencil for component catalog and UI planning whenever frontend scope exists
- **Testing**: project-specific, documented during init and refined during planning

## Naming Conventions

- Docs, specs, prompts, and ADRs should stay in English unless the initialized project explicitly chooses otherwise.
- Source-code naming and database naming are project-specific decisions that must be recorded during init.
- If the project uses Nx, follow `docs/decisions/008-project-naming-conventions.md` unless init defines a different convention.

## Frontend Planning Baseline

If a feature includes frontend or UI work, apply these defaults unless the project baseline says otherwise:

- Mobile first.
- Shared component library for reuse across apps.
- Reusable and configurable components preferred over many small variants.
- Pencil component catalog maintained as a visual reference.
- Design changes happen in Step 2 planning so Step 4 implementation follows approved `.pen` artifacts.
- Global CSS/theme tokens define the visual system.

## Commit Conventions

All commits follow **Conventional Commits** with **Gitmoji** prefixes. Format:

```
<gitmoji> <type>(<scope>): <short description>

<optional body>

Refs: <task-spec-path>
```

- Use the `sdd-commit` skill for composing and executing commits.
- Always include a `Refs:` footer linking to the relevant task/feature spec.
- Keep commits atomic — one logical change per commit.
- Never auto-commit without user approval.

## Branching Strategy

This project uses an **SDD-aligned branching strategy** (ADR-009) managed by the `sdd-branch` skill. Git Flow CLI is **not** used.

- **Integration branch**: `sandbox` (will become `develop` after bootstrap phase)
- **Feature branches**: `feature/<feature-id>` — created from the integration branch at Step 1
- **Task branches**: `task/<task-id>` — created from the feature branch at Step 4
- **Feature numbering**: choose the next `feature-id` by inspecting both `docs/specs/features/` and existing local/remote `feature/*` branches; use the next unused numeric prefix so folders and branches stay aligned.
- Merges use `--no-ff` to preserve history boundaries
- No rebasing — merges only for clear audit trail
- See the `sdd-branch` skill for detailed operations and the ADR for rationale

## Code Guidelines

- Follow existing patterns in the codebase before introducing new ones.
- Check `docs/decisions/` for rationale behind current patterns.
- When proposing a new library or pattern, suggest creating an ADR first.

### Biome Compliance (Mandatory)

After writing or modifying any source code, run Biome and resolve **all** warnings and errors before considering the change complete:

```bash
pnpm nx run-many -t check
```

- Never leave code in a non-compliant state — warnings are not acceptable.
- If Biome reports issues, fix them immediately as part of the current task.
- Do not mark a task as `Done` with outstanding Biome warnings or errors.

### Test Pass Gate (Mandatory)

A task MUST NOT be marked `Done` unless **all** related tests pass — both unit and integration:

- **Tests MUST be actually executed** — never assume tests pass without running them. Observing the test command output is mandatory.
- Run the relevant test suite before transitioning a task to `Done`.
- If any test fails, the task stays `In Progress` until all failures are resolved.
- Never mark a Definition of Done checkbox as complete if the corresponding tests have not been executed and verified as passing.
- When a task includes test scenarios in its spec, every listed scenario must have a passing test.
- **Environment failures are NOT a reason to skip tests.** If the development environment (Docker services, database, Redis, etc.) is not running or misconfigured and tests cannot execute, **stop immediately** and alert the user to fix the environment. Do not proceed with marking the task as Done.
- Never silently skip, ignore, or assume test results — every test run must produce visible, verified output.

### Test Evidence Gate (Mandatory)

Every task spec contains a **Section 10 — Test Evidence** table. This section serves as auditable proof that tests were executed and passed.

- **Before marking a task as `Done`**, populate the Test Evidence table with the exact command(s) run and their pass/fail/skip output summary.
- The Definition of Done checkbox for Test Evidence is **blocking** — if the section is empty or shows failures, the task cannot transition to `Done`.
- Never pre-fill this section during planning (Steps 1–3). It is populated only during Step 4.
- This is non-negotiable: build and lint passing alone do NOT satisfy the test gate.

## Documentation Lookup (Context7)

This workspace has the **Context7 MCP** available (`mcp_context7_resolve-library-id`, `mcp_context7_get-library-docs`).

- **Always use Context7** to fetch up-to-date documentation before writing or planning code that depends on any library, framework, or tool in the stack (NestJS, TypeORM, TanStack Router, TanStack Query, Vite, Tailwind CSS, Playwright, Jest, Vitest, React Testing Library, Biome, etc.).
- Do NOT rely solely on training data for API signatures, configuration options, decorator usage, or migration guides — verify against current docs via Context7.
- During **planning** (Step 2) and **task implementation** (Step 4), consult Context7 for every library-specific decision: correct decorator syntax, module configuration, query/mutation patterns, test utilities, CLI flags, etc.
- When a library version is ambiguous or a breaking change is suspected, resolve it through Context7 before proceeding.
