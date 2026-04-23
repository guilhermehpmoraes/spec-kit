---
description: "Implement a task spec (Step 4) — writes production code for an approved and Ready task following the project's SDD workflow"
name: "implement"
argument-hint: "Feature ID and Task ID (e.g. 001 T001)"
agent: "agent"
---
You are responsible for executing **Step 4 — Task Implementation** of this repository's SDD workflow.

This prompt receives a `feature-id` (numeric, e.g. `001`) and a `task-id` (e.g. `T001`), locates the task spec, and implements it fully — writing production code, tests, and all changes described in the task.

## Input

{{input}}

Parse the input to extract:
- **Feature ID** (numeric portion, e.g. `001`)
- **Task ID** (e.g. `T001`)

Use glob matching to find the feature folder that starts with the feature number under `docs/specs/features/`.

## Gate check

1. Find the feature folder matching the feature number under `docs/specs/features/` (e.g. `docs/specs/features/001-*/`).
2. Read the feature spec at `docs/specs/features/<feature-id>/feature.spec.md`.
3. Read the feature plan at `docs/specs/features/<feature-id>/plan.spec.md`.
4. Find and read the task spec matching the task ID under `docs/specs/features/<feature-id>/tasks/` (e.g. `T001-*.task.spec.md`).
5. **If any file does not exist** → stop and inform the user. Suggest using the appropriate SDD step prompt (`/feature`, `/plan`, `/tasks`).
6. **If the task is a visually relevant UI task and does not reference an approved Pencil artifact** → stop and inform the user. The task must be updated before implementation.
7. **If the task Status is NOT `Ready`** → **stop** and inform the user. Only tasks with status `Ready` can be implemented. Show the current status and suggest:
   - If `Draft` → ask the user to review and mark it as `Ready` first (via `/tasks` prompt or manually).
   - If `In Progress` → inform implementation is already underway.
   - If `Done` → inform the task is already completed.
   - If `Blocked` → inform the task is blocked and show the reason if available.

## Required reading

Before writing any code, read and internalize:

1. The task spec file (the primary implementation guide — **this is your blueprint**)
2. The feature spec (`feature.spec.md`) — for business context and acceptance scenarios
3. The feature plan (`plan.spec.md`) — for technical design baseline and architecture decisions
4. `.github/copilot-instructions.md` — naming conventions, stack, SDD rules
5. `docs/project.spec.md` — project context
6. All ADRs referenced by the task spec in `docs/decisions/`
7. The surface-specific architecture spec at `docs/specs/apps/<surface>/architecture.md` when the feature references one
8. The domain/area spec if referenced (check `docs/specs/domains/`)
9. Any design references named in the task spec (for example Pencil artifacts, token docs, or design-system notes)
10. Existing codebase files that will be modified (read them before editing)

## Dependency check

Before starting implementation:

1. Check the task's **Dependencies** field (Section metadata).
2. For each dependency task ID listed:
   - Find and read that task's spec file.
   - **If the dependency task Status is NOT `Done`** → **stop** and inform the user. List which dependency tasks are not yet completed and their current statuses.
3. Only proceed if all dependency tasks are `Done` or the task has no dependencies.

## Implementation process

### Phase 1 — Context and clarification

1. Read all files listed in **Required reading**.
2. Read the task's **Section 5 — Implementation Steps** carefully. This is your step-by-step guide.
3. Read the task's **Section 4 — Technical Specification** for contracts, schemas, and constraints.
4. Explore the current codebase to understand existing patterns, module structure, and conventions.
5. If anything in the task spec is ambiguous, contradictory, or incomplete — **ask the user before proceeding**. Do not guess or make assumptions about unclear requirements.
6. If the task depends on remote issue or PR context, fetch it with the GitHub or GitKraken MCP before implementing.
7. If the task changes browser-visible behavior, inspect the current flow with the Playwright MCP before editing so the expected behavior is grounded in evidence.
8. If the task has an approved Pencil reference, inspect the relevant Pencil artifact before editing so the implementation is anchored to the approved design.

### Phase 2 — Implementation

Execute the implementation steps from the task spec (Section 5) in order. For each step:

1. **Follow the task spec precisely** — implement exactly what is specified. Do not add features, refactor unrelated code, or make "improvements" beyond the task scope.
2. **Follow existing codebase patterns** — match the style, structure, and conventions already in use.
3. **Follow naming conventions** — use the project's documented naming and language rules (ADR-005).
4. **Use Context7 documentation lookup** when implementing library-specific code — verify API signatures and configuration against current docs.
5. **Use Playwright MCP** when implementing or validating browser-visible behavior, route transitions, forms, or e2e flows.
6. **Use GitKraken MCP** when you need repository status, diff, branch, worktree, or PR awareness during implementation.
7. **Use GitHub MCP** when implementation depends on remote issue, pull request, review, or release context.
8. **Create files and directories** as specified in the task's affected paths.
9. **Write tests** as specified in Section 7 — Test Scenarios. Follow existing test patterns in the codebase.
10. **Run tests** after writing them to verify they pass.
11. **Run linting** to ensure code quality compliance.
12. **If implementation reveals a meaningful visual or UX change not reflected in Pencil** — stop, update Pencil first, then continue coding from the updated design.

### Phase 3 — Validation

After implementation is complete:

1. **Run all tests** related to the changes (unit, integration, e2e as applicable).
2. **Run linting/formatting** using the project's canonical validation commands documented during bootstrap.
3. **Verify acceptance criteria** — check each criterion from Section 6 against what was implemented.
4. **Check for compile errors** across affected projects.
5. **For visually relevant UI work, verify the delivered UI against the approved Pencil design** and update the design first if they diverge.
6. If any validation fails — fix the issue and re-validate.

### Phase 4 — Status updates

After successful validation:

1. **Update the task spec**:
   - Change `Status` from `Ready` to `In Progress`, then to `Done`.
   - Update `Last Updated` date.
   - Add entries to **Section 12 — Status Log** for both transitions.
   - Fill in **Section 9 — Definition of Done** checkboxes.

2. **Update the feature spec** (if applicable):
   - If this is the first task being implemented and the feature status is `Planned` or `In Implementation` → ensure status is `In Implementation`.
   - If this is the **last task** in the feature (all tasks are `Done`) → update feature status to `Done`.
   - Update `Last Updated` date.

3. **Update the plan spec** (if applicable):
   - Update `Last Updated` date.

## Exit criteria

- All code changes from the task spec are implemented.
- Tests pass.
- Linting passes.
- Task status is `Done`.
- Feature/plan statuses are updated accordingly.
- Show in the chat:
  - The `feature-id`, `task-id`, and task title
  - Summary of files created/modified
  - Test results summary
  - Any decisions or deviations made during implementation (with justification)
  - Updated status of the task
  - Whether there are remaining tasks in the feature and their statuses

## Critical rules

- **Only implement `Ready` tasks** — never start a task that is not in `Ready` status.
- **Follow the task spec as the blueprint** — do not invent scope, skip steps, or add unspecified features.
- **Ask when in doubt** — if the task spec is ambiguous or conflicts with the codebase, ask the user rather than guessing.
- **Do not modify other task scopes** — if you discover something that needs fixing outside this task's scope, note it but do not fix it. Inform the user.
- **Naming conventions are non-negotiable** — follow the project's documented naming matrix (ADR-005).
- **Self-sufficiency principle** — the task spec should contain everything needed. If it doesn't, that's a gap to flag, not to fill silently.
- **Pencil is the design source of truth for visually relevant UI work** — do not resolve meaningful visual decisions only in code.
- **If visual scope changes, Pencil changes first** — update the prototype before continuing implementation.
- **Context7 documentation lookup**: For every library-specific implementation, use Context7 MCP to verify correctness against current documentation.
- **Playwright validation** — when frontend or e2e behavior changes, use Playwright MCP to validate the real browser flow instead of relying only on static code inspection.
- **GitHub and GitKraken context** — when issue, PR, review, branch, or repository-state context matters, use the GitHub or GitKraken MCP surfaces rather than guessing.
- **SDD skills remain mandatory** — `sdd-branch` still governs branch lifecycle steps and `sdd-commit` still governs commit composition and approval.
- **Test coverage** — implement all test scenarios from Section 7. Do not skip tests.
- **Atomic progress** — update status transitions as they happen. Mark `In Progress` when starting, `Done` when finished.
