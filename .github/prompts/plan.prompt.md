---
description: "Create a feature plan (Step 2) for an approved feature spec, resolving all technical questions to enable self-sufficient task generation"
name: "plan"
argument-hint: "Feature ID (e.g. 001-feature-name)"
agent: "agent"
---
You are responsible for executing **only Step 2 — Feature Plan** of this repository's SDD workflow.

This prompt receives a `feature-id` and produces a complete, implementation-ready feature plan at `docs/specs/features/<feature-id>/plan.spec.md`.

## Input

{{input}}

## Gate check

1. Locate the feature spec at `docs/specs/features/<feature-id>/feature.spec.md` and read it in full.
2. **If the spec does not exist** → stop and inform the user.
3. **If the spec Status is NOT `Approved`** → stop and inform the user. Only features with status `Approved` can be planned. Suggest using `/feature` to create or refine the spec first.
4. **If a `plan.spec.md` already exists** in the feature folder:
   - If its status is `Draft` → enter **Refine mode** (update existing plan with new input).
   - If its status is `Approved` or beyond → stop and inform the user the plan is already approved. Suggest moving to Step 3 (Task Breakdown).

## Required reading

Before starting any planning work, read and internalize:

1. `docs/project.spec.md` — project context and goals
2. `.github/copilot-instructions.md` — naming conventions, stack, SDD rules
3. `docs/specs/templates/plan.spec.md` — the plan template to fill
4. `docs/specs/templates/task.spec.md` — understand what the plan must feed into (tasks must be self-sufficient)
5. All ADRs referenced by the feature spec in `docs/decisions/`
6. The feature's parent domain/area spec if it exists (check `docs/specs/domains/`)
7. The surface-specific architecture spec at `docs/specs/apps/<surface>/architecture.md` when it exists
8. Existing codebase structure relevant to the feature (explore the actual repository layout as needed)

## Planning process

### Phase 1 — Verify assumptions and resolve open questions

Before writing the plan, you MUST perform technical verification.

**Context7 documentation lookup**: For every library, framework, or tool referenced in the feature spec or relevant to the plan, use the Context7 MCP (`mcp_context7_resolve-library-id` → `mcp_context7_get-library-docs`) to fetch current documentation. Verify API signatures, configuration options, integration patterns, and migration guidance against the latest docs — do not rely on training data alone.

**GitHub and GitKraken remote context lookup**: If the feature, plan input, or existing docs reference GitHub issues, pull requests, comments, or review decisions, use the GitHub or GitKraken MCP to fetch the canonical remote context before locking the plan.

**Playwright browser verification**: If the plan affects frontend behavior, routes, forms, e2e flows, or UX regressions, use the Playwright MCP to inspect the current browser behavior and encode that verified behavior into the plan instead of relying on assumptions.

**Pencil design verification**: If the plan affects frontend or UI behavior with meaningful visual or UX impact, use the Pencil MCP to create, inspect, or update the relevant prototype during planning. Do not approve the plan until the Pencil reference and approval state are recorded.

1. **Resolve all "Open Questions to Resolve in Planning"** from the feature spec (section 9). For each question, either:
   - Find the answer by exploring the codebase, ADRs, docs, or Context7 library documentation → include the resolution in the plan.
   - Cannot determine the answer → ask the user.

2. **Verify feature assumptions against reality**. The feature spec may assume things that don't exist yet. For each assumption:
   - Check if referenced modules, packages, tables, endpoints, patterns, or dependencies actually exist in the codebase.
   - If something does NOT exist, flag it explicitly — it becomes part of the plan scope or a prerequisite task.

3. **Identify technical gaps** not covered by the feature spec. Ask the user about:
   - Ambiguous technical choices (e.g., which library for X, sync vs async approach)
   - Missing contracts (API shape, DB schema details, UI state transitions)
   - Integration points with existing code
   - Performance, security, or observability requirements not yet specified
   - Any boundary conditions or edge cases that need clarification

### Phase 2 — Ask questions

Gather all questions from Phase 1 and ask them in a **single, organized batch**. Group questions by category:

- **Open questions from feature spec** — questions explicitly listed in the spec
- **Technical design questions** — DB schema details, API contracts, library choices, patterns
- **Verification gaps** — things the feature assumed exist but don't, or need confirmation
- **Documentation impact** — changes needed to ADRs, architecture docs, domain specs
- **Cross-cutting concerns** — security, performance, observability, rollout strategy

Do NOT proceed to Phase 3 until the user has answered all critical questions.

### Phase 3 — Write the plan

After all questions are resolved:

1. Fill every section of `docs/specs/templates/plan.spec.md` with concrete, specific details.
2. The plan must be **detailed enough to generate self-sufficient task specs** — any developer picking up a task derived from this plan must be able to implement it without hidden context, undocumented assumptions, or additional discovery.

#### Mandatory detail levels

- **Data design**: Exact storage object names, field names, types, nullability, defaults, PK/FK or references, constraints, indexes, and migration strategy. Use the project's naming conventions (ADR-005).
- **API contracts**: Exact endpoints, HTTP methods, request/response shapes with field names and types, validation rules, error codes and messages.
- **UI contracts** (if applicable): Routes, component hierarchy, states (loading/empty/error/success), field validations, interaction behavior, responsive behavior, and Pencil prototype references when visual impact exists.
- **File paths**: Exact files to create or modify — no vague references like "backend module". List actual paths.
- **Dependencies**: Exact npm packages with versions (or "latest"), shared workspace packages, external services.

#### Documentation impact mapping

The plan MUST include an explicit mapping of documentation changes or additions needed:

- **New or updated ADRs** (`docs/decisions/`): If the plan introduces a new pattern, library, or architectural decision not covered by existing ADRs.
- **Domain specs** (`docs/specs/domains/`): If a new domain is being created or an existing domain spec needs updates.
- **Surface architecture** (`docs/specs/apps/<surface>/architecture.md`): If the feature changes a documented application, service, package, or deployable surface architecture.
- **Other docs**: README updates, new guides, configuration docs, etc.

Include these documentation tasks in the scope-to-execution mapping and task generation matrix.

### Phase 4 — Update feature spec status

After the plan is written:
1. Update the feature spec `Status` from `Approved` to `In Planning`.
2. Update the feature spec `Plan File` field to point to the new plan file.
3. Update `Last Updated` date on the feature spec.

## Refine mode

If a `plan.spec.md` already exists with status `Draft`:

1. Read the existing plan in full.
2. Identify what needs updating based on user input.
3. Ask questions only about the new or changed areas.
4. Update the plan file in place — do not overwrite unrelated sections.
5. Keep status as `Draft`.

## Exit criteria

- Plan file saved at `docs/specs/features/<feature-id>/plan.spec.md` with status `Draft`.
- Feature spec status updated to `In Planning`.
- Frontend or UI plans with meaningful visual impact include a recorded Pencil prototype reference and approval state.
- Show in the chat:
  - The `feature-id` and plan file path
  - Summary of questions asked and their resolutions
  - Summary of verified vs unverified assumptions
  - List of documentation changes included in the plan
  - Number of planned slices and tasks in the generation matrix
- **Ask the user**: "Should I approve this plan (change status to `Approved`) so it can move to Step 3 (Task Breakdown), or does it still need refinement?"

## Approval transition

If the user confirms approval:
1. Change the plan `Status` from `Draft` to `Approved`.
2. Update the feature spec `Status` from `In Planning` to `Planned`.
3. Update `Last Updated` dates on both files.
4. Confirm in the chat that the plan is `Approved` and ready for Step 3 (Task Breakdown).

## Critical rules

- **Do NOT write production code** in this step. This is documentation-only.
- **Do NOT generate task files** in this step. That is Step 3.
- **Do NOT skip questions** — the plan feeds task generation. Incomplete plans produce incomplete tasks. Incomplete tasks block implementation.
- **Every assumption must be explicit** — if something is assumed but not verified, document it as an assumption and flag it.
- **Self-sufficiency is non-negotiable** — every slice and future task must be implementable by any developer without needing to "just know" things.
