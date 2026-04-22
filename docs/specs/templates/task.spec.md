# Task Spec: [Task Title]

- **Feature ID**: [###-feature-name]
- **Task ID**: [TXXX]
- **Status**: Draft | Awaiting Dependency | Ready | In Progress | Blocked | Done
- **Type**: Task | Bug
- **Parallelizable**: Yes | No
- **Parallelization Notes**: [Why this task is safe to run in parallel, or why it is not]
- **Date**: [YYYY-MM-DD]
- **Owner**: [Auto-filled from `git config user.name` of the person who starts implementation]
- **Feature Folder**: [docs/specs/features/<feature-id>/]
- **Feature Spec**: [docs/specs/features/<feature-id>/feature.spec.md]
- **Feature Plan**: [docs/specs/features/<feature-id>/plan.spec.md]
- **Task File**: [docs/specs/features/<feature-id>/tasks/TXXX-<short-title>.task.spec.md]
- **Related ADRs**: [ADR-00X or N/A]
- **Dependencies**: [TYYY, external dependency, or N/A]

## 1. Purpose

Describe exactly what this task must deliver and how it contributes to the feature outcome.

Rules for this team:

- Use only issue types: **Task** and **Bug**.
- Do not use **Story**.
- Keep this file complete enough that another developer can implement it without hidden context.

## 2. Scope

### In Scope

- [Concrete deliverable 1]
- [Concrete deliverable 2]

### Out of Scope

- [Explicitly excluded item 1]
- [Explicitly excluded item 2]

## 3. Context from Feature Plan

- **Plan slice**: [Setup | Foundational | Slice A | Slice B | QA | Polish]
- **Requirement refs**: [FR-001, FR-002, AC-001, AC-002]
- **Affected Paths**: [Exact file paths or folders]
- **Design References**: [`.pen` file, node IDs, screen/component names, or N/A]
- **Why this task exists**: [Dependency, scenario, or risk reduction]

## 4. Technical Specification (Required)

This section must be detailed enough for another developer to implement without additional discovery.

If a subsection does not apply, explicitly write `N/A` and explain why.

### 4.1 Backend and API Contracts (when applicable)

- **Use case flow**: [request -> service -> persistence -> response]
- **Endpoint/Event**: [method + route or event name]
- **Request contract**:

| Field | Type | Required | Validation | Notes |
| ----- | ---- | -------- | ---------- | ----- |
| [nome] | [string] | [Yes] | [min/max/format] | [details] |

- **Response contract**:

| Field | Type | Nullable | Source | Notes |
| ----- | ---- | -------- | ------ | ----- |
| [id] | [string] | [No] | [database/API] | [details] |

- **Error contract**: [status codes, error shape, and conditions]

### 4.2 Database Specification (mandatory when data impact exists)

Database object names must follow project convention in Portuguese.

#### Tables and Actions

| Schema | Tabela | Acao (Create/Alter/Drop) | Motivo |
| ------ | ------ | ------------------------ | ------ |
| [public] | [alunos] | [Create] | [new enrollment flow] |

#### Fields Specification

| Tabela | Campo | Tipo SQL | Nullable | Default | PK | FK (ref) | Unique | Index | Notes |
| ------ | ----- | -------- | -------- | ------- | -- | -------- | ------ | ----- | ----- |
| [alunos] | [id] | [uuid] | [No] | [gen_random_uuid()] | [Yes] | [N/A] | [Yes] | [PK] | [primary key] |

#### Constraints and Indexes

- **Constraints**: [name, type, columns, behavior]
- **Indexes**: [name, columns, type, reason]

#### Migration and Data Safety

- **Migration files**: [expected paths/names]
- **Forward migration steps**: [ordered SQL changes]
- **Rollback steps**: [safe revert strategy]
- **Backfill strategy**: [if needed]
- **Validation queries**: [queries to verify data integrity after migration]

### 4.3 Frontend and UX Contracts (when applicable)

- **Screens/routes affected**: [path/component]
- **Pencil references**: [`.pen` file path, screen node IDs, component catalog references]
- **Shared component reuse/creation**: [existing shared component to reuse or new shared component required]
- **Theme/token impact**: [global CSS variables, semantic tokens, spacing, typography]
- **Responsive behavior**: [mobile-first rules, breakpoints, layout changes]
- **UI states**: [loading, empty, error, success]
- **Field validations and messages**: [rules and UX feedback]
- **Interaction and edge behavior**: [important transitions and limits]

### 4.4 Cross-Cutting Constraints

- **Security/authorization**: [rules]
- **Performance expectations**: [latency/volume expectations]
- **Observability**: [logs/metrics/traces required]
- **Compatibility constraints**: [versioning, rollout window]

## 5. Implementation Steps

1. [Step 1 with concrete change]
2. [Step 2 with concrete change]
3. [Step 3 with validation or integration]

## 6. Acceptance Criteria

- [AC1: observable behavior]
- [AC2: observable behavior]
- [AC3: failure or edge behavior]

## 7. Test Scenarios

- [Scenario 1: happy path]
- [Scenario 2: validation or error path]
- [Scenario 3: edge case path]

## 8. Definition of Ready (to start Step 4)

A task is ready for implementation only if:

- [ ] Status is `Ready`.
- [ ] Scope is explicit and bounded.
- [ ] Required contracts are defined (API, DTO, event, UI state, schema).
- [ ] Technical specification is detailed enough for independent implementation.
- [ ] For data-impact tasks, table/field/type/constraint/index/migration details are fully documented.
- [ ] For UI-impact tasks, approved Pencil references are linked and implementation scope matches them.
- [ ] Acceptance criteria are testable.
- [ ] Open questions are resolved or captured as explicit assumptions.
- [ ] All dependency tasks are `Done` (if any dependencies exist).

## 9. Definition of Done

- [ ] Status moved to `Done`.
- [ ] All related tests (unit and integration) pass — no test failures allowed.
- [ ] **Section 10 (Test Evidence) is filled** with the exact command(s) executed and their output summary proving all tests pass.
- [ ] Biome reports zero warnings and zero errors on all changed files.
- [ ] Acceptance criteria are validated by tests or clear verification evidence.
- [ ] If UI is impacted, delivered screens/components match the approved Pencil artifact or the deviation is documented and re-approved in planning.

## 10. Test Evidence

This section is **mandatory** before marking the task as `Done`. Paste the exact test command(s) executed and a summary of their output. If tests were not executed, this section must remain empty and the task **cannot** transition to `Done`.

| # | Command | Result | Timestamp |
| - | ------- | ------ | --------- |
| 1 | `[exact command]` | [e.g., Tests: 12 passed, 0 failed] | [YYYY-MM-DD HH:MM] |

**Rules**:
- Every test suite relevant to the task must have a row in this table.
- "Result" must include pass/fail/skip counts copied from actual terminal output.
- If any test fails, the task stays `In Progress` — do not fill this section with failing results and mark Done.
- This section is never pre-filled during Step 3 (Task Breakdown) — it is populated only during Step 4 (Implementation).
- [ ] Edge cases listed in this task are covered.
- [ ] Links to changed files/PR/tests are registered.
- [ ] Feature spec and plan traceability remains intact.

## 10. Jira Mapping (Optional)

- **Issue Type**: Task or Bug
- **Summary**: [Short Jira title]
- **Description**: [Implementation objective]
- **Priority**: Highest | High | Medium | Low
- **Labels**: [Backend, Frontend, Integration, etc.]
- **Custom field (Tipo)**: Feature | Improvement | Technical Debt | Bugfix
- **Custom field (Tamanho)**: [ex: 1 Dia, 2 Dias, 4 Dias]

## 11. Status Log

Record status transitions to keep execution history visible.

| Date       | Status      | Notes |
| ---------- | ----------- | ----- |
| YYYY-MM-DD | Draft       | Task created from approved feature plan |
| YYYY-MM-DD | Awaiting Dependency | Task approved; waiting for [TYYY] to complete |
| YYYY-MM-DD | Ready       | All dependencies met; ready for implementation |
| YYYY-MM-DD | In Progress | Implementation started |
| YYYY-MM-DD | Done        | Implementation and validation completed |

## 12. Observations

Capture runtime observations during implementation — environment issues, library surprises, deviations from plan, workarounds applied, or any other notes that don't fit in other sections. This section feeds into the feature retrospective (Step 5).

- [Observation 1]
- [Observation 2]
