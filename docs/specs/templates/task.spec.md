# Task Spec: [Feature Name]

- **Feature ID**: [###-feature-name]
- **Date**: [YYYY-MM-DD]
- **Status**: Draft
- **Feature Spec**: [docs/specs/...]
- **Plan Spec**: [docs/specs/...]
- **Related ADRs**: [ADR-00X or N/A]

## 1. Purpose

This document breaks an approved plan into executable tasks with clear scope for the team.

Rules for this team:

- Use only issue types: **Task** and **Bug**.
- Do not use **Story**.
- Each task must be easy to understand and contain the minimum necessary detail for future implementation.

## 2. Workflow Gates (Mandatory)

1. Break feature into tasks in this spec.
2. Stop and wait for team/product validation.

Do not skip gate 2.

## 3. Task Writing Rules

- One objective per task.
- Use direct language and implementation intent.
- Write for handoff: another engineer must be able to implement the task without extra meetings.
- Include affected scope (Backend, Frontend, Infra, Data, QA).
- Include technical notes only when they reduce ambiguity.
- Prefer explicit behavior over generic wording (what should happen, when, and for whom).
- Include edge cases, error paths, and non-happy paths when relevant.
- Define expected inputs/outputs for APIs, events, or UI states when applicable.
- Use concrete examples (payloads, field names, route names, validation rules) when they improve clarity.
- Optional code snippets are allowed when they reduce ambiguity. Keep them short and illustrative, not full implementations.
- Include dependencies when a task is blocked by another one.
- Keep task size realistic for sprint planning.

Gitflow sizing rule (mandatory):

- One task should map to one branch whenever possible.
- Avoid very small tasks that create excessive branch overhead.
- Avoid oversized tasks that become long-lived feature branches.
- Target task size that can be implemented, reviewed, and merged in a short cycle.
- If a task cannot be delivered safely in one branch cycle, split it into smaller tasks.
- If multiple tasks are too small and always change the same files, consider merging them into one task.

## 4. Task Structure (Template)

Use this format for each task:

### [TXXX] [Task|Bug] [Title]

- **Type**: Task | Bug
- **Summary**: [Short Jira title]
- **Context**: [Why this task exists]
- **Scope**: Backend | Frontend | Backend+Frontend | Infra | Data | QA
- **Description**: [What must be implemented, clear and direct]
- **Implementation Details**: [Key implementation expectations, contracts, data flow, and constraints]
- **Acceptance Criteria**:
    - [AC1]
    - [AC2]
- **Test Scenarios**:
    - [Scenario 1: happy path]
    - [Scenario 2: validation/error path]
    - [Scenario 3: edge case]
- **Technical Notes**: [Important implementation notes only]
- **Example Snippet (Optional)**: [Small pseudo/code snippet only when it clarifies expected implementation]
- **Dependencies**: [TYYY, external dependency, or N/A]
- **Estimate**: [ex: 0.5 day, 1 day, 2 days]
- **Priority**: Highest | High | Medium | Low
- **Labels**: [Backend, Frontend, Integration, etc.]
- **Jira Custom Fields**:
    - **Tipo**: Feature | Improvement | Technical Debt | Bugfix
    - **Tamanho**: [ex: 1 Dia, 2 Dias, 4 Dias]

## 5. Task Quality Checklist

Before sending for validation, confirm all items:

- [ ] Title is understandable without extra explanation.
- [ ] Description is sufficient for implementation by another team member.
- [ ] Implementation details are specific enough to avoid interpretation gaps.
- [ ] Acceptance criteria are testable.
- [ ] Test scenarios cover happy path, failure path, and edge cases (when applicable).
- [ ] Type is Task or Bug only.
- [ ] Priority and estimate are defined.
- [ ] Labels and custom fields are filled.
- [ ] Dependencies are explicit.
- [ ] Task size is balanced for Gitflow (1 task = 1 healthy branch cycle).

## 5.1 Definition of Ready (Dev Handoff)

A task is ready for development only if:

- Scope is explicit and bounded.
- Required contracts are defined (API, DTO, events, UI states, or schema changes).
- Acceptance criteria can be validated in code review and tests.
- Open questions are resolved or captured as explicit assumptions.
- Dependencies are either completed or clearly sequenced.

## 5.2 Snippet Guidance (Optional)

Use snippets only to remove ambiguity. Good candidates:

- API request/response examples.
- DTO/interface skeletons.
- Validation rule examples.
- UI state mapping examples.

Avoid:

- Large code blocks.
- Full implementations.
- Snippets that conflict with existing ADRs or project conventions.

## 6. Branching Guidance (Gitflow)

Use this guidance during task breakdown:

- Prefer one branch per task.
- Branch name pattern suggestion: `feature/<jira-key>-<short-title>` or `bugfix/<jira-key>-<short-title>`.
- A task is too large when it likely needs many days of isolated development, high conflict risk, or multiple unrelated changes.
- A task is too small when it does not create independently reviewable value and only increases branch management overhead.
- Split by technical boundary when needed: API contract, backend service, frontend module, integration, tests.

## 7. Jira Mapping (Based on Team Pattern)

Map each task to Jira fields in this minimum set:

- **Issue Type**: Task or Bug
- **Summary**
- **Description**
- **Priority**
- **Labels**
- **Assignee** (optional at import time)
- **Sprint** (optional)
- **Parent** (if using parent/epic structure)
- **Custom field (Tipo)**
- **Custom field (Tamanho)**

Recommended optional fields when available:

- Start date
- Components
- Fix versions

## 8. Final Output of This Spec

This spec is complete when:

- Task breakdown is done.
- Validation gate is passed.
