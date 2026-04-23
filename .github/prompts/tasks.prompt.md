---
description: "Break down an approved feature plan into task spec files (Step 3) following the project's SDD workflow"
name: "tasks"
argument-hint: "Feature ID (e.g. 001-feature-name)"
agent: "agent"
---
You are responsible for executing **only Step 3 — Task Breakdown** of this repository's SDD workflow.

This prompt receives a `feature-id` and produces one task spec file per task under `docs/specs/features/<feature-id>/tasks/`.

## Input

{{input}}

## Gate check

1. Locate the feature spec at `docs/specs/features/<feature-id>/feature.spec.md` and read it in full.
2. Locate the feature plan at `docs/specs/features/<feature-id>/plan.spec.md` and read it in full.
3. **If either file does not exist** → stop and inform the user. Suggest using `/feature` (Step 1) or `/plan` (Step 2) first.
4. **If the plan Status is NOT `Approved`** → stop and inform the user. Only plans with status `Approved` can be broken into tasks. Suggest using `/plan` to complete and approve the plan first.
5. **If a `tasks/` folder already exists** with task files inside:
   - Enter **Incremental mode** — read all existing tasks, identify gaps or changes needed, and only create/update tasks as necessary. Do not recreate existing tasks from scratch.

## Required reading

Before starting task breakdown, read and internalize:

1. `docs/project.spec.md` — project context and goals
2. `.github/copilot-instructions.md` — naming conventions, stack, SDD rules, 5-step delivery flow
3. `docs/specs/templates/task.spec.md` — **the task template to fill for every task**
4. The feature spec at `docs/specs/features/<feature-id>/feature.spec.md`
5. The feature plan at `docs/specs/features/<feature-id>/plan.spec.md`
6. All ADRs referenced by the feature spec and plan in `docs/decisions/`
7. The feature's parent domain/area spec if it exists (check `docs/specs/domains/`)
8. The surface-specific architecture spec at `docs/specs/apps/<surface>/architecture.md` when it exists
9. Existing codebase structure relevant to the feature (explore the actual repository layout as needed to understand current state)

## Task breakdown process

### Phase 1 — Analyze the plan and extract task boundaries

1. Read the plan's **Technical Design Baseline** (Section 3.1) and **scope-to-execution mapping** thoroughly.
2. Identify natural task boundaries based on:
   - **Dependency order**: infrastructure before data layer, data layer before domain logic, domain logic before integration.
   - **Module boundaries**: each distinct module (e.g., database package, auth module, users module) can be one or more tasks.
   - **Vertical slices**: when possible, group related backend + test work into a single task for a coherent deliverable.
   - **Documentation tasks**: ADR creation, domain spec creation, architecture doc updates — these are separate tasks.
   - **Frontend design gates**: visually relevant UI implementation tasks depend on an approved Pencil prototype and should not be treated as code-only work.
3. Each task must be **independently implementable and testable** — a developer should be able to pick up a single task file and complete it without needing to read other task files.
4. Identify parallelization opportunities — mark tasks that have no mutual dependencies as parallelizable.

### Phase 2 — Define task sequence and dependencies

1. Build an ordered list of tasks with explicit dependencies (which tasks must be completed before which).
2. Number tasks sequentially: `T001`, `T002`, `T003`, etc.
3. Group tasks into logical phases if applicable (e.g., Setup, Foundational, Slice A, Slice B, QA, Polish).
4. Present the proposed task list to the user **before generating files**. Show:
   - Task ID, title, and plan slice
   - Dependencies between tasks
   - Parallelization opportunities
   - Brief scope summary per task
   - Which tasks should explicitly call for Context7, Playwright, GitHub, or GitKraken MCP usage during implementation

**Ask the user**: "Does this task breakdown look correct? Should I adjust any scope, ordering, or grouping before generating the task files?"

### Phase 3 — Generate task spec files

After user confirmation, generate one file per task:

1. **File path**: `docs/specs/features/<feature-id>/tasks/T<NNN>-<short-title>.task.spec.md`
   - Use kebab-case for `<short-title>` (e.g., `T001-docker-compose-setup.task.spec.md`)
2. **Fill every section** of `docs/specs/templates/task.spec.md` with concrete, specific details pulled from the feature spec and plan.
3. **Status**: Set all tasks to `Draft`.

#### Mandatory detail levels per task

Every task MUST include the following at the level of detail specified. If a section does not apply, write `N/A` with a brief explanation.

- **Section 4.1 — Backend and API Contracts**: Exact endpoint, method, request/response contracts with field names, types, validations, error codes and messages. Copy from the plan — do not summarize or shorten.
- **Section 4.2 — Database Specification**: Exact table names, column names, SQL types, nullability, defaults, PK/FK, constraints, indexes, migration file names, forward/rollback steps. Copy from the plan — do not summarize or shorten. When applicable, also include whether the task uses the shared base entity or equivalent abstraction, the concrete field names used for the six lifecycle metadata semantics, the soft delete representation and query behavior, and any justified hard-delete exception.
- **Section 4.3 — Frontend and UX Contracts**: Routes, components, states, field validations, interaction behavior. Copy from the plan when applicable.
- For visually relevant UI tasks, include the approved Pencil reference, the relevant design token/theme references, and expected responsive behavior.
- **Section 4.4 — Cross-Cutting Constraints**: Security, performance, observability, compatibility relevant to this specific task.
- **Section 5 — Implementation Steps**: Concrete, ordered steps with exact file paths and changes. Not vague — a developer must know exactly what to create/modify.
- **Section 6 — Acceptance Criteria**: Derived from the feature spec's acceptance scenarios. Map each relevant acceptance criterion to this task.
- **Section 7 — Test Scenarios**: Specific test cases with expected behavior for happy path, error path, and edge cases.

#### Traceability

Each task must maintain traceability back to the feature spec and plan:

- **Section 3 — Context from Feature Plan**: Reference the exact plan slice, requirements (FR-XXX), acceptance criteria (AC-XXX), and affected paths.
- **Metadata fields**: `Feature Spec`, `Feature Plan`, `Feature Folder` must link back correctly.
- **Dependencies field**: Reference other task IDs (e.g., `T001`, `T002`) where applicable.

### Phase 4 — Update feature spec and plan status

After all task files are generated:

1. Update the feature spec `Status` from `Planned` to `In Implementation`.
2. Update the plan `Status` from `Approved` to `Ready for Implementation`.
3. Update `Last Updated` dates on both files.

## Incremental mode

If task files already exist in the `tasks/` folder:

1. Read all existing task files.
2. Compare them against the current plan — identify missing tasks, outdated tasks, or scope changes.
3. Ask the user what action to take:
   - **Add missing tasks** — create new task files for gaps.
   - **Update existing tasks** — modify specific tasks based on plan changes.
   - **Full regeneration** — delete all and regenerate (only if user explicitly requests).
4. Do not overwrite or delete existing tasks without user confirmation.

## Exit criteria

- All task files saved under `docs/specs/features/<feature-id>/tasks/` with status `Draft`.
- Feature spec and plan statuses updated.
- Show in the chat:
  - The `feature-id` and task folder path
  - Complete task list with IDs, titles, dependencies, and parallelization info
  - Total number of tasks generated
  - Summary of plan slices covered
- **Ask the user**: "Should I mark any or all tasks as `Ready` for implementation, or do they need review first?"

## Approval transition

If the user confirms tasks are ready:
1. Change the specified task(s) `Status` from `Draft` to `Ready`.
   - For visually relevant UI tasks, do this only if the task includes an approved Pencil reference.
2. Confirm in the chat which tasks are now `Ready` and can proceed to Step 4 (Implementation).

## Critical rules

- **Do NOT write production code** in this step. This is documentation-only.
- **Do NOT skip the user confirmation** in Phase 2 — the task list must be validated before generating files.
- **Do NOT summarize or shorten technical details** from the plan — copy contracts, schemas, and specifications in full into each task that needs them. Tasks must be self-sufficient.
- **Every task must pass Definition of Ready** — review Section 8 of the task template before saving each file.
- **Self-sufficiency is non-negotiable** — a developer must be able to implement a task using only that task file, the feature spec, and the plan. No hidden context.
- **Naming conventions**: Follow the project's documented naming and language conventions (ADR-005).
- **Visually relevant UI tasks require design grounding** — if the plan lacks an approved Pencil reference, stop and fix the plan before marking the task `Ready`.
- **Context7 documentation lookup**: When generating implementation steps (Section 5) that reference specific library APIs, decorators, configuration, or CLI commands, use the Context7 MCP (`mcp_context7_resolve-library-id` → `mcp_context7_get-library-docs`) to verify correctness against current documentation. Do not rely on training data alone for library-specific details.
- **Playwright browser validation**: When a task changes browser-visible behavior, routes, forms, rendered states, or e2e flows, include Playwright MCP-driven validation in Section 5 or Section 7.
- **GitHub and GitKraken context**: When a task depends on issue, PR, review, branch, or repository-state context, encode the relevant GitHub or GitKraken MCP checks directly in the task instead of assuming that context will be remembered later.
