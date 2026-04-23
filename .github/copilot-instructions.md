# Copilot Instructions

## Spec Driven Development

This repository follows **Spec Driven Development (SDD)**. Significant product, architectural, or workflow changes should start from specs and documented decisions before implementation.

### Workflow

1. **Bootstrap first** — When the repository is newly adopting this kit or still contains sample placeholders, run the bootstrap/init flow before normal feature work.
2. **Spec first** — Before writing code, check `docs/specs/` for the relevant spec. If none exists, create or update one.
3. **Decide** — Architectural and workflow decisions are recorded as ADRs in `docs/decisions/`.
4. **Implement** — Write code that fulfills approved specs and plans.
5. **Validate** — Tests and acceptance criteria come from the specs, not from improvisation during implementation.

## Bootstrap Step

### Step 0 — Project Bootstrap / Init

When this kit is copied into a new or existing repository, the first operation should be `/init`.

The bootstrap flow must:

1. Ask the user for the real project context:
   - product or platform name
   - repository topology (monorepo, single repo, multi-service, etc.)
   - stacks, frameworks, runtimes, and major libraries
   - package manager or build tool
   - testing and quality tools
   - frontend design workflow, when a frontend exists
   - naming and language conventions
   - branch and release conventions
   - optional domain/application documentation structure
2. Inspect the repository to confirm what already exists.
3. Materialize the answers into at least:
   - `README.md`
   - `docs/project.spec.md`
   - `docs/architecture.md`
   - `docs/decisions/`
   - `.github/copilot-instructions.md`
4. Remove or retire any bootstrap-only instruction file created solely for the setup flow.

Until bootstrap is complete, avoid treating any sample stack, app name, or branch name in the kit as authoritative project truth.

## Mandatory 5-Step Delivery Flow

After bootstrap, follow this sequence strictly, one step per chat:

1. **Step 1 — Feature Spec Draft**: Determine the next `feature-id` by checking both existing feature folders under `docs/specs/features/` and local/remote `feature/*` branches. Generate the feature spec with `docs/specs/templates/feature.spec.md`, save it under `docs/specs/features/<feature-id>/feature.spec.md`, and set status to `Draft`. After saving the spec, invoke the `sdd-branch` skill to create and publish `feature/<feature-id>` from the configured integration branch.
2. **Step 2 — Feature Plan**: After feature spec approval, update feature status to `Approved` and generate the feature plan using `docs/specs/templates/plan.spec.md`. Save the plan at `docs/specs/features/<feature-id>/plan.spec.md`. The plan must be concrete enough to generate self-sufficient tasks. Keep this step documentation-only.
3. **Step 3 — Task Breakdown**: After the feature plan is approved, update the plan status to `Approved` and derive tasks using `docs/specs/templates/task.spec.md`. Save each task under `docs/specs/features/<feature-id>/tasks/`. Keep this step documentation-only.
4. **Step 4 — Task Implementation**: Only after the selected task spec is approved and `Ready` should implementation begin. Before starting, invoke the `sdd-branch` skill to create `task/<task-id>` from the feature branch. Update task status during execution and, after completion, offer a commit via `sdd-commit`. After the commit, invoke `sdd-branch` to merge the task branch back into the feature branch and clean up.
5. **Step 5 — Feature Finish**: After all tasks for the feature are `Done`, run the retrospective, capture learnings, update templates/instructions/ADRs when approved, and invoke `sdd-branch` to merge the feature branch back into the configured integration branch.

## Checklist-Gated Status Transitions

A spec's status MUST NOT change unless every checkbox in its corresponding checklist is checked.

- **Feature spec**: Gate checklists in section `Workflow and Status Gates` must be complete.
- **Plan spec**: `Planning Gates` and `Step 2 Completion Checklist` must be complete.
- **Task spec**: `Definition of Ready` and `Definition of Done` must be complete before the corresponding status change.

If any checkbox cannot be checked, the status transition is blocked.

## Dependency-Aware Task Status Rule

A task's status MUST reflect its dependency state:

- **`Awaiting Dependency`**: The task is approved but one or more dependencies are not `Done`.
- **`Ready`**: The task is approved and all dependencies are `Done`, or it has no dependencies.

When a task is marked `Done`, re-check dependent tasks and move them from `Awaiting Dependency` to `Ready` when all dependencies are satisfied.

## Owner Auto-Population

The **Owner** field in every spec is auto-filled from `git config user.name`.

- **Feature spec**: owner is the person creating the feature spec.
- **Plan spec**: owner is the person creating the plan.
- **Task spec**: owner is the person who starts implementation. During task breakdown, owner may stay `TBD` until implementation begins.

## Plan Mode Operational Rule

When working in plan mode through Step 3:

1. Run Steps 1, 2, and 3 in plan mode for drafting and approvals.
2. Before Step 4, run a materialization chat in write-enabled mode to persist approved artifacts exactly as approved.
3. In that chat, only write or update spec files and statuses under `docs/specs/features/<feature-id>/`.
4. Start implementation only after materialization is complete and at least one task is `Ready`.

## Continuous SDD Sharpening Loop

Triggered by **Step 5 — Feature Finish**.

1. Ask what worked well.
2. Ask where friction or rework happened.
3. Ask whether any specs, plans, or tasks were unclear.
4. Ask whether implementation deviated from the plan and why.
5. Promote proven patterns into templates, instructions, or ADRs.

Keep this lightweight. Prefer small, evidence-based improvements.

## MCP Usage in SDD

This kit expects deliberate use of MCP surfaces when they reduce ambiguity.

- **Do not replace mandatory SDD skills** when a skill is explicitly required.
- **Use Context7** for stack-specific APIs, framework configuration, or library behavior relevant to the actual project.
- **Use Playwright** when the work depends on real browser-visible behavior.
- **Use Pencil** when frontend or UI work depends on prototypes, design artifacts, tokens, or layout decisions.
- **Use GitHub or GitKraken** when issues, PRs, reviews, releases, or repository state materially affect the work.

## Reading Order

Start from:

1. `docs/project.spec.md`
2. `docs/architecture.md`
3. `docs/specs/templates/`
4. relevant feature, plan, or task artifacts
5. relevant ADRs in `docs/decisions/`
6. optional `docs/specs/domains/` and `docs/specs/apps/` docs when they exist

## Impact Review on Foundational Changes

Whenever a foundational document is added, removed, or materially changed, review all open specs for alignment before changing them.

Foundational documents include:

- `docs/project.spec.md`
- `docs/architecture.md`
- `docs/specs/apps/<app>/architecture.md`
- `docs/specs/domains/*.md`
- `docs/decisions/*.md`

Review process:

1. Scan `docs/specs/features/` for features whose status is not `Done`.
2. Check each open feature's spec, plan, and task files.
3. Flag any artifact that conflicts with or should reference the changed foundational document.
4. Present the impact summary to the user.
5. Only apply updates after user approval.

## Project Context

This file is part of a reusable kit. The consuming repository must replace the sample context during `/init`.

Document here after bootstrap:

- repository topology
- main deployable surfaces (apps, services, packages, libraries, etc.)
- optional domain or module documentation structure
- key stack/tooling choices

If the project uses Nx with app-oriented full-stack grouping, document whether it follows:

- `apps/<app>/backend`
- `apps/<app>/frontend`
- colocated unit/integration tests inside each stack
- stack-root end-to-end test folders such as `backend/test/e2e` and `frontend/e2e`

## Stack and Tooling

Do not assume a fixed stack.

After bootstrap, this section should record the actual project choices for:

- languages and runtimes
- frameworks and libraries
- package manager or build tool
- test tooling
- linting/formatting/static analysis tooling
- CI and release surfaces

## Naming Conventions

Do not assume Portuguese database names, English-only code, or any other hardcoded convention.

After bootstrap, this section should document the project's naming matrix in line with ADR-005.

## Frontend and Design Workflow

If the project includes frontend work, bootstrap should document at least:

- where design artifacts live (for example `design/`)
- where reusable UI code lives (package, library, module, or equivalent)
- where global theme and token definitions live
- whether mobile-first is the default responsive posture
- when Pencil is required for frontend or UI work

Baseline rules for frontend/UI work:

- Use Pencil for changes with meaningful visual or UX impact.
- Frontend/UI plans with relevant visual impact should include an approved Pencil prototype before plan approval.
- UI tasks should not move to `Ready` without an approved Pencil reference.
- If implementation uncovers a meaningful visual change, update Pencil first and then replicate the change in code.
- Do not let production code become the source of truth for unresolved design decisions.

## Nx App Layout (When Applicable)

If the project uses Nx and groups work by application, the default baseline may be:

- `apps/<app>/backend`
- `apps/<app>/frontend`

Test placement baseline for that structure:

- backend unit and integration tests colocated with implementation files
- backend e2e tests under the backend root test area
- frontend unit and page integration tests colocated with implementation files
- frontend e2e tests under the frontend root e2e area

Do not assume this structure unless bootstrap confirms the project uses it.

## Commit Conventions

Commits follow **Conventional Commits** with **Gitmoji** prefixes.

Format:

```text
<gitmoji> <type>(<scope>): <short description>

<optional body>

Refs: <task-spec-path>
```

- Use the `sdd-commit` skill.
- Always include a `Refs:` footer linking to the relevant task or feature spec.
- Keep commits atomic.
- Never auto-commit without user approval.

## Branching Strategy

This project uses an **SDD-aligned branching strategy** (ADR-009) managed by the `sdd-branch` skill.

After bootstrap, this section should state:

- the integration branch name
- the release branch or trunk
- any project-specific merge or publication rules

Do not hardcode `sandbox`, `develop`, or similar names unless the project has explicitly chosen them.

## Code Guidelines

- Follow existing repository patterns before introducing new ones.
- Check `docs/decisions/` for relevant rationale.
- If a new long-lived library, pattern, or workflow rule is introduced, consider creating or updating an ADR.

## Quality and Validation Gates

### Quality Tooling Gate

After writing or modifying source code, run the project's canonical validation commands as documented during bootstrap. Do not assume `pnpm nx run-many -t check` or any other specific command unless the repository has declared it as canonical.

### Test Pass Gate

A task MUST NOT be marked `Done` unless all related tests pass.

- Tests must be actually executed.
- If tests fail, the task stays `In Progress`.
- If required environment dependencies are unavailable, stop and alert the user rather than silently skipping.

### Test Evidence Gate

Every task spec contains a **Test Evidence** table.

- Before marking a task `Done`, populate that table with the exact command(s) run and the observed result summary.
- If the section is empty or contains failures, the task cannot transition to `Done`.

## Documentation Lookup (Context7)

Use Context7 whenever the current task depends on library, framework, tool, or platform behavior that may vary by version or stack.

- Do not rely solely on training data for stack-specific APIs.
- During planning and implementation, verify library-specific details against current documentation.
- Resolve ambiguity before proceeding when a version or API change matters.