# ADR-008: Workspace Project Naming Conventions

## Status

Accepted

## Context

Nx-based monorepos need unambiguous project names as the workspace grows. Short generic names such as `backend` or `frontend` do not scale.

This caused:

- **Ambiguous names** — `backend` and `frontend` give no context about which application they belong to. As the monorepo grows, these names would collide.
- **Inconsistent scoping** — mixing scoped and unscoped package names makes it unclear whether scoping is intentional.
- **Mixed configuration locations** — some projects stored Nx configuration (e.g., `implicitDependencies`) in `package.json` under an `nx` key, while others used `project.json`. This split made it harder to find and audit Nx configuration.

## Decision

This ADR applies only to Nx-based workspaces that use project names in `project.json` and package metadata in `package.json`.

Typical ambiguity problems include names like `backend` or `frontend` that do not identify which application they belong to.

### Package Names (`package.json` -> `name`)

All projects in the monorepo should use a configurable package scope prefix. Application projects include the application name to avoid collisions:

| Project Type | Pattern | Example |
|--------------|---------|---------|
| Application backend | `@<scope>/<app>-backend` | `@acme/billing-backend` |
| Application frontend | `@<scope>/<app>-frontend` | `@acme/billing-frontend` |
| Application backend tests | `@<scope>/<app>-backend-tests` | `@acme/billing-backend-tests` |
| Application frontend tests | `@<scope>/<app>-frontend-tests` | `@acme/billing-frontend-tests` |
| Shared package | `@<scope>/<package>` | `@acme/database` |
| Shared package tests | `@<scope>/<package>-tests` | `@acme/database-tests` |

### Nx Project Names (`project.json` → `name`)

Nx project names drop the package scope but include the application prefix:

| Project Type | Pattern | Example |
|--------------|---------|---------|
| Application backend | `<app>-backend` | `billing-backend` |
| Application frontend | `<app>-frontend` | `billing-frontend` |
| Application backend tests | `<app>-backend-tests` | `billing-backend-tests` |
| Application frontend tests | `<app>-frontend-tests` | `billing-frontend-tests` |
| Shared package | `<package>` | `database` |
| Shared package tests | `<package>-tests` | `database-tests` |

### Nx Configuration Location

All Nx project configuration MUST live in `project.json` files — never in the `nx` key inside `package.json`. This gives a single, predictable location for:

- `name`
- `projectType`
- `sourceRoot`
- `tags`
- `targets`
- `implicitDependencies`

Sub-project `package.json` files contain only package metadata: `name`, `version`, `private`, and entry points (`main`, `types`) for library packages.

For non-Nx projects, document naming conventions in the relevant architecture doc instead of applying this ADR directly.

### Cross-References

When referencing other projects in Nx configuration (e.g., `implicitDependencies`, `dependsOn`, `buildTarget`), use the Nx project name (without the package scope):

```json
{
    "implicitDependencies": ["billing-backend"],
    "targets": {
        "e2e": {
            "dependsOn": ["billing-backend:build", "billing-backend:serve"]
        }
    }
}
```

## Consequences

### Positive

- Project names are unambiguous — every name identifies both the application and the role.
- Adding a new application follows a predictable pattern: `billing-backend`, `billing-frontend`, `@acme/billing-backend`, etc.
- All Nx configuration lives in one place (`project.json`), making it easy to audit and search.
- Consistent package scoping across all `package.json` files.

### Negative

- Slightly longer names in Nx commands (e.g., `pnpm nx run billing-backend:serve` instead of `pnpm nx run backend:serve`).
- Existing CI scripts, documentation, or developer habits referencing old names need updating.

### Neutral

- The naming convention adds no runtime cost — names affect only tooling and developer experience.
- pnpm workspace resolution is unaffected since all packages remain `private: true`.
