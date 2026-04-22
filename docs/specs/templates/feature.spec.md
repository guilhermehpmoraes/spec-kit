# Feature Spec: [Feature Name]

- **Feature ID**: [###-feature-name]
- **Status**: Draft | Approved | In Planning | Planned | In Implementation | Done | Archived
- **Created**: [YYYY-MM-DD]
- **Last Updated**: [YYYY-MM-DD]
- **Owner**: [Auto-filled from `git config user.name` of the person who creates this spec]
- **Domain**: [Domain, bounded context, subsystem, or N/A]
- **Application/System**: [Application, service, platform, or repository area]
- **Source Inputs**: [Request text, design doc, requirement links]
- **Feature Folder**: [docs/specs/features/<feature-id>/]
- **Feature File**: [docs/specs/features/<feature-id>/feature.spec.md]
- **Plan File**: [docs/specs/features/<feature-id>/plan.spec.md or TBD]
- **Task Folder**: [docs/specs/features/<feature-id>/tasks/]
- **Related ADRs**: [ADR-001, ADR-00X or N/A]

## 1. Context

Describe the problem in plain language and why this feature matters now.

## 2. Scope

### In Scope

- [What this feature will deliver]
- [Main capability 2]

### Out of Scope

- [What is explicitly not part of this feature]
- [Future work kept out of this iteration]

## 3. User Scenarios (Prioritized)

### Scenario 1 - [Title] (P1)

[Describe the user journey and expected value]

#### Acceptance Scenarios

1. **Given** [initial state], **When** [action], **Then** [expected result]
2. **Given** [initial state], **When** [action], **Then** [expected result]

### Scenario 2 - [Title] (P2)

[Describe the user journey and expected value]

#### Acceptance Scenarios

1. **Given** [initial state], **When** [action], **Then** [expected result]

### Scenario 3 - [Title] (P3)

[Describe the user journey and expected value]

#### Acceptance Scenarios

1. **Given** [initial state], **When** [action], **Then** [expected result]

## 4. Functional Requirements

- **FR-001**: System MUST [capability]
- **FR-002**: System MUST [capability]
- **FR-003**: Users MUST be able to [interaction]

## 5. Engineering Quality Constraints

- **EQ-001 (KISS)**: The implementation MUST prefer the simplest design that satisfies all acceptance scenarios.
- **EQ-002 (Self-descriptive code)**: New code MUST be understandable through naming and structure without relying on comments.
- **EQ-003 (Comments policy)**: Comments MUST be added only for non-obvious intent, tradeoffs, or constraints.
- **EQ-004 (Testability)**: Boundaries and responsibilities MUST enable focused unit and integration tests.
- **EQ-005 (Patterns baseline)**: Design choices MUST follow current ADR baseline unless explicitly justified.

## 6. Dependencies and Risks

### Dependencies

- [Service, API, package, team dependency]

### Risks

- [Risk description] -> [Mitigation]

## 7. Success Criteria

- **SC-001**: [Measurable outcome]
- **SC-002**: [Measurable outcome]

## 8. Validation Mapping

Map each acceptance criterion to tests before implementation starts.

- **AC-001** -> [Test type: unit/integration/e2e] -> [Target module/flow]
- **AC-002** -> [Test type: unit/integration/e2e] -> [Target module/flow]

Also map engineering constraints to validation checks.

- **EQ-001/EQ-005** -> [Design review against ADRs]
- **EQ-002/EQ-003** -> [Code review checklist]
- **EQ-004** -> [Unit/integration test coverage target]

## 9. Planning Inputs for Step 2

Use this section to make Step 2 deterministic.

### Affected Areas

- [Apps, services, packages, modules, database areas, infra, QA]

### Proposed Implementation Slices

- [Slice 1: core backend change]
- [Slice 2: frontend integration]
- [Slice 3: validation, tests, rollout]

### Contracts and Constraints

- [API endpoints, DTOs, UI states, schema constraints, external integrations]

### Technical Detail Baseline (Step 2 Input)

Capture enough technical detail so each generated task can be implemented without hidden context.

- **Data changes (if applicable)**: [tables, collections, topics, files, schemas, fields, types, nullability, defaults, constraints]
- **API or integration contracts (if applicable)**: [endpoint, method, event, request fields/types, response fields/types, error cases]
- **UI contracts (if applicable)**: [states, transitions, validations, empty/error states]
- **Migration or rollout notes (if applicable)**: [forward migration, rollback strategy, backfill, compatibility]

### Known Dependencies

- [Service, API, package, team dependency]

### Open Questions to Resolve in Planning

- [Question 1]
- [Question 2]

## 10. Workflow and Status Gates

### Status Transition Rules

- On creation in Step 1, set **Status** to `Draft` and save the feature spec file.
- After user approval, update **Status** to `Approved`.
- When Step 2 starts, update **Status** to `In Planning`.
- After feature plan approval and task file generation, update **Status** to `Planned`.
- When implementation of the first task starts, update **Status** to `In Implementation`.
- After all scoped tasks are done and validated, update **Status** to `Done`.

### Gate to move Step 1 -> Step 2

- [ ] Scope and requirements are approved.
- [ ] Acceptance scenarios are testable.
- [ ] Planning inputs are complete enough to derive tasks.

### Gate to move Step 2 -> Step 3

- [ ] Feature plan is approved.
- [ ] Task files are generated under `docs/specs/features/<feature-id>/tasks/`.
- [ ] At least one task has status `Ready`.

## 11. Optional Data Impact

Fill this section only if the feature changes domain data.

### Entities

- **[Entity Name]**: [Role in this feature]

### Data Notes

- [Migration, constraints, consistency, ownership notes]

## 12. Fixed Learnings Input (SDD Evolution)

Complete this section after finishing the feature.

### Keep

- [Practice that worked and should continue]

### Avoid

- [Practice that caused friction and should stop]

### Promote to Standard

- [Rule/template/checklist item to adopt from now on]

### Process Improvement for Next Cycle

- [One or two concrete improvements only]
