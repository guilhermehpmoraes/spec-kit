# ADR-008: Project Naming Conventions

## Status

Accepted

## Context

The monorepo contains multiple Nx projects: applications (backend, frontend) and shared packages. Initially, projects used short, unscoped names (`backend`, `frontend`, `database-tests`) in both `package.json` and `project.json`. Only `@satie/database` followed the scoped naming pattern.

This caused:

- **Ambiguous names** — `backend` and `frontend` give no context about which application they belong to. As the monorepo grows to host more applications (Admin, Satie, and future ones), these names would collide.
- **Inconsistent scoping** — `@satie/database` used the org scope while everything else did not, making it unclear whether scoping was intentional.
- **Mixed configuration locations** — some projects stored Nx configuration (e.g., `implicitDependencies`) in `package.json` under an `nx` key, while others used `project.json`. This split made it harder to find and audit Nx configuration.

## Decision

### Package Names (`package.json` → `name`)

All projects in the monorepo use the `@satie/` scope prefix. Application projects include the application name to avoid collisions:

| Project Type | Pattern | Example |
|--------------|---------|---------|
| Application backend | `@satie/<app>-backend` | `@satie/admin-backend` |
| Application frontend | `@satie/<app>-frontend` | `@satie/admin-frontend` |
| Application backend tests | `@satie/<app>-backend-tests` | `@satie/admin-backend-tests` |
| Application frontend tests | `@satie/<app>-frontend-tests` | `@satie/admin-frontend-tests` |
| Shared package | `@satie/<package>` | `@satie/database` |
| Shared package tests | `@satie/<package>-tests` | `@satie/database-tests` |

### Nx Project Names (`project.json` → `name`)

Nx project names drop the `@satie/` scope (Nx does not require it) but include the application prefix:

| Project Type | Pattern | Example |
|--------------|---------|---------|
| Application backend | `<app>-backend` | `admin-backend` |
| Application frontend | `<app>-frontend` | `admin-frontend` |
| Application backend tests | `<app>-backend-tests` | `admin-backend-tests` |
| Application frontend tests | `<app>-frontend-tests` | `admin-frontend-tests` |
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

### Cross-References

When referencing other projects in Nx configuration (e.g., `implicitDependencies`, `dependsOn`, `buildTarget`), use the Nx project name (without the `@satie/` scope):

```json
{
    "implicitDependencies": ["admin-backend"],
    "targets": {
        "e2e": {
            "dependsOn": ["admin-backend:build", "admin-backend:serve"]
        }
    }
}
```

## Consequences

### Positive

- Project names are unambiguous — every name identifies both the application and the role.
- Adding a new application (e.g., Satie) follows a predictable pattern: `satie-backend`, `satie-frontend`, `@satie/satie-backend`, etc.
- All Nx configuration lives in one place (`project.json`), making it easy to audit and search.
- Consistent `@satie/` scoping across all `package.json` files.

### Negative

- Slightly longer names in Nx commands (e.g., `pnpm nx run admin-backend:serve` instead of `pnpm nx run backend:serve`).
- Existing CI scripts, documentation, or developer habits referencing old names need updating.

### Neutral

- The naming convention adds no runtime cost — names affect only tooling and developer experience.
- pnpm workspace resolution is unaffected since all packages remain `private: true`.
