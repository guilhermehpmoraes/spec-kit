# ADR-003: Quality Gates and Tooling Baseline

## Status

Accepted

## Context

The original kit referenced quality tooling expectations without a dedicated ADR. That creates broken links and leaves each repository without a clear place to document linting, formatting, type checking, test execution, and validation command ownership.

## Decision

Every project adopting this kit must define a documented quality baseline during bootstrap.

At minimum, the project must specify:

- formatter and/or linter choices
- static analysis or type-checking expectations when applicable
- unit, integration, and end-to-end validation strategy when applicable
- canonical commands or tasks for validation
- whether warnings are allowed or treated as failures

The exact tools are project-specific. The kit does not require Biome, ESLint, Checkstyle, SpotBugs, Vitest, Jest, Maven, Gradle, Nx, or any other tool by default.

## Consequences

- Validation rules become explicit instead of being implied by sample stack choices.
- Prompts and implementation flows can reference project-defined commands instead of guessing.
- New repositories must invest a little time during bootstrap to define their real quality gates.