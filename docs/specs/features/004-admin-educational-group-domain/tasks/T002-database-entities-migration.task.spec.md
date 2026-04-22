# Task Spec: Database Entities and Migration

- **Feature ID**: 004-admin-educational-group-domain
- **Task ID**: T002
- **Status**: Done
- **Type**: Task
- **Parallelizable**: Yes
- **Parallelization Notes**: No dependency on T001 (utils package). Entities and migration don't use utility functions. Both are foundational tasks that can be developed in parallel.
- **Date**: 2026-04-20
- **Last Updated**: 2026-04-20
- **Owner**: Guilherme Moraes
- **Feature Folder**: docs/specs/features/004-admin-educational-group-domain/
- **Feature Spec**: docs/specs/features/004-admin-educational-group-domain/feature.spec.md
- **Feature Plan**: docs/specs/features/004-admin-educational-group-domain/plan.spec.md
- **Task File**: docs/specs/features/004-admin-educational-group-domain/tasks/T002-database-entities-migration.task.spec.md
- **Related ADRs**: ADR-004, ADR-005
- **Dependencies**: N/A

## 1. Purpose

Create all 5 TypeORM entity classes and the database migration for the educational group domain. Register the new entities in `app.module.ts` and `typeorm.config.ts`. This task establishes the data layer foundation that all subsequent tasks depend on.

## 2. Scope

### In Scope

- Create 5 TypeORM entity files under `apps/admin/backend/src/educational-group/entities/`
- Create 1 migration file under `apps/admin/backend/src/migrations/`
- Register all 5 entities in `app.module.ts` TypeORM config
- Register all 5 entities in `config/typeorm.config.ts` DataSource config
- All entities inherit from `EntidadeBase` (audit fields + soft delete)
- All relationships, foreign keys, unique constraints, and indexes defined in entities

### Out of Scope

- Module, controller, service, or DTO creation (T003, T004, T005)
- Utility functions (T001)
- Running the migration against any database (done during implementation, not part of deliverable)
- Seed data

## 3. Context from Feature Plan

- **Plan slice**: Slice 1 — Database Entities and Migration
- **Requirement refs**: FR-009 (EntidadeBase inheritance)
- **Affected Paths**:
  - `apps/admin/backend/src/educational-group/entities/` (NEW — 5 entity files)
  - `apps/admin/backend/src/migrations/` (NEW — 1 migration file)
  - `apps/admin/backend/src/app/app.module.ts` (MODIFIED — add entities to TypeORM config)
  - `apps/admin/backend/src/config/typeorm.config.ts` (MODIFIED — add entities to DataSource)
- **Why this task exists**: The data layer must exist before any service or controller can be built. All CRUD operations (T003–T005) depend on these entities and the migration.

## 4. Technical Specification (Required)

### 4.1 Backend and API Contracts (when applicable)

N/A — This task creates only entities and migration. No endpoints.

### 4.2 Database Specification (mandatory when data impact exists)

Database object names follow project convention in Portuguese (ADR-005).

#### Tables and Actions

| Schema | Tabela | Acao (Create/Alter/Drop) | Motivo |
| ------ | ------ | ------------------------ | ------ |
| public | `usuario_mestre` | Create | Master user credentials per educational group |
| public | `parametros_grupo_educacional` | Create | Database parameters (nome_banco) per group |
| public | `grupo_educacional` | Create | Core tenant entity — educational group |
| public | `usuario_comum` | Create | Common users within a group |
| public | `assinatura` | Create | Plan subscription links (group ↔ plan) |

#### Fields Specification

**Table: `usuario_mestre`**

| Tabela | Campo | Tipo SQL | Nullable | Default | PK | FK (ref) | Unique | Index | Notes |
| ------ | ----- | -------- | -------- | ------- | -- | -------- | ------ | ----- | ----- |
| usuario_mestre | `id_usuario_mestre` | uuid | No | gen_random_uuid() | Yes | N/A | Yes | PK | Primary key |
| usuario_mestre | `usuario` | varchar(255) | No | N/A | No | N/A | Yes | UQ | Auto-generated: `mestre-{env}-{slug}` |
| usuario_mestre | `senha` | varchar(255) | No | N/A | No | N/A | No | N/A | Bcrypt-hashed, 16-char random |
| usuario_mestre | `criado_por` | varchar(255) | No | N/A | No | N/A | No | N/A | EntidadeBase |
| usuario_mestre | `modificado_por` | varchar(255) | No | N/A | No | N/A | No | N/A | EntidadeBase |
| usuario_mestre | `criado_as` | timestamp | No | NOW() | No | N/A | No | N/A | EntidadeBase |
| usuario_mestre | `modificado_as` | timestamp | No | NOW() | No | N/A | No | N/A | EntidadeBase |
| usuario_mestre | `deletado_as` | timestamp | Yes | NULL | No | N/A | No | N/A | EntidadeBase soft delete |
| usuario_mestre | `deletado_por` | varchar(255) | Yes | NULL | No | N/A | No | N/A | EntidadeBase |

**Table: `parametros_grupo_educacional`**

| Tabela | Campo | Tipo SQL | Nullable | Default | PK | FK (ref) | Unique | Index | Notes |
| ------ | ----- | -------- | -------- | ------- | -- | -------- | ------ | ----- | ----- |
| parametros_grupo_educacional | `id_parametros_grupo_educacional` | uuid | No | gen_random_uuid() | Yes | N/A | Yes | PK | Primary key |
| parametros_grupo_educacional | `nome_banco` | varchar(255) | No | N/A | No | N/A | Yes | UQ | Auto-generated: `db_{env}_{slug}` |
| parametros_grupo_educacional | `criado_por` | varchar(255) | No | N/A | No | N/A | No | N/A | EntidadeBase |
| parametros_grupo_educacional | `modificado_por` | varchar(255) | No | N/A | No | N/A | No | N/A | EntidadeBase |
| parametros_grupo_educacional | `criado_as` | timestamp | No | NOW() | No | N/A | No | N/A | EntidadeBase |
| parametros_grupo_educacional | `modificado_as` | timestamp | No | NOW() | No | N/A | No | N/A | EntidadeBase |
| parametros_grupo_educacional | `deletado_as` | timestamp | Yes | NULL | No | N/A | No | N/A | EntidadeBase soft delete |
| parametros_grupo_educacional | `deletado_por` | varchar(255) | Yes | NULL | No | N/A | No | N/A | EntidadeBase |

**Table: `grupo_educacional`**

| Tabela | Campo | Tipo SQL | Nullable | Default | PK | FK (ref) | Unique | Index | Notes |
| ------ | ----- | -------- | -------- | ------- | -- | -------- | ------ | ----- | ----- |
| grupo_educacional | `id_grupo_educacional` | uuid | No | gen_random_uuid() | Yes | N/A | Yes | PK | Primary key |
| grupo_educacional | `descricao` | varchar(255) | No | N/A | No | N/A | No | N/A | Group description/name |
| grupo_educacional | `id_usuario_mestre` | uuid | No | N/A | No | usuario_mestre(id_usuario_mestre) | Yes | UQ + FK | 1:1 relationship |
| grupo_educacional | `id_parametros_grupo_educacional` | uuid | No | N/A | No | parametros_grupo_educacional(id_parametros_grupo_educacional) | Yes | UQ + FK | 1:1 relationship |
| grupo_educacional | `criado_por` | varchar(255) | No | N/A | No | N/A | No | N/A | EntidadeBase |
| grupo_educacional | `modificado_por` | varchar(255) | No | N/A | No | N/A | No | N/A | EntidadeBase |
| grupo_educacional | `criado_as` | timestamp | No | NOW() | No | N/A | No | N/A | EntidadeBase |
| grupo_educacional | `modificado_as` | timestamp | No | NOW() | No | N/A | No | N/A | EntidadeBase |
| grupo_educacional | `deletado_as` | timestamp | Yes | NULL | No | N/A | No | N/A | EntidadeBase soft delete |
| grupo_educacional | `deletado_por` | varchar(255) | Yes | NULL | No | N/A | No | N/A | EntidadeBase |

**Table: `usuario_comum`**

| Tabela | Campo | Tipo SQL | Nullable | Default | PK | FK (ref) | Unique | Index | Notes |
| ------ | ----- | -------- | -------- | ------- | -- | -------- | ------ | ----- | ----- |
| usuario_comum | `id_usuario_comum` | uuid | No | gen_random_uuid() | Yes | N/A | Yes | PK | Primary key |
| usuario_comum | `email` | varchar(255) | No | N/A | No | N/A | No | N/A | User email |
| usuario_comum | `senha` | varchar(255) | No | N/A | No | N/A | No | N/A | Bcrypt-hashed, auto-generated |
| usuario_comum | `verificado` | boolean | No | false | No | N/A | No | N/A | Future email verification |
| usuario_comum | `id_grupo_educacional` | uuid | No | N/A | No | grupo_educacional(id_grupo_educacional) | No | FK + IDX | N:1 relationship |
| usuario_comum | `criado_por` | varchar(255) | No | N/A | No | N/A | No | N/A | EntidadeBase |
| usuario_comum | `modificado_por` | varchar(255) | No | N/A | No | N/A | No | N/A | EntidadeBase |
| usuario_comum | `criado_as` | timestamp | No | NOW() | No | N/A | No | N/A | EntidadeBase |
| usuario_comum | `modificado_as` | timestamp | No | NOW() | No | N/A | No | N/A | EntidadeBase |
| usuario_comum | `deletado_as` | timestamp | Yes | NULL | No | N/A | No | N/A | EntidadeBase soft delete |
| usuario_comum | `deletado_por` | varchar(255) | Yes | NULL | No | N/A | No | N/A | EntidadeBase |

**Table: `assinatura`**

| Tabela | Campo | Tipo SQL | Nullable | Default | PK | FK (ref) | Unique | Index | Notes |
| ------ | ----- | -------- | -------- | ------- | -- | -------- | ------ | ----- | ----- |
| assinatura | `id_assinatura` | uuid | No | gen_random_uuid() | Yes | N/A | Yes | PK | Primary key |
| assinatura | `id_grupo_educacional` | uuid | No | N/A | No | grupo_educacional(id_grupo_educacional) | No | FK + IDX | Part of unique constraint |
| assinatura | `id_plano` | uuid | No | N/A | No | plano(id_plano) | No | FK | Part of unique constraint (cross-domain) |
| assinatura | `criado_por` | varchar(255) | No | N/A | No | N/A | No | N/A | EntidadeBase |
| assinatura | `modificado_por` | varchar(255) | No | N/A | No | N/A | No | N/A | EntidadeBase |
| assinatura | `criado_as` | timestamp | No | NOW() | No | N/A | No | N/A | EntidadeBase |
| assinatura | `modificado_as` | timestamp | No | NOW() | No | N/A | No | N/A | EntidadeBase |
| assinatura | `deletado_as` | timestamp | Yes | NULL | No | N/A | No | N/A | EntidadeBase soft delete |
| assinatura | `deletado_por` | varchar(255) | Yes | NULL | No | N/A | No | N/A | EntidadeBase |

#### Constraints and Indexes

- **UQ_usuario_mestre_usuario**: UNIQUE on `usuario_mestre(usuario)` — prevents slug collision
- **UQ_parametros_grupo_educacional_nome_banco**: UNIQUE on `parametros_grupo_educacional(nome_banco)` — prevents slug collision
- **UQ_grupo_educacional_id_usuario_mestre**: UNIQUE on `grupo_educacional(id_usuario_mestre)` — enforces 1:1
- **UQ_grupo_educacional_id_parametros**: UNIQUE on `grupo_educacional(id_parametros_grupo_educacional)` — enforces 1:1
- **FK_grupo_educacional_usuario_mestre**: FK `grupo_educacional(id_usuario_mestre)` → `usuario_mestre(id_usuario_mestre)`
- **FK_grupo_educacional_parametros**: FK `grupo_educacional(id_parametros_grupo_educacional)` → `parametros_grupo_educacional(id_parametros_grupo_educacional)`
- **FK_usuario_comum_grupo_educacional**: FK `usuario_comum(id_grupo_educacional)` → `grupo_educacional(id_grupo_educacional)`
- **FK_assinatura_grupo_educacional**: FK `assinatura(id_grupo_educacional)` → `grupo_educacional(id_grupo_educacional)`
- **FK_assinatura_plano**: FK `assinatura(id_plano)` → `plano(id_plano)` (cross-domain)
- **UQ_assinatura_grupo_plano**: UNIQUE on `assinatura(id_grupo_educacional, id_plano)` — prevents duplicate plan per group
- **IDX_usuario_comum_grupo**: INDEX on `usuario_comum(id_grupo_educacional)` — efficient lookup by group
- **IDX_assinatura_grupo**: INDEX on `assinatura(id_grupo_educacional)` — efficient lookup by group

#### Migration and Data Safety

- **Migration file**: `apps/admin/backend/src/migrations/1713100000003-CreateEducationalGroupTables.ts`
- **Forward migration steps** (ordered by FK dependencies):
  1. Create `usuario_mestre` table with PK, columns, unique constraint on `usuario`
  2. Create `parametros_grupo_educacional` table with PK, columns, unique constraint on `nome_banco`
  3. Create `grupo_educacional` table with PK, columns, FKs to `usuario_mestre` and `parametros_grupo_educacional`, unique constraints on FK columns
  4. Create `usuario_comum` table with PK, columns, FK to `grupo_educacional`, index on `id_grupo_educacional`
  5. Create `assinatura` table with PK, columns, FKs to `grupo_educacional` and `plano`, unique constraint on `(id_grupo_educacional, id_plano)`, index on `id_grupo_educacional`
- **Rollback steps** (reverse FK order):
  1. Drop `assinatura` table
  2. Drop `usuario_comum` table
  3. Drop `grupo_educacional` table
  4. Drop `parametros_grupo_educacional` table
  5. Drop `usuario_mestre` table
- **Backfill strategy**: N/A — new tables, no existing data
- **Validation queries**:
  - `SELECT table_name FROM information_schema.tables WHERE table_schema = 'public' AND table_name IN ('usuario_mestre', 'parametros_grupo_educacional', 'grupo_educacional', 'usuario_comum', 'assinatura');` — should return 5 rows after forward migration

### 4.3 Frontend and UX Contracts (when applicable)

N/A — Backend-only feature.

### 4.4 Cross-Cutting Constraints

- **Security/authorization**: N/A — entities only, no endpoints.
- **Performance expectations**: Indexes on FK columns for efficient joins and lookups.
- **Observability**: N/A.
- **Compatibility constraints**: Cross-domain FK from `assinatura.id_plano` to `plano.id_plano` (Subscription domain). The `plano` table must already exist (created in feature 003). The entity uses a `@ManyToOne` relation to `Plano` entity from the Subscription domain.

## 5. Implementation Steps

1. **Create entity files** under `apps/admin/backend/src/educational-group/entities/`:

   a. `usuario-mestre.entity.ts` — `UsuarioMestre` entity:
      - Extends `EntidadeBase` from `@satie/database`
      - `@Entity("usuario_mestre")`
      - PK: `idUsuarioMestre` (uuid, `@PrimaryGeneratedColumn("uuid")`)
      - `usuario`: varchar(255), unique
      - `senha`: varchar(255)
      - Inverse relation: `@OneToOne(() => GrupoEducacional, g => g.usuarioMestre)`

   b. `parametros-grupo-educacional.entity.ts` — `ParametrosGrupoEducacional` entity:
      - Extends `EntidadeBase`
      - `@Entity("parametros_grupo_educacional")`
      - PK: `idParametrosGrupoEducacional` (uuid)
      - `nomeBanco`: varchar(255), unique
      - Inverse relation: `@OneToOne(() => GrupoEducacional, g => g.parametros)`

   c. `grupo-educacional.entity.ts` — `GrupoEducacional` entity:
      - Extends `EntidadeBase`
      - `@Entity("grupo_educacional")`
      - PK: `idGrupoEducacional` (uuid)
      - `descricao`: varchar(255)
      - `idUsuarioMestre`: uuid (FK column)
      - `@OneToOne(() => UsuarioMestre)` + `@JoinColumn({ name: "id_usuario_mestre" })` — unique via the JoinColumn
      - `idParametrosGrupoEducacional`: uuid (FK column)
      - `@OneToOne(() => ParametrosGrupoEducacional)` + `@JoinColumn({ name: "id_parametros_grupo_educacional" })` — unique via the JoinColumn
      - `@OneToMany(() => UsuarioComum, u => u.grupoEducacional)` — common users relation
      - `@OneToMany(() => Assinatura, a => a.grupoEducacional)` — subscriptions relation

   d. `usuario-comum.entity.ts` — `UsuarioComum` entity:
      - Extends `EntidadeBase`
      - `@Entity("usuario_comum")`
      - PK: `idUsuarioComum` (uuid)
      - `email`: varchar(255)
      - `senha`: varchar(255)
      - `verificado`: boolean, default false
      - `idGrupoEducacional`: uuid (FK column)
      - `@ManyToOne(() => GrupoEducacional, g => g.usuariosComuns)` + `@JoinColumn({ name: "id_grupo_educacional" })`
      - `@Index()` on `idGrupoEducacional` column

   e. `assinatura.entity.ts` — `Assinatura` entity:
      - Extends `EntidadeBase`
      - `@Entity("assinatura")`
      - PK: `idAssinatura` (uuid)
      - `idGrupoEducacional`: uuid (FK column)
      - `idPlano`: uuid (FK column)
      - `@ManyToOne(() => GrupoEducacional, g => g.assinaturas)` + `@JoinColumn({ name: "id_grupo_educacional" })`
      - `@ManyToOne(() => Plano)` + `@JoinColumn({ name: "id_plano" })` — cross-domain FK
      - `@Index()` on `idGrupoEducacional`
      - `@Unique(["idGrupoEducacional", "idPlano"])` — composite unique constraint

2. **Create migration file** at `apps/admin/backend/src/migrations/1713100000003-CreateEducationalGroupTables.ts`:
   - Use TypeORM `MigrationInterface` with `QueryRunner`
   - `up()`: Create tables in FK dependency order (usuario_mestre → parametros → grupo_educacional → usuario_comum → assinatura)
   - `down()`: Drop tables in reverse order (assinatura → usuario_comum → grupo_educacional → parametros → usuario_mestre)
   - Use raw SQL via `queryRunner.query()` for precise control (follow existing migration patterns)

3. **Update `apps/admin/backend/src/app/app.module.ts`**:
   - Import all 5 new entity classes
   - Add them to the entity array in `TypeOrmModule.forRootAsync` `useFactory`

4. **Update `apps/admin/backend/src/config/typeorm.config.ts`**:
   - Import all 5 new entity classes
   - Add them to the `entities` array in the `DataSource` config

## 6. Acceptance Criteria

- AC1: All 5 entity files exist and compile without errors.
- AC2: Each entity extends `EntidadeBase` and has the correct decorators, columns, and relations as specified.
- AC3: Migration `up()` creates all 5 tables with correct columns, types, constraints, and indexes.
- AC4: Migration `down()` drops all 5 tables in reverse FK order.
- AC5: `app.module.ts` includes all 5 entities in the TypeORM entity list.
- AC6: `typeorm.config.ts` includes all 5 entities in the DataSource entity list.
- AC7: `Assinatura` entity has a cross-domain `@ManyToOne` relation to `Plano` from the Subscription domain.
- AC8: The composite unique constraint `(id_grupo_educacional, id_plano)` exists on the `assinatura` table.
- AC9: Biome reports zero warnings and errors on all new/modified files.

## 7. Test Scenarios

- **Entity compilation**: All entity files import correctly and TypeORM metadata is valid (verified via build).
- **Migration forward**: Running `typeorm migration:run` creates all 5 tables (verified during Step 4 implementation).
- **Migration rollback**: Running `typeorm migration:revert` drops all 5 tables cleanly.
- **Relationship integrity**: FK constraints prevent orphan records (e.g., cannot insert `usuario_comum` with non-existent `id_grupo_educacional`).
- **Unique constraints**: Duplicate `usuario` in `usuario_mestre` or `nome_banco` in `parametros_grupo_educacional` fails with constraint violation.

Note: Full integration tests are covered in T006. This task validates via build compilation and migration execution.

## 8. Definition of Ready (to start Step 4)

- [x] Status is `Ready`.
- [x] Scope is explicit and bounded.
- [x] Required contracts are defined (API, DTO, event, UI state, schema).
- [x] Technical specification is detailed enough for independent implementation.
- [x] For data-impact tasks, table/field/type/constraint/index/migration details are fully documented.
- [x] Acceptance criteria are testable.
- [x] Open questions are resolved or captured as explicit assumptions.
- [x] All dependency tasks are `Done` (if any dependencies exist).

## 9. Definition of Done

- [x] Status moved to `Done`.
- [x] All related tests (unit and integration) pass — no test failures allowed.
- [x] **Section 10 (Test Evidence) is filled** with the exact command(s) executed and their output summary proving all tests pass.
- [x] Biome reports zero warnings and zero errors on all changed files.
- [x] Acceptance criteria are validated by tests or clear verification evidence.

## 10. Test Evidence

This section is **mandatory** before marking the task as `Done`. Paste the exact test command(s) executed and a summary of their output. If tests were not executed, this section must remain empty and the task **cannot** transition to `Done`.

| # | Command | Result | Timestamp |
| - | ------- | ------ | --------- |
| 1 | `pnpm nx run admin-backend:build` | webpack compiled successfully | 2026-04-20 |
| 2 | `pnpm nx run admin-backend:typecheck` | tsc --build passed (0 errors) | 2026-04-20 |
| 3 | `npx biome check` (8 changed files) | Checked 8 files. No fixes applied. | 2026-04-20 |
| 4 | `pnpm nx run admin-backend-tests:test` | 5 suites, 48 tests passed, 0 failed | 2026-04-20 |

**Rules**:
- Every test suite relevant to the task must have a row in this table.
- "Result" must include pass/fail/skip counts copied from actual terminal output.
- If any test fails, the task stays `In Progress` — do not fill this section with failing results and mark Done.
- This section is never pre-filled during Step 3 (Task Breakdown) — it is populated only during Step 4 (Implementation).
- [x] Edge cases listed in this task are covered.
- [x] Links to changed files/PR/tests are registered.
- [x] Feature spec and plan traceability remains intact.

## 11. Status Log

| Date       | Status | Notes |
| ---------- | ------ | ----- |
| 2026-04-20 | Draft  | Task created from approved feature plan |
| 2026-04-20 | Ready  | No dependencies; ready for implementation |
| 2026-04-20 | In Progress | Implementation started — entities, migration, module registration |
| 2026-04-20 | Done | All 5 entities created, migration file created, app.module.ts and typeorm.config.ts updated. Build, typecheck, lint, and tests pass. |

## 12. Observations

- Follow existing entity patterns from `UsuarioAdmin` (Identity domain) and `Plano` (Subscription domain) for decorator usage and naming.
- The `Assinatura` entity crosses domain boundaries by referencing `Plano` from the Subscription domain. This is an accepted design decision — `assinatura` is owned by the educational-group domain with a read-only dependency on `plano`.
- Use `@JoinColumn({ name: "..." })` with explicit snake_case column names to ensure compatibility with `SnakeNamingStrategy` (even though the strategy handles it automatically, explicit names improve readability).
