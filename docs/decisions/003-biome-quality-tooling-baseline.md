# ADR-003: Quality Tooling Baseline by Stack Profile

## Status

Accepted

## Context

The project needs a consistent baseline for formatting and linting before creating the first domain projects.

Using many overlapping tools for style and linting increases maintenance cost, slows feedback loops, and creates avoidable divergence across repositories and modules.

The standard is to prioritize deterministic formatting, fast feedback, and minimal configuration overlap.

## Decision

We adopt one primary formatter/linter baseline per active language/runtime profile, documented in project specs and build scripts.

For TypeScript/JavaScript profiles, Biome is the recommended default.

### Scope

- Each project must explicitly define its quality tooling baseline in docs/specs.
- Tooling should avoid redundant overlap (for example, multiple formatters for the same files unless justified).
- Configuration files selected by the stack become the source of truth for formatting/linting rules.

### Operational Baseline

- Root scripts should expose the standard workflow when applicable:
    - `lint`
    - `lint:fix`
    - `format`
    - `format:check`
- Build/task orchestration should keep lint/format targets consistent and cache-friendly.

## Consequences

- Faster local and CI feedback for lint/format tasks.
- Reduced tooling complexity by avoiding unnecessary parallel formatter/linter stacks.
- Consistent formatting from day zero with stack-specific flexibility.
- Future projects must document and maintain compatible lint/format targets.
- Any proposal that increases tooling overlap must be documented in a new ADR.
