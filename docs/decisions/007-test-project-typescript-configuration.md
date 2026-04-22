# ADR-007: Monorepo TypeScript Configuration Strategy

## Status

Accepted (supersedes previous version scoped to test projects only)

## Context

This Nx monorepo contains multiple project types — backend applications (NestJS), frontend applications (Vite + React), shared libraries (NestJS packages), and separated test projects — each with different TypeScript requirements.

Configuring TypeScript consistently across these project types presents several challenges:

1. **Module system split** — Backend projects (NestJS) run on Node.js without a bundler and require CommonJS. Frontend projects use Vite which expects ESNext modules. The base config must pick a sensible default while allowing projects to override.
2. **Project references and incremental builds** — Nx uses TypeScript project references (`composite`, `declaration`, `references`) for incremental compilation. Source projects must opt into composite mode, while the root and umbrella configs use `files: []` to delegate to references.
3. **Decorator metadata** — NestJS and TypeORM rely on `experimentalDecorators` and `emitDecoratorMetadata` for dependency injection and entity mapping. Without these flags, decorator-based patterns fail silently at runtime.
4. **Test isolation** — Test projects live as siblings (`<project>-tests/`) and must not leak test framework types (`@types/jest`, `vitest`) into production code. Each test project scopes its own `types` array.
5. **Module resolution modes** — `"bundler"` resolution (TS 5.x) is correct for Vite and the base config, but CommonJS projects must use `"node"` resolution to match Node.js runtime behavior.
6. **Library vs. application conventions** — Nx conventions distinguish libraries (`tsconfig.lib.json`) from applications (`tsconfig.app.json`). Libraries must emit declaration files so consuming projects get type information.

## Decision

### 1. Base Config (`tsconfig.base.json`)

The root base config provides shared defaults for all projects:

```jsonc
{
    "compileOnSave": false,
    "compilerOptions": {
        "rootDir": ".",
        "sourceMap": true,
        "declaration": false,
        "moduleResolution": "bundler",
        "emitDecoratorMetadata": true,
        "experimentalDecorators": true,
        "importHelpers": true,
        "target": "ES2022",
        "module": "ESNext",
        "lib": ["ES2022", "dom"],
        "skipLibCheck": true,
        "skipDefaultLibCheck": true,
        "baseUrl": ".",
        "paths": {
            "@satie/database": ["packages/database/src/index.ts"]
        }
    },
    "exclude": ["node_modules"]
}
```

Key rationale:

| Field | Value | Why |
|---|---|---|
| `compileOnSave` | `false` | Nx controls compilation, not the editor. Prevents accidental out-of-place artifacts. |
| `rootDir` | `"."` | Anchors to monorepo root so TypeScript preserves folder structure correctly in output. Without it, TS infers incorrectly in monorepo setups. |
| `sourceMap` | `true` | Generates `.js.map` files so stack traces point to TypeScript source, not compiled JS. Essential for debugging. |
| `declaration` | `false` | Default off; only library projects override to `true` in their own config. |
| `module` | `"ESNext"` | Preserves native `import`/`export`, letting bundlers decide packaging. Backend and library projects override to `"CommonJS"`. |
| `moduleResolution` | `"bundler"` | TS 5.x mode that mimics modern bundler behavior (Vite, esbuild). Allows extensionless imports and `exports` field in `package.json`. Backend overrides to `"node"`. |
| `target` | `"ES2022"` | Node.js 18+ and modern browsers support ES2022 natively. No unnecessary downcompilation. |
| `emitDecoratorMetadata` | `true` | Required by NestJS — reflection metadata enables constructor-based DI. In base for consistency. |
| `experimentalDecorators` | `true` | Required for legacy decorator syntax (`@Controller`, `@Module`, etc.) used by NestJS. |
| `importHelpers` | `true` | Deduplicates TS helper functions via `tslib` instead of inlining them per file. Reduces bundle size. |
| `lib` | `["ES2022", "dom"]` | ES2022 APIs (`Array.at()`, `Object.hasOwn()`, etc.) + DOM for frontend. Including `dom` is harmless for backend (affects type-checking only, not output). |
| `skipLibCheck` | `true` | Skips type-checking of third-party `.d.ts` files. Practically mandatory — many npm packages have inconsistent types. |
| `skipDefaultLibCheck` | `true` | Complements `skipLibCheck` for TS standard library definitions (`lib.dom.d.ts`, `lib.es2022.d.ts`). |
| `baseUrl` | `"."` | Required for `paths` aliases to resolve correctly. |
| `paths` | `@satie/database → ...` | Maps workspace package aliases to source files. Enables cross-package imports with editor navigation and autocomplete. |

### 2. Root Solution Config (`tsconfig.json`)

```jsonc
{
    "extends": "./tsconfig.base.json",
    "files": [],
    "references": [
        { "path": "./apps/admin/backend" },
        { "path": "./apps/admin/backend-tests" },
        { "path": "./apps/admin/frontend" },
        { "path": "./apps/admin/frontend-tests" },
        { "path": "./packages/database" },
        { "path": "./packages/database-tests" }
    ]
}
```

- **`files: []`** — compiles nothing directly; exists solely to aggregate references.
- **`references`** — declares all subprojects for `tsc --build` incremental compilation and Nx dependency graph resolution.

### 3. Source Project Configs

Each source project uses a **two-file pattern**:

#### Umbrella config (`tsconfig.json`)

```jsonc
{
    "extends": "<path>/tsconfig.base.json",
    "files": [],
    "include": [],
    "references": [{ "path": "./tsconfig.app.json" }]  // or ./tsconfig.lib.json
}
```

Compiles nothing; groups references. Nx and editors use this as the project entry point.

#### Backend Application (`apps/admin/backend/tsconfig.app.json`)

```jsonc
{
    "extends": "./tsconfig.json",
    "compilerOptions": {
        "outDir": "../../../dist/apps/admin/backend",
        "module": "CommonJS",
        "moduleResolution": "node",
        "target": "ES2022",
        "emitDecoratorMetadata": true,
        "experimentalDecorators": true,
        "declaration": true,
        "composite": true
    },
    "include": ["src/**/*.ts"],
    "references": [
        { "path": "../../../packages/database/tsconfig.lib.json" }
    ]
}
```

- **`module: "CommonJS"` + `moduleResolution: "node"`** — NestJS runs on Node.js without a bundler. CommonJS requires the classic Node.js resolution algorithm. Using `"bundler"` with CommonJS causes inconsistencies.
- **`composite: true` + `declaration: true`** — mandatory for project references and incremental builds. `composite` enforces `declaration` and requires `outDir`.
- **`references`** — declares dependency on shared libraries (`database`) so `tsc --build` compiles them first.
- **Decorator flags repeated explicitly** — ensures they're active in the compilation unit even though inherited from base. Good practice for NestJS projects.

#### Frontend Application (`apps/admin/frontend/tsconfig.app.json`)

```jsonc
{
    "extends": "./tsconfig.json",
    "compilerOptions": {
        "outDir": "../../../dist/apps/admin/frontend",
        "module": "ESNext",
        "moduleResolution": "bundler",
        "target": "ES2022",
        "jsx": "react-jsx",
        "lib": ["ES2022", "DOM", "DOM.Iterable"],
        "types": ["vite/client"],
        "composite": true,
        "declaration": true,
        "allowImportingTsExtensions": false,
        "noEmit": false
    },
    "include": ["src/**/*.ts", "src/**/*.tsx"]
}
```

- **`module: "ESNext"` + `moduleResolution: "bundler"`** — Vite handles module bundling natively via esbuild. This is the official Vite + TS 5.x recommendation.
- **`jsx: "react-jsx"`** — React 17+ automatic JSX transform. No manual `import React` needed in every file.
- **`lib` adds `"DOM.Iterable"`** — required for `for...of` over `NodeList`, `HTMLCollection`, etc.
- **`types: ["vite/client"]`** — adds `import.meta.env`, `import.meta.hot`, and static asset import types (`import logo from './logo.svg'`).
- **`allowImportingTsExtensions: false`** — explicitly disallows `.ts` extensions in import paths. Unnecessary with Vite and can cause issues.
- **`noEmit: false`** — explicit because composite mode requires emit capability.

#### Shared Library (`packages/database/tsconfig.lib.json`)

```jsonc
{
    "extends": "./tsconfig.json",
    "compilerOptions": {
        "outDir": "../../dist/packages/database",
        "module": "CommonJS",
        "moduleResolution": "node",
        "target": "ES2022",
        "emitDecoratorMetadata": true,
        "experimentalDecorators": true,
        "declaration": true,
        "composite": true
    },
    "include": ["src/**/*.ts"]
}
```

- **`tsconfig.lib.json`** — Nx convention: libraries use `lib.json`, applications use `app.json`. Semantic distinction respected by Nx generators.
- **`declaration: true`** — mandatory for libraries. Consuming projects need `.d.ts` files for type checking across project boundaries.
- **`module: "CommonJS"`** — consumed by backend which runs without a bundler on Node.js.

### 4. Test Project Configs

Test projects extend the base config directly and scope test framework types to prevent leaking into production code.

#### Backend / Package Tests (Jest)

Pattern used by `apps/admin/backend-tests/` and `packages/database-tests/`:

```jsonc
{
    "extends": "<path>/tsconfig.base.json",
    "compilerOptions": {
        "outDir": "<path>/dist/<project>-tests",
        "module": "CommonJS",
        "moduleResolution": "node",
        "target": "ES2022",
        "emitDecoratorMetadata": true,
        "experimentalDecorators": true,
        "composite": true,
        "declaration": true,
        "types": ["jest", "node"]
    },
    "include": ["src/**/*.spec.ts", "src/**/*.test.ts"],
    "references": []
}
```

- **`types: ["jest", "node"]`** — scopes type visibility to Jest globals (`describe`, `it`, `expect`, `beforeEach`, etc.) and Node.js APIs (`process`, `Buffer`, `__dirname`). Keeps test types out of production code.
- **`include` restricted to test files only** — production code is resolved through `paths` aliases in the base config, not by glob-including sibling source.
- **`references: []`** — empty; Nx manages build ordering across projects.
- **Decorator flags** — required because tests import backend/library code that uses NestJS/TypeORM decorators.
- **`composite: true` + `declaration: true`** — participates in the `tsc --build` graph consistently with other projects.

#### Frontend Tests (Playwright e2e)

Pattern used by `apps/admin/frontend-tests/`:

```jsonc
{
    "extends": "<path>/tsconfig.base.json",
    "compilerOptions": {
        "outDir": "<path>/dist/apps/admin/frontend-tests",
        "module": "ESNext",
        "moduleResolution": "bundler",
        "target": "ES2022",
        "lib": ["ES2022", "DOM"],
        "sourceMap": false,
        "types": ["node"]
    },
    "include": ["playwright.config.ts", "src/**/*.spec.ts", "src/**/*.test.ts"]
}
```

- **No `composite` or `declaration`** — e2e tests don't participate in the incremental build graph.
- **`types: ["node"]`** — Playwright tests only need Node.js APIs. Playwright provides its own assertion API, not Jest/Vitest globals.
- **`sourceMap: false`** — e2e tests don't need source maps for debugging compiled output.
- **`lib` includes `"DOM"`** — Playwright tests may reference DOM types.
- **`include` has `playwright.config.ts`** — the Playwright configuration file needs type-checking alongside test files.
- **`module: "ESNext"` + `moduleResolution: "bundler"`** — consistent with the frontend stack it tests.

### 5. Configuration Matrix

| Config | `module` | `moduleResolution` | `composite` | `declaration` | `types` | Decorator flags |
|---|---|---|---|---|---|---|
| Base | `ESNext` | `bundler` | — | `false` | — | Yes |
| Backend app | `CommonJS` | `node` | `true` | `true` | — | Yes |
| Frontend app | `ESNext` | `bundler` | `true` | `true` | `["vite/client"]` | — |
| Database lib | `CommonJS` | `node` | `true` | `true` | — | Yes |
| Backend tests | `CommonJS` | `node` | `true` | `true` | `["jest", "node"]` | Yes |
| Database tests | `CommonJS` | `node` | `true` | `true` | `["jest", "node"]` | Yes |
| Frontend tests (e2e) | `ESNext` | `bundler` | — | — | `["node"]` | — |

### Rules

1. **Module system must match runtime.** `CommonJS` + `"node"` resolution for Node.js-direct projects (backend, libraries, their tests). `ESNext` + `"bundler"` resolution for Vite-based projects (frontend, frontend e2e tests).
2. **`composite: true` + `declaration: true`** for all projects that participate in `tsc --build` incremental compilation.
3. **Decorator flags (`experimentalDecorators`, `emitDecoratorMetadata`)** must be present in any config that compiles or type-checks NestJS/TypeORM code, including tests that import such code.
4. **Test projects scope `types` explicitly** to prevent test framework globals from leaking into production code.
5. **Test `include` patterns use only `*.spec.ts` and `*.test.ts`** — production code is resolved via path aliases and project references, never via glob-inclusion of sibling source.
6. **Never import test primitives from `node:test`.** Jest provides globals via `@types/jest`. Importing from `node:test` shadows Jest types and breaks `expect`/`jest`.
7. **Library projects use `tsconfig.lib.json`; application projects use `tsconfig.app.json`.** This follows Nx conventions; generators respect the distinction.
8. **`outDir` must point to `dist/` at the monorepo root**, preserving the project's relative path structure (e.g., `dist/apps/admin/backend`, `dist/packages/database`).

## Consequences

### Easier

- Consistent, documented pattern for all project types — straightforward to add new apps, libs, and test suites.
- IDE type-checking and navigation work correctly within and across project boundaries via `paths` aliases.
- Incremental builds via `tsc --build` + Nx reduce compilation time.
- Test framework types are isolated from production code.
- The two-file umbrella pattern keeps project entry points clean and separates compilation concerns.

### Harder

- Each project must maintain its own tsconfig with the correct overrides rather than relying entirely on inheritance from base.
- Adding a new shared library requires updating `paths` in `tsconfig.base.json` and adding `references` in consuming projects.
- The two-file pattern (umbrella + app/lib) per source project adds structural overhead but pays off in clarity.
- `module`/`moduleResolution` must be kept in sync — `CommonJS` + `"node"` or `ESNext` + `"bundler"`. Mismatches cause subtle resolution failures.
