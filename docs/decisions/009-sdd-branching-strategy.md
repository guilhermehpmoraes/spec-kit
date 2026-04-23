# ADR-009: SDD Branching Strategy

## Status

Accepted

## Context

Spec Driven Development benefits from a branching model that mirrors the lifecycle of feature specs and task implementation. At the same time, different teams may use different integration branches or release habits.

## Decision

The kit adopts an **SDD-aligned branching model** with project-configurable branch names.

### Baseline Hierarchy

```text
main or trunk
  └── integration branch
        └── feature/<feature-id>
              └── task/<task-id>
```

### Baseline Rules

- Feature branches are created when a new feature spec is started.
- Task branches are created when an approved task enters implementation.
- Task branches merge back into the parent feature branch.
- Feature branches merge back into the configured integration branch.
- Releases to the production branch are gated by project-defined validation.

### Configuration

The consuming project must define during bootstrap:

- the integration branch name
- whether release happens to `main`, `master`, `trunk`, or another branch
- required validation before merge or release
- whether task branches are local-only or published remotely

## Consequences

- The SDD workflow has a clear branch lifecycle.
- The kit no longer assumes `sandbox` or `develop` for every repository.
- Branch skills and prompts must read project-specific configuration instead of hardcoding it.