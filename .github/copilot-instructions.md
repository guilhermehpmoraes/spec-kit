# Copilot Instructions

## Spec Driven Development

This project follows **Spec Driven Development (SDD)**. Every feature, module, or architectural change starts from a spec before any code is written.

### Bootstrap Mode (`docs/init.md`)

If `docs/init.md` exists and the user is adapting this kit to a new project, run the initialization workflow before Step 1.

Initialization is a one-time bootstrap operation used to:

- define the real project identity and repository shape
- configure stack, languages, runtimes, and tooling
- generalize or scope ADRs correctly
- update foundational docs to describe the actual project

Initialization is complete only when:

1. foundational docs reflect the target project rather than the template
2. stack-specific decisions are scoped correctly
3. stale example material is archived, removed, or clearly left as reference
4. `docs/init.md` is deleted

### Workflow

1. **Spec first** — Before writing code, check `docs/specs/` for the relevant spec. If none exists, create or update one.
2. **Decide** — Architectural decisions are recorded as ADRs in `docs/decisions/`. Reference the relevant ADR number in specs and code when applicable.
3. **Implement** — Write code that fulfills the spec. Reference the spec path in PR descriptions and commit messages when relevant.
4. **Validate** — Tests and acceptance criteria come from the spec, not invented during implementation.

### Mandatory 5-Step Delivery Flow

Follow this sequence strictly, one step per chat:

1. **Step 1 — Feature Spec Draft**: When the user provides feature input (text, notes, design doc, requirement doc), determine the next `feature-id` by checking both the existing feature folders under `docs/specs/features/` and the existing local or remote `feature/*` branches, then generate the feature spec using `docs/specs/templates/feature.spec.md` and save it as a draft file under `docs/specs/features/<feature-id>/feature.spec.md`. Set status to `Draft`. **After saving the spec, invoke the `sdd-branch` skill to create and publish `feature/<feature-id>` from the integration branch** (see ADR-009). Do not edit or create production code.
2. **Step 2 — Feature Plan**: After feature spec approval, update feature status to `Approved` and generate the feature plan using `docs/specs/templates/plan.spec.md`. Save the plan at `docs/specs/features/<feature-id>/plan.spec.md`. The plan must be highly specific and technical: list exact files to create/modify, libraries to install, database changes, API contracts, UI contracts, and any other concrete implementation detail. Keep this step documentation-only (no production code changes, no task files yet).
3. **Step 3 — Task Breakdown**: After the feature plan is approved, update plan status to `Approved` and derive tasks from the plan using `docs/specs/templates/task.spec.md`. Save each task as its own file under `docs/specs/features/<feature-id>/tasks/`. Every generated task MUST include detailed technical implementation data (for example: database tables/fields/types/constraints/indexes/migrations when applicable, and API/UI contracts where applicable). Keep this step documentation-only (no production code changes).
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

- This repository is a reusable SDD kit that may back a single project, a monorepo, or a mixed-language workspace.
- Applications and services are runtime surfaces; domains are optional bounded contexts documented when useful (see ADR-004).
- When the workspace is a monorepo, apps and shared packages should be documented explicitly in architecture docs.
- Each application or service can have its own architecture spec at `docs/specs/apps/<app>/`.
- Domain specs live in `docs/specs/domains/` when the project uses them.

## Stack

- Stack choices are project-specific and must be documented during initialization.
- Preferred defaults for many projects using this kit:
   - monorepo with Nx when multiple apps or packages are expected
   - pnpm for JavaScript and TypeScript workspaces
   - Biome for supported frontend and JS/TS surfaces
- Mixed-language workspaces are valid. Example: Java backend with Biome on frontend only.
- App or service architecture docs are the source of truth for framework-specific guidance.

## Naming Conventions

- Naming conventions are project-specific and must be documented explicitly.
- If different layers use different naming languages or casing rules, record them in architecture docs or ADRs.
- Do not assume the current project uses database naming in the same language as source code.
- Docs, specs, and ADRs should remain clear and consistent in one chosen documentation language.

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

- **Integration branch**: configurable per project, defaulting to `develop` unless documented otherwise
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

### Code Quality Compliance (Mandatory)

After writing or modifying source code, run the quality checks required by the active stack and resolve **all** relevant warnings and errors before considering the change complete.

- Prefer Biome where it is supported and already established for the project.
- If the stack requires another tool for a surface, use the documented tool for that surface.
- Do not mark a task as `Done` with outstanding quality failures on changed code.
- In Nx workspaces, prefer a workspace-scoped validation command such as `pnpm nx run-many -t check` when that target exists.

### Test Pass Gate (Mandatory)

A task MUST NOT be marked `Done` unless **all** related tests pass — both unit and integration:

- **Tests MUST be actually executed** — never assume tests pass without running them. Observing the test command output is mandatory.
- Run the relevant test suite before transitioning a task to `Done`.
- If any test fails, the task stays `In Progress` until all failures are resolved.
- Never mark a Definition of Done checkbox as complete if the corresponding tests have not been executed and verified as passing.
- When a task includes test scenarios in its spec, every listed scenario must have a passing test.
- **Environment failures are NOT a reason to skip tests.** If the required environment is not running or is misconfigured and tests cannot execute, **stop immediately** and alert the user to fix the environment. Do not proceed with marking the task as Done.
- Never silently skip, ignore, or assume test results — every test run must produce visible, verified output.

### Test Evidence Gate (Mandatory)

Every task spec contains a **Section 10 — Test Evidence** table. This section serves as auditable proof that tests were executed and passed.

- **Before marking a task as `Done`**, populate the Test Evidence table with the exact command(s) run and their pass/fail/skip output summary.
- The Definition of Done checkbox for Test Evidence is **blocking** — if the section is empty or shows failures, the task cannot transition to `Done`.
- Never pre-fill this section during planning (Steps 1–3). It is populated only during Step 4.
- This is non-negotiable: build and lint passing alone do NOT satisfy the test gate.

## Documentation Lookup (Context7)

This workspace has the **Context7 MCP** available (`mcp_context7_resolve-library-id`, `mcp_context7_get-library-docs`).

- **Always use Context7** to fetch up-to-date documentation before writing or planning code that depends on any library, framework, or tool in the active stack.
- Do NOT rely solely on training data for API signatures, configuration options, decorator usage, or migration guides — verify against current docs via Context7.
- During **planning** (Step 2) and **task implementation** (Step 4), consult Context7 for every library-specific decision: correct decorator syntax, module configuration, query/mutation patterns, test utilities, CLI flags, etc.
- When a library version is ambiguous or a breaking change is suspected, resolve it through Context7 before proceeding.

## MCP Usage Policy In SDD Flow

When a relevant MCP exists for the task, prefer it over ad hoc manual exploration or generic alternatives.

### GitKraken MCP

- Use GitKraken MCP when the task depends on Git workflow context, issue context, launchpad prioritization, review flow, or structured commit composition.
- In SDD, use it when starting work from an issue, reviewing PR-oriented work, inspecting assigned work, or organizing commits when the user asks to commit.
- Prefer GitKraken issue and review tools over manual Git-only exploration when the task is really about issue, PR, or work context rather than local file edits.

### GitHub MCP

- Use GitHub MCP when the task requires remote repository information or remote repository actions.
- This includes repository search, issue lookup, PR lookup, comments, releases, remote file operations, or other GitHub-hosted context needed to plan or execute work.
- In SDD, use it whenever feature or task context comes from GitHub issues or PRs, or when progress needs to be reflected back into GitHub artifacts.

### Playwright MCP

- Use Playwright MCP whenever the task involves browser behavior, UI validation, end-to-end verification, or reproduction of a web flow.
- Do not wait for an explicit "use Playwright" request if browser automation is the right validation tool.
- In SDD Step 4 and Step 5, prefer Playwright MCP for validating frontend flows, API docs UIs, authenticated browser journeys, and regressions that are best checked in a real browser.

### Context7 MCP

- Context7 remains mandatory for library, framework, and tool documentation lookups as described above.

### Practical Rule

- Step 1 to Step 3: use GitHub, GitKraken, and Context7 whenever the feature, plan, or task depends on remote issue context, review context, or current library documentation.
- Step 4: use Context7 for implementation decisions, GitHub and GitKraken for issue or PR context when needed, and Playwright for browser-based validation.
- Step 5: use GitKraken or GitHub when retrospective, review, PR, or issue closure work depends on those systems.
