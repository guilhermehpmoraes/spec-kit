# ADR-006: Single Version Dependency Policy

## Status

Accepted

## Context

The monorepo uses pnpm workspaces with Nx for task orchestration. Initially, individual projects (`apps/admin/backend`, `packages/database`) maintained their own `dependencies` and `peerDependencies` in local `package.json` files.

This caused:

- **Nested `node_modules`** inside sub-projects where pnpm resolved version mismatches between root and local declarations.
- **Version drift** — the same package (e.g., `@nestjs/common`) was declared at `^11.0.0` in one project and `^11.1.19` at the root, leading to potential runtime conflicts.
- **Maintenance overhead** — every new dependency had to be added in the correct local `package.json` and kept in sync across projects.

Nx supports both "independently maintained dependencies" and "single version policy" strategies. For an integrated monorepo where all applications share the same stack and deploy from the same CI pipeline, single version policy is the recommended approach.

## Decision

Adopt a **single version policy** for all third-party dependencies:

1. **All third-party dependencies are declared exclusively in the root `package.json`** — both `dependencies` and `devDependencies`.
2. **Sub-project `package.json` files contain only metadata** — `name`, `version`, `private`, `main`/`types` (for packages), and Nx configuration. No `dependencies`, `devDependencies`, or `peerDependencies` for external packages.
3. **Inter-workspace references** (e.g., `@satie/database`) use pnpm workspace linking via `pnpm-workspace.yaml` packages globs — no explicit `workspace:*` entries needed for private, non-publishable packages.
4. **Nx analyzes imports in source code** to determine the project dependency graph, independent of `package.json` declarations.

## Consequences

### Positive

- Single source of truth for all dependency versions — no drift or conflicts.
- No nested `node_modules` in sub-projects.
- Simpler onboarding: `pnpm install` at the root is all that's needed.
- Easier upgrades: update a version once, all projects use it.
- Smaller lockfile diffs on updates.

### Negative

- All teams must coordinate on dependency upgrades (acceptable for current team size).
- Adding a new dependency requires editing the root `package.json` (minor friction).

### Neutral

- Nx task caching and dependency graph analysis are not affected — Nx traces actual imports, not `package.json` declarations.

## References

- [Nx Dependency Management Strategies](https://nx.dev/docs/concepts/decisions/dependency-management)
