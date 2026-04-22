# ADR-009: SDD Branching Strategy

## Status

Accepted

## Context

The project previously used Git Flow for branch management. Git Flow supports a single level of feature branches from `develop`, with no concept of sub-branches for individual tasks within a feature.

Our SDD (Spec Driven Development) workflow introduces a **two-level branching hierarchy**: each feature has its own long-lived branch, and each task within that feature gets a short-lived branch off the feature branch. Git Flow's CLI cannot manage this nesting — attempting to force it causes friction with `git flow feature finish` and related commands.

Additionally, our branch lifecycle is tightly coupled to SDD steps (spec → plan → tasks → implement → finish), making a custom skill more natural than a generic tool.

## Decision

Replace Git Flow with a **custom SDD-aligned branching strategy** managed by the `sdd-branch` Copilot skill. The strategy uses plain `git` commands (no `git-flow` CLI dependency).

### Branch Hierarchy

```
main                                          ← production-ready
  └── develop (currently: sandbox)            ← integration branch
        └── feature/<feature-id>              ← created at Step 1 (feature spec)
              └── task/<task-id>              ← created at Step 4 (task implementation)
```

### Branch Naming Conventions

| Branch Type | Pattern | Example |
|-------------|---------|---------|
| Integration | `develop` (or `sandbox` during bootstrap) | `develop` |
| Feature | `feature/<feature-id>` | `feature/001-admin-identity-domain` |
| Task | `task/<task-id>` | `task/T001-setup-typeorm` |

- `<feature-id>` matches the feature folder name under `docs/specs/features/`.
- `<task-id>` matches the task spec filename prefix (e.g., `T001-setup-typeorm` from `T001-setup-typeorm.task.spec.md`).

### Integration Branch Configuration

The default integration branch is `develop`. During the bootstrap phase, `sandbox` is used as the integration branch. The active integration branch is configured in the skill and can be changed without modifying the workflow.

### Lifecycle Tied to SDD Steps

| SDD Step | Branch Action |
|----------|---------------|
| **Step 1 — Feature Spec** | Create `feature/<feature-id>` from integration branch, publish it to the remote, and checkout feature branch. |
| **Step 2 — Feature Plan** | Work on feature branch (docs only). |
| **Step 3 — Task Breakdown** | Work on feature branch (docs only). |
| **Step 4 — Task Implementation** | Create `task/<task-id>` from feature branch. Implement on task branch. After commit, merge task branch into feature branch (fast-forward or no-ff). Delete task branch. |
| **Step 5 — Feature Finish** | Merge feature branch into integration branch. Delete feature branch. |
| **Release** | Validate integration branch (build, lint, test, e2e). If all pass, merge into `main`. |

### Merge Strategy

- **Task → Feature**: Merge with `--no-ff` to preserve task boundary in history, then delete task branch.
- **Feature → Integration**: Merge with `--no-ff` to preserve feature boundary in history, then delete feature branch.
- **Integration → Main (Release)**: Merge with `--no-ff` after full validation (typecheck, lint, build, test, e2e). Integration branch is NOT deleted.
- No rebasing across branches — merges keep a clear audit trail aligned with specs.
- Feature branches are published to the remote immediately after creation so the branch lifecycle is visible and recoverable across sessions.

### Conflict Resolution

- Before creating a task branch, pull latest from the feature branch.
- Before merging feature into integration, pull latest integration and merge into feature first (resolve conflicts on the feature branch, not on integration).

## Consequences

### Positive

- **SDD-aligned** — Branch lifecycle mirrors the spec workflow exactly, reducing cognitive overhead.
- **Clear history** — Each task and feature is visible as a merge commit with references to spec files.
- **No external tooling** — Uses plain `git` commands; no `git-flow` CLI dependency.
- **Flexible integration branch** — Easy to switch from `sandbox` to `develop` when ready.

### Negative

- **Manual management** — Relies on the `sdd-branch` skill for automation; without it, developers must remember the conventions.
- **More branches** — More active branches at any given time compared to flat Git Flow.

### Neutral

- Releases to `main` are manual (triggered via `/release`) and gated by a full validation pipeline. There is no automated CI-driven release flow at this stage.
