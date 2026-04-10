# Feature Spec: [Feature Name]

- **Feature ID**: [###-feature-name]
- **Status**: Draft
- **Created**: [YYYY-MM-DD]
- **Owner**: [Name or Team]
- **Domain**: [Domain name from project context]
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

## 4.1 Engineering Quality Constraints

- **EQ-001 (KISS)**: The implementation MUST prefer the simplest design that satisfies all acceptance scenarios.
- **EQ-002 (Self-descriptive code)**: New code MUST be understandable through naming and structure without relying on comments.
- **EQ-003 (Comments policy)**: Comments MUST be added only for non-obvious intent, tradeoffs, or constraints.
- **EQ-004 (Testability)**: Boundaries and responsibilities MUST enable focused unit/integration tests.
- **EQ-005 (Patterns baseline)**: Design choices MUST follow current ADR baseline unless explicitly justified.

## 5. Dependencies and Risks

### Dependencies

- [Service, API, package, team dependency]

### Risks

- [Risk description] -> [Mitigation]

## 6. Success Criteria

- **SC-001**: [Measurable outcome]
- **SC-002**: [Measurable outcome]

## 7. Validation Mapping

Map each acceptance criterion to tests before implementation starts.

- **AC-001** -> [Test type: unit/integration/e2e] -> [Target module/flow]
- **AC-002** -> [Test type: unit/integration/e2e] -> [Target module/flow]

Also map engineering constraints to validation checks.

- **EQ-001/EQ-005** -> [Design review against ADRs]
- **EQ-002/EQ-003** -> [Code review checklist]
- **EQ-004** -> [Unit/integration test coverage target]

## 8. Optional Data Impact

Fill this section only if the feature changes domain data.

### Entities

- **[Entity Name]**: [Role in this feature]

### Data Notes

- [Migration, constraints, consistency, ownership notes]

## 9. Fixed Learnings Input (SDD Evolution)

Complete this section after finishing the feature.

### Keep

- [Practice that worked and should continue]

### Avoid

- [Practice that caused friction and should stop]

### Promote to Standard

- [Rule/template/checklist item to adopt from now on]

### Process Improvement for Next Cycle

- [One or two concrete improvements only]
