# ADR-003: Biome Quality Tooling Baseline

## Status

Accepted

## Context

The project needs a single, fast, and consistent baseline for formatting and linting before creating the first domain projects.

Using multiple tools for style and linting increases maintenance cost, slows feedback loops, and creates avoidable divergence across apps and packages.

The team standard is to prioritize performance and deterministic formatting.

## Decision

We adopt Biome as the default code quality tool for the monorepo.

### Scope

- Biome is the standard formatter and linter for source code in this workspace.
- ESLint is not part of the baseline for new projects.
- The root `biome.json` is the source of truth for formatting and linting rules.

### Operational Baseline

- Root scripts expose the standard workflow:
    - `lint`
    - `lint:fix`
    - `format`
    - `format:check`
- Nx integration uses target defaults for `lint` with cache enabled and workspace-level inputs, including `biome.json`, so future project lint targets stay consistent.

## Consequences

- Faster local and CI feedback for lint/format tasks.
- Reduced tooling complexity by avoiding parallel formatter/linter stacks.
- Consistent formatting and import organization from day zero.
- Future projects must expose lint targets compatible with this baseline.
- Any proposal to reintroduce ESLint or split tooling must be documented in a new ADR.
