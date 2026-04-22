# ADR-003: Biome Quality Tooling Baseline

## Status

Accepted

## Context

The project needs a fast and consistent baseline for formatting and linting, but this kit must also support stacks where Biome is not the right tool for every language surface.

Using too many tools for the same language surface increases maintenance cost, slows feedback loops, and creates avoidable divergence across apps and packages.

The team preference is to prioritize performance and deterministic formatting, with Biome as the first choice where it fits.

## Decision

We adopt the following quality tooling strategy:

1. **Biome is the preferred default** for supported JavaScript, TypeScript, JSON, CSS, and related frontend surfaces.
2. **Projects may use additional or alternative tooling** for unsupported ecosystems such as Java, Kotlin, Python, Go, or framework-specific requirements.
3. **Each language surface must have an explicit quality tool** documented in repo-wide ADRs or the owning app or service architecture doc.
4. **A single surface should avoid redundant overlapping tools** unless there is a clear justification.

### Scope

- Biome is the standard formatter and linter for supported surfaces in this workspace.
- ESLint or other tools are allowed when the stack or ecosystem requires them.
- When Biome is active, the root `biome.json` is the source of truth for those surfaces.

### Operational Baseline

- Root scripts or workspace tasks should expose a standard workflow when applicable:
    - `lint`
    - `lint:fix`
    - `format`
    - `format:check`
- In Nx workspaces, target defaults should keep lint and format behavior consistent across projects.
- In non-Nx repositories, document the equivalent commands in project-level docs.

## Consequences

- Faster local and CI feedback for lint/format tasks.
- Reduced tooling complexity by avoiding parallel formatter/linter stacks on the same surface.
- Consistent formatting and import organization from day zero.
- Future projects must expose quality checks compatible with this baseline.
- Mixed-tooling projects remain valid as long as the tool choice is explicit per surface.
