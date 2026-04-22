# ADR-007: TypeScript Workspace Configuration Strategy

## Status

Accepted for TypeScript surfaces only

## Context

Some projects using this kit contain TypeScript across multiple surfaces with different runtime and build requirements.

Configuring TypeScript consistently across these project types presents several challenges:

1. **Module system split** — different surfaces may require different module targets and resolution strategies.
2. **Shared baseline vs local override** — a reusable base config helps consistency, but surface-specific overrides remain necessary.
3. **Framework flags** — decorators, JSX, DOM libs, and bundler resolution vary by stack.
4. **Test isolation** — test framework types must not leak into production code.
5. **Project references and incremental builds** — optional but valuable in larger TS workspaces.

## Decision

### 1. Base Config

Maintain a shared base config for TypeScript surfaces when the repository contains more than one TS project.

Recommended responsibilities of the base config:

- shared compiler defaults
- common path aliases when needed
- default target and module assumptions that the majority of surfaces can override safely
- shared strictness, source map, and library settings

### 2. Surface-Specific Overrides

Each TS surface should override what its runtime actually needs, including when applicable:

- module format
- module resolution
- JSX support
- DOM libs
- decorator flags
- output directory
- declaration emission
- test framework types

### 3. Project References

Use project references when the workspace benefits from incremental compilation and clear dependency ordering.

This is most valuable in larger TypeScript monorepos, but it is optional for smaller repos.

### 4. Test Isolation

Test projects or test configs must scope their `types` explicitly so Jest, Vitest, or other test globals do not leak into production code.

### 5. Runtime Match Rule

TypeScript configuration must match the real runtime and build system rather than relying on one universal default.

- Do not use bundler-oriented resolution for server-only CommonJS builds unless the toolchain explicitly supports it.
- Do not use Node-only assumptions in browser-facing or bundler-managed applications.

### 6. Framework-Specific Flags

Decorator flags, JSX transforms, DOM libraries, and bundler-specific settings should appear only where the framework requires them.

## Consequences

### Easier

- Consistent, documented TS configuration across multiple surfaces.
- Less accidental mismatch between runtime behavior and compiler assumptions.
- Test framework types stay isolated from production code.

### Harder

- Teams must maintain the discipline to scope flags and test types correctly.
- This ADR does not apply to non-TypeScript surfaces.
