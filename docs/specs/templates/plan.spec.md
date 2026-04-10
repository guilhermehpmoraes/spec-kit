# Implementation Plan: [Feature Name]

- **Feature ID**: [###-feature-name]
- **Date**: [YYYY-MM-DD]
- **Status**: Draft
- **Feature Spec**: [docs/specs/features/your-feature.spec.md]
- **Related ADRs**: [ADR-001, ADR-00X or N/A]

## 1. Planning Goal

Describe the implementation approach for this feature and define the work slices that will be converted into tasks.

## 2. Summary

- **Problem to solve**: [Short statement from feature spec]
- **Primary outcome**: [What will be true when feature is done]
- **Implementation approach**: [High-level technical direction]

## 3. Technical Context

### Stack Baseline (Project Profile)

- **Repository model**: [Nx monorepo | single repo | other]
- **Backend**: [chosen stack or N/A]
- **Frontend**: [chosen stack or N/A]
- **Database/Data stores**: [chosen stack or N/A]
- **Tests**: [unit/integration/e2e stack]
- **Package/build manager**: [pnpm | npm | Maven | Gradle | etc.]

### Feature-Specific Context

- **Touched apps/packages**: [apps/<domain>/backend, apps/<domain>/frontend, packages/<name>]
- **New dependencies**: [Library or N/A]
- **Data impact**: [Schema/read-model/write-model impact or N/A]
- **Constraints**: [Latency, security, compliance, rollout, etc.]

## 4. SDD Gates

Must pass before task breakdown:

- [ ] Feature spec approved and linked
- [ ] Acceptance scenarios are testable
- [ ] Functional requirements mapped to implementation areas
- [ ] ADRs linked for architectural decisions (or explicitly N/A)
- [ ] Risks and dependencies reviewed

## 4.1 Engineering Gates

Must pass before implementation starts:

- [ ] KISS-first approach documented (why simpler option is enough)
- [ ] Pattern choice aligned with ADR baseline (or deviation justified)
- [ ] Comment strategy documented (where comments are truly needed)
- [ ] Testability confirmed for each implementation slice
- [ ] Complexity hotspots identified with mitigation

## 5. Scope-to-Work Mapping

Map scope into implementation slices that can become task groups.

### Slice A - [Name]

- **Goal**: [What this slice delivers]
- **Requirements covered**: [FR-001, FR-00X]
- **Acceptance covered**: [AC references]
- **Expected outputs**: [Code/tests/docs]

### Slice B - [Name]

- **Goal**: [What this slice delivers]
- **Requirements covered**: [FR-001, FR-00X]
- **Acceptance covered**: [AC references]
- **Expected outputs**: [Code/tests/docs]

## 6. Repository Impact

List concrete paths expected to change.

```text
apps/
  <domain>/
    backend/
    frontend/
packages/
  <shared-lib>/
docs/
  specs/
  decisions/ (if needed)
```

### Planned Changes by Area

- **Backend**: [Modules, handlers, services, repositories, tests]
- **Frontend**: [Routes, pages, components, queries, tests]
- **Shared packages**: [Types, utilities, UI components]
- **Database**: [Migrations, seeds, constraints]

## 7. Validation Strategy

- **Unit tests**: [What to cover]
- **Integration tests**: [What flows/contracts to validate]
- **E2E tests**: [Critical journeys]
- **Non-functional checks**: [Performance, accessibility, security checks if applicable]

## 8. Rollout and Safety

- **Feature flags**: [Needed or N/A]
- **Backward compatibility**: [Impact and mitigation]
- **Monitoring/observability**: [Logs/metrics/alerts]
- **Rollback plan**: [How to safely disable/revert]

## 9. Task Planning Output

Define how tasks must be produced from this plan.

- Group tasks by slice (A, B, C...)
- Keep each task independently verifiable
- Map each task to requirement and test target
- Identify dependencies and parallelizable work
- Mark MVP-critical tasks first

## 10. Complexity Tracking (Optional)

Fill only when complexity is intentionally increased and must be justified.

| Decision            | Why Needed     | Simpler Alternative Rejected Because |
| ------------------- | -------------- | ------------------------------------ |
| [e.g., New package] | [Current need] | [Why simpler option is insufficient] |

Use this table whenever the plan deviates from KISS-first direction.

## 11. Post-Implementation Feedback

Complete after delivery to sharpen future plans.

### What worked in planning

- [Planning choice that improved delivery]

### What caused friction

- [Planning gap that created rework/delay]

### Planning standard updates

- [Rule/template improvement to carry forward]
