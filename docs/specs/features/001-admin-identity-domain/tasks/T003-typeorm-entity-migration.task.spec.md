# Task Spec: TypeORM Setup, Entity, and Migrations

- **Feature ID**: 001-admin-identity-domain
- **Task ID**: T003
- **Status**: Done
- **Type**: Task
- **Parallelizable**: No
- **Parallelization Notes**: Depends on T001 (Docker Compose for database) and T002 (@satie/database package). Cannot start until both are done.
- **Date**: 2026-04-15
- **Owner**: TBD
- **Feature Folder**: docs/specs/features/001-admin-identity-domain/
- **Feature Spec**: docs/specs/features/001-admin-identity-domain/feature.spec.md
- **Feature Plan**: docs/specs/features/001-admin-identity-domain/plan.spec.md
- **Task File**: docs/specs/features/001-admin-identity-domain/tasks/T003-typeorm-entity-migration.task.spec.md
- **Related ADRs**: ADR-002, ADR-005
- **Dependencies**: T001, T002

## 1. Purpose

Configure TypeORM in the Admin backend using `@satie/database`, create the `UsuarioAdmin` entity, generate the initial migration to create the `usuario_admin` table, seed the super-admin user, set up `ConfigModule` for environment variables, bootstrap Swagger/Scalar API docs, and configure the global `ValidationPipe`.

## 2. Scope

### In Scope

- Create `UsuarioAdmin` entity extending `EntidadeBase`
- Configure `TypeOrmModule.forRootAsync()` in `AppModule` using `createTypeOrmConfig()` from `@satie/database`
- Configure `ConfigModule.forRoot()` in `AppModule`
- Create TypeORM DataSource config file for CLI migrations (`apps/admin/backend/src/config/typeorm.config.ts`)
- Create migration: `CreateUsuarioAdmin` (table creation with all columns, constraints, indexes)
- Create migration: `SeedSuperAdmin` (idempotent super-admin insert)
- Set up Swagger/Scalar in `main.ts` with `DocumentBuilder`
- Set up global `ValidationPipe` in `main.ts`
- Install all required dependencies in `apps/admin/backend/package.json`
- Import `RedisModule` from `@satie/database` in `AppModule`

### Out of Scope

- CRUD endpoints for users (T004)
- Auth endpoints (T005)
- JWT/Passport configuration (T005)
- Redis usage for token revocation (T005)

## 3. Context from Feature Plan

- **Plan slice**: Slice 3 — TypeORM + Entity + Migration Setup
- **Requirement refs**: FR-002, FR-009, FR-010, FR-011, FR-012, FR-014, FR-015, FR-016
- **Affected Paths**:
  - `apps/admin/backend/src/identity/users/entities/usuario-admin.entity.ts` (new)
  - `apps/admin/backend/src/migrations/<timestamp>-CreateUsuarioAdmin.ts` (new)
  - `apps/admin/backend/src/migrations/<timestamp>-SeedSuperAdmin.ts` (new)
  - `apps/admin/backend/src/config/typeorm.config.ts` (new)
  - `apps/admin/backend/src/app/app.module.ts` (modify)
  - `apps/admin/backend/src/main.ts` (modify)
  - `apps/admin/backend/package.json` (modify)
- **Why this task exists**: Data layer must be established before any business logic can be built on top.

## 4. Technical Specification (Required)

### 4.1 Backend and API Contracts (when applicable)

**Swagger/Scalar setup** (no domain endpoints yet, but infrastructure is configured):

- API title: `"Admin API"`
- API version: `"1.0"`
- Bearer auth scheme: `addBearerAuth()`
- Swagger JSON: `/api/docs-json`
- Scalar UI: `/api/docs`

**ValidationPipe** global config in `main.ts`:

```typescript
app.useGlobalPipes(
  new ValidationPipe({
    whitelist: true,
    forbidNonWhitelisted: true,
    transform: true,
  }),
);
```

### 4.2 Database Specification (mandatory when data impact exists)

Database object names follow Portuguese convention (ADR-005).

#### Tables and Actions

| Schema | Tabela | Ação (Create/Alter/Drop) | Motivo |
| ------ | ------ | ------------------------ | ------ |
| public | `usuario_admin` | Create | Store admin user accounts |

#### Fields Specification

| Tabela | Campo | Tipo SQL | Nullable | Default | PK | FK (ref) | Unique | Index | Notes |
| ------ | ----- | -------- | -------- | ------- | -- | -------- | ------ | ----- | ----- |
| `usuario_admin` | `id_usuario_admin` | UUID | No | `gen_random_uuid()` | Yes | N/A | Yes | PK | Primary key |
| `usuario_admin` | `email` | VARCHAR(255) | No | — | No | N/A | Yes | `idx_usuario_admin_email` | Unique constraint for login lookup |
| `usuario_admin` | `senha` | VARCHAR(255) | No | — | No | N/A | No | — | bcrypt hash |
| `usuario_admin` | `criado_por` | VARCHAR(255) | No | — | No | N/A | No | — | Inherited from `EntidadeBase` |
| `usuario_admin` | `modificado_por` | VARCHAR(255) | No | — | No | N/A | No | — | Inherited from `EntidadeBase` |
| `usuario_admin` | `criado_as` | TIMESTAMP | No | `NOW()` | No | N/A | No | — | Inherited from `EntidadeBase` |
| `usuario_admin` | `modificado_as` | TIMESTAMP | No | `NOW()` | No | N/A | No | — | Inherited from `EntidadeBase` |
| `usuario_admin` | `deletado_as` | TIMESTAMP | Yes | NULL | No | N/A | No | — | Soft delete marker |
| `usuario_admin` | `deletado_por` | VARCHAR(255) | Yes | NULL | No | N/A | No | — | Soft delete actor |

#### Constraints and Indexes

- **PK**: `pk_usuario_admin` — `id_usuario_admin`
- **UNIQUE**: `uq_usuario_admin_email` — `email`
- **INDEX**: `idx_usuario_admin_email` — `email` (B-tree, for login lookup performance)

#### Migration and Data Safety

- **Migration files**:
  - `apps/admin/backend/src/migrations/<timestamp>-CreateUsuarioAdmin.ts`
  - `apps/admin/backend/src/migrations/<timestamp>-SeedSuperAdmin.ts`

- **Forward migration steps (CreateUsuarioAdmin)**:
  1. Create table `usuario_admin` with all columns, PK, unique constraint on `email`, index on `email`.

- **Rollback steps (CreateUsuarioAdmin)**:
  1. Drop table `usuario_admin`.

- **Forward migration steps (SeedSuperAdmin)**:
  1. Check if any admin user exists (`SELECT COUNT(*) FROM usuario_admin`).
  2. If count is 0, insert super-admin with:
     - `email`: value from `ADMIN_SEED_EMAIL` env var (default `admin@satie.local`)
     - `senha`: bcrypt hash of `ADMIN_SEED_PASSWORD` env var
     - `criado_por`: `'system'`
     - `modificado_por`: `'system'`
  3. If count > 0, skip (idempotent).

- **Rollback steps (SeedSuperAdmin)**:
  1. Delete from `usuario_admin` where `email = ADMIN_SEED_EMAIL`.

- **Validation queries**:
  - `SELECT * FROM usuario_admin WHERE email = 'admin@satie.local'` — confirms seed exists
  - `SELECT COUNT(*) FROM usuario_admin` — confirms exactly 1 record after fresh seed

### 4.3 Frontend and UX Contracts (when applicable)

N/A.

### 4.4 Cross-Cutting Constraints

- **Security**: Seed password MUST be bcrypt-hashed in the migration. Never store plaintext.
- **Performance**: Index on `email` for fast login lookups.
- **Observability**: TypeORM logging can be enabled via `logging: true` in dev config (optional, not mandatory).
- **Compatibility**: `synchronize: false` — migrations are the only schema management mechanism.

## 5. Implementation Steps

1. **Install dependencies** in `apps/admin/backend/`:
   ```bash
   cd apps/admin/backend
   pnpm add @nestjs/typeorm @nestjs/config @nestjs/swagger @satie/database class-validator class-transformer bcrypt uuid
   pnpm add -D @types/bcrypt @types/uuid
   ```

2. **Create `apps/admin/backend/src/identity/users/entities/usuario-admin.entity.ts`**:

```typescript
import { Entity, PrimaryGeneratedColumn, Column } from "typeorm";
import { EntidadeBase } from "@satie/database";

@Entity("usuario_admin")
export class UsuarioAdmin extends EntidadeBase {
  @PrimaryGeneratedColumn("uuid")
  idUsuarioAdmin: string;

  @Column({ type: "varchar", length: 255, unique: true })
  email: string;

  @Column({ type: "varchar", length: 255 })
  senha: string;
}
```

3. **Create `apps/admin/backend/src/config/typeorm.config.ts`** (DataSource for CLI):

```typescript
import { DataSource } from "typeorm";
import { SnakeNamingStrategy } from "@satie/database";
import { UsuarioAdmin } from "../identity/users/entities/usuario-admin.entity";
import { config } from "dotenv";

config({ path: "../../../.env" });

export default new DataSource({
  type: "postgres",
  host: process.env.DATABASE_HOST || "localhost",
  port: Number(process.env.DATABASE_PORT) || 5432,
  username: process.env.DATABASE_USER || "satie",
  password: process.env.DATABASE_PASSWORD || "satie",
  database: process.env.DATABASE_NAME || "satie_dev",
  entities: [UsuarioAdmin],
  migrations: [__dirname + "/../migrations/*{.ts,.js}"],
  namingStrategy: new SnakeNamingStrategy(),
  synchronize: false,
});
```

4. **Update `apps/admin/backend/src/app/app.module.ts`**:

```typescript
import { Module } from "@nestjs/common";
import { ConfigModule, ConfigService } from "@nestjs/config";
import { TypeOrmModule } from "@nestjs/typeorm";
import { createTypeOrmConfig } from "@satie/database";
import { RedisModule } from "@satie/database";
import { AppController } from "./app.controller";
import { AppService } from "./app.service";
import { UsuarioAdmin } from "../identity/users/entities/usuario-admin.entity";

@Module({
  imports: [
    ConfigModule.forRoot({ isGlobal: true }),
    TypeOrmModule.forRootAsync({
      imports: [ConfigModule],
      useFactory: (configService: ConfigService) =>
        createTypeOrmConfig(configService, [UsuarioAdmin], []),
      inject: [ConfigService],
    }),
    RedisModule,
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

5. **Update `apps/admin/backend/src/main.ts`** — add Swagger and ValidationPipe:

```typescript
import { Logger, ValidationPipe } from "@nestjs/common";
import { NestFactory } from "@nestjs/core";
import { DocumentBuilder, SwaggerModule } from "@nestjs/swagger";
import { AppModule } from "./app/app.module";

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  const globalPrefix = "api";
  app.setGlobalPrefix(globalPrefix);

  app.useGlobalPipes(
    new ValidationPipe({
      whitelist: true,
      forbidNonWhitelisted: true,
      transform: true,
    }),
  );

  const swaggerConfig = new DocumentBuilder()
    .setTitle("Admin API")
    .setVersion("1.0")
    .addBearerAuth()
    .build();
  const document = SwaggerModule.createDocument(app, swaggerConfig);
  SwaggerModule.setup("api/docs", app, document);

  const port = process.env.PORT || 3000;
  await app.listen(port);
  Logger.log(`🚀 Application is running on: http://localhost:${port}/${globalPrefix}`);
}

bootstrap();
```

6. **Create migration `CreateUsuarioAdmin`**: Use TypeORM CLI to generate, or write manually:
   ```bash
   pnpm typeorm migration:generate -d apps/admin/backend/src/config/typeorm.config.ts apps/admin/backend/src/migrations/CreateUsuarioAdmin
   ```
   If manual, ensure the migration creates the table with all columns from section 4.2.

7. **Create migration `SeedSuperAdmin`**: Write manually — insert super-admin with bcrypt-hashed password (using `bcrypt.hashSync()` at migration time), idempotent check (only insert if no users exist).

8. **Run migrations**: 
   ```bash
   docker compose up -d
   pnpm typeorm migration:run -d apps/admin/backend/src/config/typeorm.config.ts
   ```

9. **Verify**: Start the app with `pnpm nx serve backend`, visit `http://localhost:3000/api/docs` to confirm Swagger UI loads. Check database: `SELECT * FROM usuario_admin` shows the seeded admin.

## 6. Acceptance Criteria

- AC1: `UsuarioAdmin` entity exists at the specified path, extends `EntidadeBase`, has `idUsuarioAdmin`, `email`, `senha` columns.
- AC2: `AppModule` imports `ConfigModule.forRoot()`, `TypeOrmModule.forRootAsync()` with `createTypeOrmConfig`, and `RedisModule`.
- AC3: Running `typeorm migration:run` creates the `usuario_admin` table with all columns, constraints, and indexes from section 4.2.
- AC4: `SeedSuperAdmin` migration inserts a super-admin user with bcrypt-hashed password (idempotent — skips if users exist).
- AC5: Swagger UI is accessible at `/api/docs` with Bearer auth configured.
- AC6: `ValidationPipe` is globally configured with `whitelist`, `forbidNonWhitelisted`, and `transform`.
- AC7: The app starts successfully and connects to PostgreSQL and Redis.

## 7. Test Scenarios

- **Scenario 1 (happy path)**: Run migrations on a fresh database → `usuario_admin` table exists with correct schema → super-admin user seeded.
- **Scenario 2 (idempotent seed)**: Run `SeedSuperAdmin` twice → only one admin user exists (no duplicate).
- **Scenario 3 (rollback)**: Revert `CreateUsuarioAdmin` → table is dropped.
- **Scenario 4 (app startup)**: `pnpm nx serve backend` → app starts → connects to DB → Swagger at `/api/docs`.
- **Scenario 5 (validation)**: `SELECT * FROM information_schema.columns WHERE table_name = 'usuario_admin'` lists all expected columns with correct types.

## 8. Definition of Ready (to start Step 4)

- [x] Status is `Draft` (will be `Ready` after approval).
- [x] Scope is explicit and bounded.
- [x] Required contracts are defined — entity, migration, Swagger, ValidationPipe.
- [x] Technical specification is detailed enough for independent implementation.
- [x] For data-impact tasks, table/field/type/constraint/index/migration details are fully documented.
- [x] Acceptance criteria are testable.
- [x] Open questions are resolved.
- [x] Dependencies are completed or clearly sequenced — T001, T002 required first.

## 9. Definition of Done

- [x] Status moved to `Done`.
- [x] Acceptance criteria validated.
- [x] Edge cases covered (idempotent seed, rollback).
- [x] Links to changed files/PR/tests are registered.
- [x] Feature spec and plan traceability remains intact.

## 10. Jira Mapping (Optional)

- **Issue Type**: Task
- **Summary**: TypeORM setup, UsuarioAdmin entity, and migrations
- **Description**: Configure TypeORM in Admin backend, create UsuarioAdmin entity, generate migrations, seed super-admin, set up Swagger and ValidationPipe.
- **Priority**: Highest
- **Labels**: Backend
- **Custom field (Tipo)**: Feature
- **Custom field (Tamanho)**: 2 Dias

## 11. Status Log

| Date       | Status | Notes |
| ---------- | ------ | ----- |
| 2026-04-15 | Draft  | Task created from approved feature plan |
| 2026-04-15 | Ready  | Task approved for implementation |
| 2026-04-15 | In Progress | Implementation started |
| 2026-04-15 | Done | Migration CLI fixed: created tsconfig.migrations.json to decouple SWC compilation from path resolution, updated all Nx targets to use pnpm exec |
