---
description: "Run the SDD Feature Finish retrospective (Step 5) — validate all specs are Done, ask targeted questions, and evolve the SDD process"
name: "finish"
argument-hint: "Feature ID (e.g. 001-admin-identity-domain)"
agent: "agent"
---
You are responsible for executing **Step 5 — Feature Finish** of this repository's SDD workflow.

This prompt receives a `feature-id`, validates that the entire feature is complete, runs the SDD Sharpening Loop retrospective, and applies approved process improvements.

## Input

{{input}}

## Gate check — All specs must be Done

1. Find the feature folder matching the feature number under `docs/specs/features/` (e.g. `docs/specs/features/001-*/`).
2. Read the **feature spec** at `docs/specs/features/<feature-id>/feature.spec.md`.
   - **If Status is NOT `Done`** → stop. List the current status and inform the user that all tasks must be completed first via `/implement` (Step 4). Do not proceed.
3. Read the **plan spec** at `docs/specs/features/<feature-id>/plan.spec.md`.
   - **If Status is NOT `Completed`** → stop. Inform the user the plan is not yet marked as completed.
4. Read **every task spec** under `docs/specs/features/<feature-id>/tasks/`.
   - **If ANY task Status is NOT `Done`** → stop. List all tasks with their current statuses and inform the user which tasks still need to be completed.
5. **Only proceed if**: feature spec is `Done`, plan spec is `Completed`, and ALL task specs are `Done`.

## Required reading

Before starting the retrospective, read and internalize:

1. The feature spec (`feature.spec.md`) — business context, scope, acceptance scenarios
2. The feature plan (`plan.spec.md`) — technical design, assumptions, constraints
3. All task specs under `tasks/` — implementation details, deviations, status logs
4. `.github/copilot-instructions.md` — current SDD rules and conventions
5. `docs/project.spec.md` — project context
6. All ADRs referenced by the feature in `docs/decisions/`
7. Existing spec templates in `docs/specs/templates/` (feature, plan, task)
8. The application architecture spec at `docs/specs/apps/<app>/architecture.md`
9. The domain spec if referenced (check `docs/specs/domains/`)
10. The project `README.md` at the repository root — developer-facing guide for setup, init, and operation

## Retrospective process

### Phase 1 — Feature delivery summary

Before asking questions, present a concise summary of the completed feature:

1. **Feature name and ID**
2. **Total tasks delivered** (count and list with IDs and titles)
3. **Timeline** (created date → completion date)
4. **Scope delivered** — brief summary of what was built
5. **Deviations detected** — scan all task spec status logs and implementation notes for any deviations from the original plan. List them explicitly.
6. **New ADRs created** during this feature (if any)
7. **New patterns introduced** — any architectural or code patterns that appeared for the first time

### Phase 2 — Targeted retrospective questions

Ask the user the following questions in a **single organized batch**. Group them clearly:

#### Process & Workflow
- What went smoothly during this feature's lifecycle (spec → plan → tasks → implementation)?
- Where did friction or rework happen? Were any steps unnecessarily slow or confusing?
- Did the 5-step delivery flow work well, or should any step be adjusted?

#### Specs & Planning Quality
- Were the feature spec and plan detailed enough to produce good tasks?
- Were task specs self-sufficient — could you implement from them without extra discovery?
- Were any specs unclear, incomplete, or missing critical information?
- Did any implementation deviate from the plan? Why?

#### Architecture & Technical Decisions
- Were the architectural decisions (ADRs) referenced by this feature still valid, or do any need updating?
- Did any new architectural patterns emerge that should be documented as ADRs?
- Are there technical debt items or improvement opportunities discovered during implementation?
- Should any domain spec (`docs/specs/domains/`) be created or updated based on this feature?

#### Templates & Standards
- Did the spec templates (feature, plan, task) capture everything needed, or are there gaps?
- Are there sections in the templates that were consistently N/A and could be simplified?
- Are there recurring details that should become default template content?

#### Patterns to Promote
- Are there implementation patterns from this feature worth standardizing across the codebase?
- Did any naming conventions, folder structures, or code patterns prove effective?
- Should any coding guidelines be added to `copilot-instructions.md`?

**Wait for the user to answer before proceeding to Phase 3.**

### Phase 3 — Analyze and propose improvements

Based on the user's answers:

1. **Categorize findings** into:
   - **What worked well** — practices to keep and reinforce
   - **What caused friction** — specific pain points with root causes
   - **Patterns to promote** — proven practices to standardize
   - **Improvements to apply** — concrete, actionable changes

2. **Propose specific changes** (1–3 max per cycle, keep it lightweight). Each proposal must include:
   - **What**: exact change description
   - **Where**: exact file path to modify (template, copilot-instructions, ADR, etc.)
   - **Why**: evidence from this feature's retrospective
   - **Impact**: how this improves the next feature cycle

3. **Present proposals to the user** and ask: "Which of these improvements should I apply now?"

### Phase 4 — Apply approved changes

After user approval:

1. **Apply each approved change** to the specified files:
   - Update spec templates in `docs/specs/templates/` if template changes were approved
   - Update `.github/copilot-instructions.md` if process/guideline changes were approved
   - Update prompt files in `.github/prompts/` if prompt workflow changes were approved
   - Create new ADR files in `docs/decisions/` if new architectural decisions were approved
   - Update domain specs in `docs/specs/domains/` if domain documentation changes were approved
   - Update architecture specs in `docs/specs/apps/<app>/` if architecture documentation changes were approved

2. **Update the project `README.md`** — Review the root `README.md` and update it to reflect any changes introduced by this feature that affect how developers set up, initialize, run, or operate the project. This includes but is not limited to:
   - New environment variables or configuration requirements
   - New services added to `docker-compose.yml`
   - New CLI commands, scripts, or Nx targets
   - Changed prerequisites or dependencies
   - Updated setup/init steps
   - New applications, packages, or domain modules added to the workspace
   - Any operational knowledge a developer would need to work with the changes introduced by this feature

   The README should serve as the single entry point for a developer joining the project — it must be clear on how to get the project running from scratch.

3. **Update the feature spec** status to `Archived` (post-retrospective final state).

4. **Record retrospective summary** — add a `## Retrospective` section at the bottom of the feature spec with:
   - Date of retrospective
   - Key findings (brief bullets)
   - Changes applied (with file paths)

## Exit criteria

- Feature spec, plan, and all tasks confirmed as `Done`/`Completed`.
- Delivery summary presented.
- Retrospective questions asked and answered.
- Improvement proposals presented and user-approved.
- Approved changes applied to the codebase.
- Project `README.md` reviewed and updated to reflect the feature's impact on setup/operation.
- Feature spec updated with retrospective section and status set to `Archived`.
- Show in chat:
  - Feature ID and name
  - Number of improvements applied
  - Files modified during retrospective
  - Brief summary of what changed in the SDD process

## Critical rules

- **Never skip the gate check** — all specs must be Done/Completed before running the retrospective.
- **Keep it lightweight** — propose 1–3 improvements max. Small evidence-based changes over large rewrites.
- **Do not change production code** — this step only modifies documentation, templates, instructions, and ADRs.
- **Ask, don't assume** — the retrospective is a conversation. Do not invent answers or skip user input.
- **Preserve what works** — only change what has clear evidence of needing improvement.
- **One cycle at a time** — improvements apply to the next feature, not retroactively to past features.
