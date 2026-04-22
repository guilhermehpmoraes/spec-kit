# Feature Plan: [Feature Name]

- **Feature ID**: [###-feature-name]
- **Plan ID**: [PLAN-###-feature-name]
- **Status**: Draft | Approved | Ready for Implementation | Completed
- **Date**: [YYYY-MM-DD]
- **Owner**: [Auto-filled from `git config user.name` of the person who creates this spec]
- **Feature Folder**: [docs/specs/features/<feature-id>/]
- **Feature Spec**: [docs/specs/features/<feature-id>/feature.spec.md]
- **Plan File**: [docs/specs/features/<feature-id>/plan.spec.md]
- **Task Output Folder**: [docs/specs/features/<feature-id>/tasks/]
- **Related ADRs**: [ADR-001, ADR-00X or N/A]

## 1. Planning Goal

Describe the implementation approach for the approved feature with enough technical detail to enable task breakdown in the next step (Step 3).

## 2. Summary

- **Feature objective**: [What the feature must accomplish]
- **Why now**: [Context from feature spec]
- **Expected outcome**: [What will be true when this feature is delivered]
- **Implementation direction**: [High-level technical direction]

## 3. Technical Context

### Stack Baseline (Satie)

- **Monorepo**: Nx + pnpm
- **Backend**: NestJS
- **Frontend**: Vite + React + TanStack Router + TanStack Query + Tailwind CSS
- **Database**: PostgreSQL
- **Tests**: Jest + Supertest, Vitest + React Testing Library, Playwright

### Feature-Specific Context

- **Touched apps/packages**: [apps/<domain>/backend, apps/<domain>/frontend, packages/<name>]
- **New dependencies**: [Library or N/A]
- **Data impact**: [Schema/read-model/write-model impact or N/A]
- **Constraints**: [Latency, security, compliance, rollout, etc.]

### Constraints and Assumptions

- **Inputs**: [API payloads, events, forms, existing state]
- **Outputs**: [API responses, UI states, writes, side effects]
- **Boundary conditions**: [Invalid inputs, empty states, race conditions, retries]

## 3.1 Technical Design Baseline (Required)

This section must be detailed enough to generate implementation-ready task specs.

### Database Design (fill when data impact exists)

#### Tables and Actions

| Schema | Table | Action (Create/Alter/Drop) | Purpose |
| ------ | ----- | -------------------------- | ------- |
| [public] | [nome_tabela] | [Create] | [Why this table changes] |

#### Columns Specification

| Tabela | Campo | Tipo SQL | Nullable | Default | PK | FK Ref | Unique | Index | Notes |
| ------ | ----- | -------- | -------- | ------- | -- | ------ | ------ | ----- | ----- |
| [usuarios] | [id] | [uuid] | [No] | [gen_random_uuid()] | [Yes] | [N/A] | [Yes] | [PK] | [Primary key] |

#### Constraints and Indexes

- [Constraint name, type, target columns, behavior]
- [Index name, target columns, index type, reason]

#### Migration Strategy

- **Forward migration**: [what is created/altered and in which order]
- **Rollback strategy**: [how to safely revert]
- **Backfill strategy**: [if needed]
- **Compatibility window**: [old/new code coexistence notes]

### API and Integration Contracts (fill when applicable)

- **Endpoint/Consumer**: [method + path or event]
- **Request contract**: [field name, type, validation rules]
- **Response contract**: [field name, type, mapping]
- **Error contract**: [codes/messages/conditions]

### UI Contracts (fill when applicable)

- **Screen/Route**: [path or component context]
- **States**: [loading, empty, error, success]
- **Validation rules**: [field-level rules and messages]
- **Interaction notes**: [important transitions or edge behavior]

## 4. Planning Gates (Step 2)

Must pass before the plan is approved and task breakdown (Step 3) can begin:

- [ ] Feature spec status is `Approved`.
- [ ] Feature scope is explicit and bounded.
- [ ] Required contracts are defined or referenced.
- [ ] Acceptance criteria are testable.
- [ ] Dependencies are completed or properly sequenced.
- [ ] Risks and edge cases are reviewed.
- [ ] Technical design baseline is complete for all impacted layers.

### Engineering Gates

- [ ] KISS-first approach is documented for major slices.
- [ ] Pattern choice aligns with ADR baseline or deviation is justified.
- [ ] Comment strategy is documented where comments are truly needed.
- [ ] Testability is confirmed for slice boundaries.
- [ ] Complexity hotspots are identified with mitigation.

## 5. Scope-to-Execution Mapping

Map feature scope into implementation slices that will become task files.

### Slice 1 - [Name]

- **Goal**: [What this slice delivers]
- **Files to touch**: [Exact paths]
- **Requirements covered**: [FR-001, FR-002]
- **Acceptance covered**: [AC references]
- **Expected outputs**: [Code/tests/docs]

### Slice 2 - [Name]

- **Goal**: [What this slice delivers]
- **Files to touch**: [Exact paths]
- **Requirements covered**: [FR-001, FR-003]
- **Acceptance covered**: [AC references]
- **Expected outputs**: [Code/tests/docs]

## 6. Task Generation Matrix

Use this matrix to derive one task file per row.

| Task ID | Title | Type | Plan Slice | Parallelizable | Dependencies | Requirement Refs | Technical Scope | Output File |
| ------- | ----- | ---- | ---------- | -------------- | ------------ | ---------------- | --------------- | ----------- |
| [T001]  | [Title] | Task | [Slice 1] | [No] | [N/A or T000] | [FR-001, AC-001] | [DB migration + API contract] | [docs/specs/features/<feature-id>/tasks/T001-<short-title>.task.spec.md] |
| [T002]  | [Title] | Task | [Slice 2] | [Yes] | [T001] | [FR-002, AC-002] | [Frontend states + integration] | [docs/specs/features/<feature-id>/tasks/T002-<short-title>.task.spec.md] |

## 7. Task File Generation Rules

- Create one file per task under `docs/specs/features/<feature-id>/tasks/`.
- Use naming pattern: `TXXX-<short-title>.task.spec.md`.
- Use `docs/specs/templates/task.spec.md` for every task file.
- Each task must reference the same feature spec and this feature plan.
- Start each task with status `Draft`; set to `Ready` after task-level approval.
- If a task cannot be delivered safely in one branch cycle, split it before approval.
- For data-impact tasks, include explicit table, field, SQL type, nullability, default, PK/FK, constraints, indexes, and migration notes.
- For API/UI-impact tasks, include explicit request/response/UI state contracts.

## 8. Repository Impact

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
- **QA**: [Unit tests, integration tests, E2E tests, manual checks]

## 9. Validation Strategy

- **Unit tests**: [What to cover]
- **Integration tests**: [What flows/contracts to validate]
- **E2E tests**: [Critical journeys]
- **Non-functional checks**: [Performance, accessibility, security checks if applicable]

## 10. Rollout and Safety

- **Feature flags**: [Needed or N/A]
- **Backward compatibility**: [Impact and mitigation]
- **Monitoring/observability**: [Logs/metrics/alerts]
- **Rollback plan**: [How to safely disable/revert]

## 11. Step 2 Completion Checklist

- [ ] Plan is approved.
- [ ] Technical design baseline is complete for all impacted layers.
- [ ] Scope-to-execution mapping covers all feature requirements.
- [ ] Task generation matrix is defined and consistent with scope.
- [ ] Dependencies between planned slices are explicit.
- [ ] Feature spec status is updated to `Planned`.
- [ ] Plan is ready to hand off to Step 3 (Task Breakdown).

## 12. Post-Implementation Feedback

Complete after feature delivery to sharpen future planning.

### What worked in planning

- [Planning choice that improved delivery]

### What caused friction

- [Planning gap that created rework or delay]

### Planning standard updates

- [Rule/template improvement to carry forward]
