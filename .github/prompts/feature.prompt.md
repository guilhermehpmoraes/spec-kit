---
description: "Create or refine a feature spec draft (Step 1) following the project's SDD workflow"
name: "feature"
argument-hint: "Describe a new feature OR provide the feature-id of an existing Draft to refine"
agent: "agent"
---
You are responsible for executing **only Step 1 — Feature Spec Draft** of this repository's SDD workflow.

This prompt supports two modes:
- **New feature**: create a fresh spec from a feature description.
- **Refine existing Draft**: update an existing spec that is in `Draft` status with new or revised information.

Use the input provided below by the user as the primary input.

## Mode detection

1. Check if the user input references an existing `feature-id` (e.g. `001-feature-name`) or an existing feature spec path.
2. If it does, locate the file at `docs/specs/features/<feature-id>/feature.spec.md` and read it.
   - If the spec **Status is `Draft`** → enter **Refine mode**.
   - If the spec **Status is NOT `Draft`** (e.g. `Approved`, `In Planning`, etc.) → **stop** and inform the user that this prompt only works for features in `Draft` status. Suggest using the appropriate SDD step for the current status.
3. If no existing feature is referenced → enter **New feature mode**.

---

## New feature mode

### Goal
Create a complete feature spec draft with status `Draft`, following the project's official template.

### Required steps
1. Read and follow the rules from:
   - `docs/project.spec.md`
   - `.github/copilot-instructions.md`
   - `docs/specs/templates/feature.spec.md`
   - Relevant ADRs in `docs/decisions/` (including ADR-004 for domain/area context)
   - If the feature request references a GitHub issue, PR, or review discussion, fetch that remote context with the GitHub or GitKraken MCP before drafting the spec.

2. Ask pertinent refinement questions **before finalizing the spec** to reduce ambiguity. Prioritize questions about:
   - Problem and business value
   - Target product/surface (application, service, package, library, etc.) and domain/area (if applicable)
   - Scope and out of scope
   - User scenarios and priority (P1/P2/P3)
   - Functional requirements
   - Technical constraints and dependencies
   - Measurable success criteria
   - Data, API, and UI impacts
   - Use Playwright MCP when the feature depends on current browser behavior, rendered states, or UX regressions that should be verified instead of inferred.

3. If information remains missing after questions, record explicit assumptions in the spec and flag gaps in "Open Questions to Resolve in Planning".

4. Generate a `feature-id` in the format `###-feature-name` (kebab-case), create the feature folder, and save the file at:
   - `docs/specs/features/<feature-id>/feature.spec.md`

5. Set status to `Draft` and fill in all applicable sections of the feature spec template — including the `Domain/Area` and `Product/Surface` fields.

6. Do not write production code in this step.

### Exit criteria
- Deliver the spec in the correct file path.
- Show in the chat:
  - The generated `feature-id`
  - Path of the created file
  - Short list of refinement questions asked
  - Summary of assumptions made
- **Ask the user**: "Should I approve this feature (change status to `Approved`) so it can move to the planning step, or does it still need refinement?"

---

## Refine mode

### Goal
Update an existing `Draft` feature spec with new information provided by the user, ask targeted questions about the new input, and keep the spec consistent.

### Required steps
1. Read the existing feature spec file in full.

2. Read and follow the rules from:
   - `docs/project.spec.md`
   - `.github/copilot-instructions.md`
   - `docs/specs/templates/feature.spec.md`
   - Relevant ADRs referenced in the spec
   - If the refinement is driven by a GitHub issue, PR, or review thread, refresh that remote context with the GitHub or GitKraken MCP before updating the draft.

3. Identify which sections of the spec are affected by the user's new input.

4. Ask targeted refinement questions **specific to the new information** before applying changes. Focus on:
   - Impacts on existing scope, scenarios, and functional requirements
   - New acceptance criteria or changes to existing ones
   - New dependencies, risks, or constraints introduced
   - Consistency with sections that are NOT being changed

5. Apply the changes to the spec file:
   - Update affected sections.
   - Update `Last Updated` date.
   - Keep status as `Draft`.
   - Do NOT remove or overwrite existing content that is unrelated to the new input unless the user explicitly requests it.

6. Do not write production code in this step.

### Exit criteria
- The spec file is updated in place.
- Show in the chat:
  - The `feature-id` and file path
  - Summary of what was changed
  - List of refinement questions asked about the new input
  - Any new assumptions or open questions added
- **Ask the user**: "Should I approve this feature (change status to `Approved`) so it can move to the planning step, or does it still need refinement?"

---

## Approval transition

If the user confirms approval:
1. Change the spec `Status` field from `Draft` to `Approved`.
2. Update `Last Updated` date.
3. Confirm in the chat that the feature is now `Approved` and ready for Step 2 (Feature Plan + Task Files).

---

## Feature input
{{input}}
