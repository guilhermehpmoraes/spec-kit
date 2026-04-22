# Task Spec: Shared Database Package @satie/database

- **Feature ID**: 001-admin-identity-domain
- **Task ID**: T002
- **Status**: Done
- **Type**: Task
- **Parallelizable**: Yes
- **Parallelization Notes**: No dependency on any other task. Safe to run in parallel with T000 and T001. This is the first shared package in the monorepo.
- **Date**: 2026-04-15
- **Last Updated**: 2026-04-15
- **Owner**: TBD
- **Feature Folder**: docs/specs/features/001-admin-identity-domain/
- **Feature Spec**: docs/specs/features/001-admin-identity-domain/feature.spec.md
- **Feature Plan**: docs/specs/features/001-admin-identity-domain/plan.spec.md
- **Task File**: docs/specs/features/001-admin-identity-domain/tasks/T002-shared-database-package.task.spec.md
- **Related ADRs**: ADR-005
- **Dependencies**: N/A

## 1. Purpose

Create the `@satie/database` shared package in `packages/database/` containing `EntidadeBase` (audit + soft-delete fields), a TypeORM configuration helper, SnakeNamingStrategy re-export, and a Redis module (`RedisModule` + `RedisService`). This package is the foundation for all database and cache operations across applications.

## 2. Scope

### In Scope

- Create `packages/database/` with full Nx project configuration
- Implement `EntidadeBase` abstract entity class with all audit and soft-delete columns
- Implement `createTypeOrmConfig()` helper for consistent TypeORM DataSource configuration
- Re-export `SnakeNamingStrategy` from `typeorm-naming-strategies`
- Implement `RedisModule` (NestJS global dynamic module) and `RedisService` (wrapping `ioredis`)
- Add `@satie/database` path alias to `tsconfig.base.json`
- Install required dependencies: `typeorm`, `@nestjs/typeorm`, `pg`, `typeorm-naming-strategies`, `ioredis`, `@nestjs/common`, `@nestjs/config`
- Add devDependencies: `@types/ioredis`
- Barrel export (`src/index.ts`)

### Out of Scope

- Application-specific entities (handled in T003)
- Application-level TypeORM module registration (handled in T003)
- Redis usage for token revocation logic (handled in T005)

## 3. Context from Feature Plan

- **Plan slice**: Slice 2 — Shared Database Package (`@satie/database`)
- **Requirement refs**: FR-014, FR-015, FR-016, FR-017, FR-018
- **Affected Paths**:
  - `packages/database/` (entire new package)
  - `tsconfig.base.json` (add path alias)
- **Why this task exists**: All applications need a shared foundation for entity base classes, TypeORM configuration, snake_case naming, and Redis connectivity. This is the first shared package in the monorepo.

## 4. Technical Specification (Required)

### 4.1 Backend and API Contracts (when applicable)

N/A — this is a library package, not an API.

**Exported API surface:**

| Export | Type | Purpose |
|--------|------|---------|
| `EntidadeBase` | Abstract class | Base entity with audit + soft-delete columns |
| `createTypeOrmConfig(configService)` | Function | Creates TypeORM `DataSourceOptions` from `ConfigService` |
| `SnakeNamingStrategy` | Class (re-export) | Enforces snake_case for all DB names |
| `RedisModule` | NestJS Module | Global dynamic Redis module |
| `RedisService` | NestJS Injectable | Redis client wrapper |

### 4.2 Database Specification (mandatory when data impact exists)

No direct table creation — `EntidadeBase` defines the inherited columns that will appear in all entity tables:

| Campo | Tipo SQL | Nullable | Default | Notes |
| ----- | -------- | -------- | ------- | ----- |
| `criado_por` | VARCHAR(255) | No | — | Set by application on create |
| `modificado_por` | VARCHAR(255) | No | — | Set by application on create/update |
| `criado_as` | TIMESTAMP | No | `NOW()` | `@CreateDateColumn` |
| `modificado_as` | TIMESTAMP | No | `NOW()` | `@UpdateDateColumn` |
| `deletado_as` | TIMESTAMP | Yes | NULL | `@DeleteDateColumn` for soft delete |
| `deletado_por` | VARCHAR(255) | Yes | NULL | Set alongside soft delete |

### 4.3 Frontend and UX Contracts (when applicable)

N/A.

### 4.4 Cross-Cutting Constraints

- **Compatibility constraints**: Package must work with TypeORM 0.3.x and NestJS 11.x.
- **Redis fail-closed**: `RedisService` must propagate connection errors (do not silently swallow). Consuming modules decide how to handle failures.
- **Naming convention**: Entity class and properties in Portuguese (ADR-005). Module/service code in English.

## 5. Implementation Steps

1. **Create package directory structure**:
   ```
   packages/database/
   ├── package.json
   ├── project.json
   ├── tsconfig.json
   ├── tsconfig.lib.json
   ├── src/
   │   ├── index.ts
   │   ├── entidade-base.ts
   │   ├── typeorm-config.ts
   │   ├── naming-strategy.ts
   │   └── redis/
   │       ├── index.ts
   │       ├── redis.module.ts
   │       └── redis.service.ts
   ```

2. **Create `packages/database/package.json`**:
   - Name: `@satie/database`
   - Set `"main"` and `"types"` pointing to `src/index.ts` (or compiled output)
   - Dependencies: `typeorm`, `typeorm-naming-strategies`, `ioredis`, `pg`
   - Peer dependencies: `@nestjs/common`, `@nestjs/config`, `@nestjs/typeorm`

3. **Create `packages/database/project.json`**: Nx project config with `"projectType": "library"`, `"sourceRoot": "packages/database/src"`.

4. **Create `packages/database/tsconfig.json`**: Extends `../../tsconfig.base.json`.

5. **Create `packages/database/tsconfig.lib.json`**: Library build config extending `./tsconfig.json`, `compilerOptions.outDir: "../../dist/packages/database"`, include `src/**/*.ts`.

6. **Implement `packages/database/src/entidade-base.ts`** — `EntidadeBase` abstract class:

```typescript
import {
  CreateDateColumn,
  UpdateDateColumn,
  DeleteDateColumn,
  Column,
} from "typeorm";

export abstract class EntidadeBase {
  @Column({ type: "varchar", length: 255 })
  criadoPor: string;

  @Column({ type: "varchar", length: 255 })
  modificadoPor: string;

  @CreateDateColumn({ type: "timestamp" })
  criadoAs: Date;

  @UpdateDateColumn({ type: "timestamp" })
  modificadoAs: Date;

  @DeleteDateColumn({ type: "timestamp", nullable: true })
  deletadoAs: Date | null;

  @Column({ type: "varchar", length: 255, nullable: true })
  deletadoPor: string | null;
}
```

7. **Implement `packages/database/src/naming-strategy.ts`**:

```typescript
export { SnakeNamingStrategy } from "typeorm-naming-strategies";
```

8. **Implement `packages/database/src/typeorm-config.ts`** — `createTypeOrmConfig()` helper:

```typescript
import type { ConfigService } from "@nestjs/config";
import type { TypeOrmModuleOptions } from "@nestjs/typeorm";
import { SnakeNamingStrategy } from "./naming-strategy";

export function createTypeOrmConfig(
  configService: ConfigService,
  entities: Function[],
  migrations: Function[],
): TypeOrmModuleOptions {
  return {
    type: "postgres",
    host: configService.get<string>("DATABASE_HOST", "localhost"),
    port: configService.get<number>("DATABASE_PORT", 5432),
    username: configService.get<string>("DATABASE_USER", "satie"),
    password: configService.get<string>("DATABASE_PASSWORD", "satie"),
    database: configService.get<string>("DATABASE_NAME", "satie_dev"),
    entities,
    migrations,
    namingStrategy: new SnakeNamingStrategy(),
    synchronize: false,
  };
}
```

9. **Implement `packages/database/src/redis/redis.service.ts`**:

```typescript
import { Injectable, OnModuleDestroy } from "@nestjs/common";
import { ConfigService } from "@nestjs/config";
import Redis from "ioredis";

@Injectable()
export class RedisService extends Redis implements OnModuleDestroy {
  constructor(configService: ConfigService) {
    super({
      host: configService.get<string>("REDIS_HOST", "localhost"),
      port: configService.get<number>("REDIS_PORT", 6379),
    });
  }

  async onModuleDestroy() {
    await this.quit();
  }
}
```

10. **Implement `packages/database/src/redis/redis.module.ts`**:

```typescript
import { Global, Module } from "@nestjs/common";
import { RedisService } from "./redis.service";

@Global()
@Module({
  providers: [RedisService],
  exports: [RedisService],
})
export class RedisModule {}
```

11. **Implement `packages/database/src/redis/index.ts`**:

```typescript
export { RedisModule } from "./redis.module";
export { RedisService } from "./redis.service";
```

12. **Implement `packages/database/src/index.ts`** (barrel export):

```typescript
export { EntidadeBase } from "./entidade-base";
export { createTypeOrmConfig } from "./typeorm-config";
export { SnakeNamingStrategy } from "./naming-strategy";
export { RedisModule, RedisService } from "./redis/index";
```

13. **Update `tsconfig.base.json`**: Add path alias under `compilerOptions.paths`:

```json
"paths": {
  "@satie/database": ["packages/database/src/index.ts"]
}
```

14. **Install dependencies**: Run `pnpm install` from root to link the workspace package. Install package-specific deps:
    - `cd packages/database && pnpm add typeorm typeorm-naming-strategies ioredis pg`
    - `cd packages/database && pnpm add -D @types/ioredis`
    - Peer deps: `@nestjs/common`, `@nestjs/config`, `@nestjs/typeorm`

15. **Update root `tsconfig.json`**: Add project reference for `packages/database`.

16. **Verify**: Import `EntidadeBase` and `RedisService` in a test file or verify TypeScript compilation passes with `pnpm nx run database:build` (or `tsc --noEmit` in the package).

## 6. Acceptance Criteria

- AC1: `packages/database/` exists with all specified files.
- AC2: `@satie/database` is importable from other workspace packages (path alias works).
- AC3: `EntidadeBase` has all 6 audit/soft-delete fields with correct TypeORM decorators.
- AC4: `createTypeOrmConfig()` returns a valid `TypeOrmModuleOptions` object with `SnakeNamingStrategy`.
- AC5: `RedisModule` is a global NestJS module exporting `RedisService`.
- AC6: `RedisService` connects to Redis using host/port from `ConfigService`.
- AC7: TypeScript compilation passes without errors.

## 7. Test Scenarios

- **Scenario 1 (happy path)**: Import `EntidadeBase` in another package → compiles successfully → all 6 fields are present.
- **Scenario 2 (happy path)**: `createTypeOrmConfig(mockConfigService, [], [])` returns config with `type: "postgres"`, `namingStrategy` instanceof `SnakeNamingStrategy`, `synchronize: false`.
- **Scenario 3 (happy path)**: `RedisService` instantiation with mocked `ConfigService` creates an ioredis client targeting the configured host/port.
- **Scenario 4 (edge case)**: `createTypeOrmConfig` uses defaults when `ConfigService` returns undefined for optional vars.

## 8. Definition of Ready (to start Step 4)

- [x] Status is `Draft` (will be `Ready` after approval).
- [x] Scope is explicit and bounded.
- [x] Required contracts are defined — export API, EntidadeBase columns, config helper signature.
- [x] Technical specification is detailed enough for independent implementation.
- [x] For data-impact tasks, column details fully documented.
- [x] Acceptance criteria are testable.
- [x] Open questions are resolved.
- [x] Dependencies are completed or clearly sequenced — no dependencies.

## 9. Definition of Done

- [x] Status moved to `Done`.
- [x] Acceptance criteria validated.
- [x] Edge cases covered.
- [x] Links to changed files/PR/tests are registered.
- [x] Feature spec and plan traceability remains intact.

## 10. Jira Mapping (Optional)

- **Issue Type**: Task
- **Summary**: Create shared database package @satie/database
- **Description**: Implement EntidadeBase, TypeORM config helper, SnakeNamingStrategy re-export, Redis module in packages/database/.
- **Priority**: Highest
- **Labels**: Backend, Infrastructure
- **Custom field (Tipo)**: Feature
- **Custom field (Tamanho)**: 2 Dias

## 11. Status Log

| Date       | Status | Notes |
| ---------- | ------ | ----- |
| 2026-04-15 | Draft  | Task created from approved feature plan |
| 2026-04-15 | Ready  | Task approved for implementation |
| 2026-04-15 | In Progress | Implementation started |
| 2026-04-15 | Done | All files created, tests passing (9/9), biome clean, TypeScript compilation clean |
