# Task Spec: Database Entities and Migration

- **Feature ID**: 003-admin-subscription-domain
- **Task ID**: T001
- **Status**: Done
- **Type**: Task
- **Parallelizable**: No
- **Parallelization Notes**: This is the foundational data-layer task — all other tasks depend on its entities and migration.
- **Date**: 2026-04-17
- **Last Updated**: 2026-04-17
- **Owner**: Guilherme Moraes
- **Feature Folder**: docs/specs/features/003-admin-subscription-domain/
- **Feature Spec**: docs/specs/features/003-admin-subscription-domain/feature.spec.md
- **Feature Plan**: docs/specs/features/003-admin-subscription-domain/plan.spec.md
- **Task File**: docs/specs/features/003-admin-subscription-domain/tasks/T001-database-entities-and-migration.task.spec.md
- **Related ADRs**: ADR-004, ADR-005
- **Dependencies**: N/A

## 1. Purpose

Create the four TypeORM entities (`Produto`, `Permissao`, `Plano`, `PlanoPermissao`) and a single database migration that creates all four tables in dependency order. Register the new entities in the app module's TypeORM configuration and the CLI DataSource config. This establishes the data layer foundation for the entire subscription domain.

## 2. Scope

### In Scope

- TypeORM entity: `Produto` (extends `EntidadeBase`)
- TypeORM entity: `Permissao` (extends `EntidadeBase`, FK to `Produto`)
- TypeORM entity: `Plano` (extends `EntidadeBase`, FK to `Produto`)
- TypeORM entity: `PlanoPermissao` (extends `EntidadeBase`, FK to `Plano` and `Permissao`, unique composite constraint)
- Single migration file creating all 4 tables with FKs, constraints, and indexes
- Update `app.module.ts` to register new entities in `createTypeOrmConfig()`
- Update `typeorm.config.ts` to add new entities to CLI DataSource

### Out of Scope

- NestJS modules, controllers, services, or DTOs (T002, T003)
- Seed data for products or permissions (future task)
- API endpoints (T002, T003)
- Tests for entities (covered by integration tests in T004)

## 3. Context from Feature Plan

- **Plan slice**: Slice 1 — Database Entities and Migration
- **Requirement refs**: FR-011, FR-013, FR-014, FR-015, SC-001
- **Affected Paths**:
  - `apps/admin/backend/src/subscription/products/entities/produto.entity.ts` (create)
  - `apps/admin/backend/src/subscription/products/entities/permissao.entity.ts` (create)
  - `apps/admin/backend/src/subscription/plans/entities/plano.entity.ts` (create)
  - `apps/admin/backend/src/subscription/plans/entities/plano-permissao.entity.ts` (create)
  - `apps/admin/backend/src/migrations/<timestamp>-CreateSubscriptionTables.ts` (create)
  - `apps/admin/backend/src/app/app.module.ts` (modify)
  - `apps/admin/backend/src/config/typeorm.config.ts` (modify)
- **Why this task exists**: All other subscription tasks depend on the entities and database tables being in place.

## 4. Technical Specification (Required)

### 4.1 Backend and API Contracts (when applicable)

N/A — This task creates only entities and a migration. No API endpoints.

### 4.2 Database Specification (mandatory when data impact exists)

Database object names must follow project convention in Portuguese.

#### Tables and Actions

| Schema | Tabela | Acao (Create/Alter/Drop) | Motivo |
| ------ | ------ | ------------------------ | ------ |
| public | `produto` | Create | Platform product catalog (e.g., Satie) |
| public | `permissao` | Create | Product-scoped permission/capability catalog |
| public | `plano` | Create | Subscription plan tier for a specific product |
| public | `plano_permissao` | Create | Junction table linking plans to permissions |

#### Fields Specification

**Table: `produto`**

| Tabela | Campo | Tipo SQL | Nullable | Default | PK | FK (ref) | Unique | Index | Notes |
| ------ | ----- | -------- | -------- | ------- | -- | -------- | ------ | ----- | ----- |
| produto | id_produto | UUID | No | gen_random_uuid() | Yes | N/A | Yes | PK | Primary key |
| produto | descricao | VARCHAR(255) | No | — | No | N/A | No | — | Product description |
| produto | codigo_produto | VARCHAR(100) | No | — | No | N/A | Yes | idx_produto_codigo_produto | Unique product code |
| produto | criado_por | VARCHAR(255) | No | — | No | N/A | No | — | EntidadeBase |
| produto | modificado_por | VARCHAR(255) | No | — | No | N/A | No | — | EntidadeBase |
| produto | criado_as | TIMESTAMP | No | NOW() | No | N/A | No | — | EntidadeBase |
| produto | modificado_as | TIMESTAMP | No | NOW() | No | N/A | No | — | EntidadeBase |
| produto | deletado_as | TIMESTAMP | Yes | NULL | No | N/A | No | — | EntidadeBase |
| produto | deletado_por | VARCHAR(255) | Yes | NULL | No | N/A | No | — | EntidadeBase |

**Table: `permissao`**

| Tabela | Campo | Tipo SQL | Nullable | Default | PK | FK (ref) | Unique | Index | Notes |
| ------ | ----- | -------- | -------- | ------- | -- | -------- | ------ | ----- | ----- |
| permissao | id_permissao | UUID | No | gen_random_uuid() | Yes | N/A | Yes | PK | Primary key |
| permissao | codigo_permissao | VARCHAR(100) | No | — | No | N/A | Yes | idx_permissao_codigo_permissao | Unique permission code |
| permissao | id_produto | UUID | No | — | No | produto(id_produto) | No | idx_permissao_id_produto | FK to product |
| permissao | descricao | VARCHAR(255) | No | — | No | N/A | No | — | Permission description |
| permissao | criado_por | VARCHAR(255) | No | — | No | N/A | No | — | EntidadeBase |
| permissao | modificado_por | VARCHAR(255) | No | — | No | N/A | No | — | EntidadeBase |
| permissao | criado_as | TIMESTAMP | No | NOW() | No | N/A | No | — | EntidadeBase |
| permissao | modificado_as | TIMESTAMP | No | NOW() | No | N/A | No | — | EntidadeBase |
| permissao | deletado_as | TIMESTAMP | Yes | NULL | No | N/A | No | — | EntidadeBase |
| permissao | deletado_por | VARCHAR(255) | Yes | NULL | No | N/A | No | — | EntidadeBase |

**Table: `plano`**

| Tabela | Campo | Tipo SQL | Nullable | Default | PK | FK (ref) | Unique | Index | Notes |
| ------ | ----- | -------- | -------- | ------- | -- | -------- | ------ | ----- | ----- |
| plano | id_plano | UUID | No | gen_random_uuid() | Yes | N/A | Yes | PK | Primary key |
| plano | nome | VARCHAR(150) | No | — | No | N/A | No | — | Plan name |
| plano | descricao | TEXT | Yes | NULL | No | N/A | No | — | Plan description |
| plano | cor | VARCHAR(7) | No | — | No | N/A | No | — | Hex color (#RRGGBB) |
| plano | ativo | BOOLEAN | No | true | No | N/A | No | — | Active flag |
| plano | id_produto | UUID | No | — | No | produto(id_produto) | No | idx_plano_id_produto | FK to product (immutable after creation) |
| plano | criado_por | VARCHAR(255) | No | — | No | N/A | No | — | EntidadeBase |
| plano | modificado_por | VARCHAR(255) | No | — | No | N/A | No | — | EntidadeBase |
| plano | criado_as | TIMESTAMP | No | NOW() | No | N/A | No | — | EntidadeBase |
| plano | modificado_as | TIMESTAMP | No | NOW() | No | N/A | No | — | EntidadeBase |
| plano | deletado_as | TIMESTAMP | Yes | NULL | No | N/A | No | — | EntidadeBase |
| plano | deletado_por | VARCHAR(255) | Yes | NULL | No | N/A | No | — | EntidadeBase |

**Table: `plano_permissao`**

| Tabela | Campo | Tipo SQL | Nullable | Default | PK | FK (ref) | Unique | Index | Notes |
| ------ | ----- | -------- | -------- | ------- | -- | -------- | ------ | ----- | ----- |
| plano_permissao | id_plano_permissao | UUID | No | gen_random_uuid() | Yes | N/A | Yes | PK | Primary key |
| plano_permissao | id_plano | UUID | No | — | No | plano(id_plano) | No | idx_plano_permissao_id_plano | FK to plan |
| plano_permissao | id_permissao | UUID | No | — | No | permissao(id_permissao) | No | idx_plano_permissao_id_permissao | FK to permission |
| plano_permissao | criado_por | VARCHAR(255) | No | — | No | N/A | No | — | EntidadeBase |
| plano_permissao | modificado_por | VARCHAR(255) | No | — | No | N/A | No | — | EntidadeBase |
| plano_permissao | criado_as | TIMESTAMP | No | NOW() | No | N/A | No | — | EntidadeBase |
| plano_permissao | modificado_as | TIMESTAMP | No | NOW() | No | N/A | No | — | EntidadeBase |
| plano_permissao | deletado_as | TIMESTAMP | Yes | NULL | No | N/A | No | — | EntidadeBase |
| plano_permissao | deletado_por | VARCHAR(255) | Yes | NULL | No | N/A | No | — | EntidadeBase |

#### Constraints and Indexes

**Constraints:**

| Constraint | Type | Table | Columns | Notes |
|------------|------|-------|---------|-------|
| `pk_produto` | PRIMARY KEY | `produto` | `id_produto` | — |
| `uq_produto_codigo_produto` | UNIQUE | `produto` | `codigo_produto` | — |
| `pk_permissao` | PRIMARY KEY | `permissao` | `id_permissao` | — |
| `uq_permissao_codigo_permissao` | UNIQUE | `permissao` | `codigo_permissao` | — |
| `fk_permissao_produto` | FOREIGN KEY | `permissao` | `id_produto` → `produto(id_produto)` | — |
| `pk_plano` | PRIMARY KEY | `plano` | `id_plano` | — |
| `fk_plano_produto` | FOREIGN KEY | `plano` | `id_produto` → `produto(id_produto)` | — |
| `pk_plano_permissao` | PRIMARY KEY | `plano_permissao` | `id_plano_permissao` | — |
| `fk_plano_permissao_plano` | FOREIGN KEY | `plano_permissao` | `id_plano` → `plano(id_plano)` | — |
| `fk_plano_permissao_permissao` | FOREIGN KEY | `plano_permissao` | `id_permissao` → `permissao(id_permissao)` | — |
| `uq_plano_permissao_plano_permissao` | UNIQUE | `plano_permissao` | `(id_plano, id_permissao)` | Prevent duplicate permission assignments |

**Indexes:**

| Index | Table | Columns | Type | Reason |
|-------|-------|---------|------|--------|
| `idx_produto_codigo_produto` | `produto` | `codigo_produto` | B-tree | Lookup by product code |
| `idx_permissao_codigo_permissao` | `permissao` | `codigo_permissao` | B-tree | Lookup by permission code |
| `idx_permissao_id_produto` | `permissao` | `id_produto` | B-tree | List permissions by product |
| `idx_plano_id_produto` | `plano` | `id_produto` | B-tree | List plans by product |
| `idx_plano_permissao_id_plano` | `plano_permissao` | `id_plano` | B-tree | List permissions for a plan |
| `idx_plano_permissao_id_permissao` | `plano_permissao` | `id_permissao` | B-tree | Reverse lookup: plans using a permission |

#### Migration and Data Safety

- **Migration file**: `apps/admin/backend/src/migrations/<timestamp>-CreateSubscriptionTables.ts`
- **Forward migration steps**:
  1. Create table `produto` with all columns, PK, and unique constraint on `codigo_produto`.
  2. Create index `idx_produto_codigo_produto` on `produto`.
  3. Create table `permissao` with all columns, PK, unique constraint on `codigo_permissao`, and FK to `produto`.
  4. Create indexes `idx_permissao_codigo_permissao` and `idx_permissao_id_produto` on `permissao`.
  5. Create table `plano` with all columns, PK, and FK to `produto`.
  6. Create index `idx_plano_id_produto` on `plano`.
  7. Create table `plano_permissao` with all columns, PK, FKs to `plano` and `permissao`, and unique constraint on `(id_plano, id_permissao)`.
  8. Create indexes `idx_plano_permissao_id_plano` and `idx_plano_permissao_id_permissao` on `plano_permissao`.
- **Rollback steps**: Drop tables in reverse dependency order: `plano_permissao` → `plano` → `permissao` → `produto`.
- **Backfill strategy**: N/A — tables are created empty.
- **Validation queries**:
  - `SELECT table_name FROM information_schema.tables WHERE table_schema = 'public' AND table_name IN ('produto', 'permissao', 'plano', 'plano_permissao');` — should return 4 rows.
  - `SELECT indexname FROM pg_indexes WHERE tablename IN ('produto', 'permissao', 'plano', 'plano_permissao');` — verify all indexes exist.

### 4.3 Frontend and UX Contracts (when applicable)

N/A — Backend-only task.

### 4.4 Cross-Cutting Constraints

- **Security/authorization**: N/A — no API surface in this task.
- **Performance expectations**: N/A — migration runs once.
- **Observability**: N/A.
- **Compatibility constraints**: Purely additive — no existing tables or code affected.

## 5. Implementation Steps

1. **Create `Produto` entity** at `apps/admin/backend/src/subscription/products/entities/produto.entity.ts`:
   - Extend `EntidadeBase` from `@satie/database`.
   - Define `@PrimaryGeneratedColumn("uuid") idProduto: string`.
   - Define `@Column({ type: "varchar", length: 255 }) descricao: string`.
   - Define `@Column({ type: "varchar", length: 100, unique: true }) codigoProduto: string`.
   - Define `@OneToMany(() => Permissao, (p) => p.produto) permissoes: Permissao[]`.
   - Define `@OneToMany(() => Plano, (p) => p.produto) planos: Plano[]`.

2. **Create `Permissao` entity** at `apps/admin/backend/src/subscription/products/entities/permissao.entity.ts`:
   - Extend `EntidadeBase`.
   - Define `@PrimaryGeneratedColumn("uuid") idPermissao: string`.
   - Define `@Column({ type: "varchar", length: 100, unique: true }) codigoPermissao: string`.
   - Define `@Column({ type: "uuid" }) idProduto: string`.
   - Define `@Column({ type: "varchar", length: 255 }) descricao: string`.
   - Define `@ManyToOne(() => Produto, (p) => p.permissoes) @JoinColumn({ name: "id_produto" }) produto: Produto`.

3. **Create `Plano` entity** at `apps/admin/backend/src/subscription/plans/entities/plano.entity.ts`:
   - Extend `EntidadeBase`.
   - Define `@PrimaryGeneratedColumn("uuid") idPlano: string`.
   - Define `@Column({ type: "varchar", length: 150 }) nome: string`.
   - Define `@Column({ type: "text", nullable: true }) descricao: string | null`.
   - Define `@Column({ type: "varchar", length: 7 }) cor: string`.
   - Define `@Column({ type: "boolean", default: true }) ativo: boolean`.
   - Define `@Column({ type: "uuid" }) idProduto: string`.
   - Define `@ManyToOne(() => Produto, (p) => p.planos) @JoinColumn({ name: "id_produto" }) produto: Produto`.
   - Define `@OneToMany(() => PlanoPermissao, (pp) => pp.plano) planoPermissoes: PlanoPermissao[]`.

4. **Create `PlanoPermissao` entity** at `apps/admin/backend/src/subscription/plans/entities/plano-permissao.entity.ts`:
   - Extend `EntidadeBase`.
   - Add `@Unique(["idPlano", "idPermissao"])` at class level.
   - Define `@PrimaryGeneratedColumn("uuid") idPlanoPermissao: string`.
   - Define `@Column({ type: "uuid" }) idPlano: string`.
   - Define `@Column({ type: "uuid" }) idPermissao: string`.
   - Define `@ManyToOne(() => Plano, (p) => p.planoPermissoes) @JoinColumn({ name: "id_plano" }) plano: Plano`.
   - Define `@ManyToOne(() => Permissao) @JoinColumn({ name: "id_permissao" }) permissao: Permissao`.

5. **Create migration** at `apps/admin/backend/src/migrations/<timestamp>-CreateSubscriptionTables.ts`:
   - Use TypeORM `Table` builder API (same pattern as existing `CreateUsuarioAdmin` migration).
   - Create all 4 tables in dependency order: `produto` → `permissao` → `plano` → `plano_permissao`.
   - Define all columns, FKs, unique constraints, and indexes per specification.
   - Implement `down()` to drop tables in reverse order.

6. **Update `app.module.ts`** at `apps/admin/backend/src/app/app.module.ts`:
   - Import `Produto`, `Permissao`, `Plano`, `PlanoPermissao` entities.
   - Add them to the `createTypeOrmConfig()` entities array: `createTypeOrmConfig(configService, [UsuarioAdmin, Produto, Permissao, Plano, PlanoPermissao], [])`.

7. **Update `typeorm.config.ts`** at `apps/admin/backend/src/config/typeorm.config.ts`:
   - Import `Produto`, `Permissao`, `Plano`, `PlanoPermissao` entities.
   - Add them to the `entities` array: `entities: [UsuarioAdmin, Produto, Permissao, Plano, PlanoPermissao]`.

8. **Run Biome check**: `pnpm nx run-many -t check` to verify all new files comply.

## 6. Acceptance Criteria

- AC1: All 4 TypeORM entities exist in the correct paths and extend `EntidadeBase`.
- AC2: Entity relationships are correctly defined (OneToMany, ManyToOne, JoinColumn with correct FK names).
- AC3: `PlanoPermissao` has a unique composite constraint on `(idPlano, idPermissao)`.
- AC4: Migration creates all 4 tables with correct columns, types, constraints, indexes, and FKs.
- AC5: Migration `down()` drops all 4 tables in reverse dependency order.
- AC6: `app.module.ts` registers all 4 new entities in `createTypeOrmConfig()`.
- AC7: `typeorm.config.ts` includes all 4 new entities in the DataSource `entities` array.
- AC8: `pnpm nx run-many -t check` (Biome) passes with zero warnings/errors.

## 7. Test Scenarios

- Scenario 1 (migration forward): Running `pnpm nx run admin-backend:migration:run` creates all 4 tables with correct schema — verify via `\d produto`, `\d permissao`, `\d plano`, `\d plano_permissao` in psql.
- Scenario 2 (migration rollback): Running `pnpm nx run admin-backend:migration:revert` drops all 4 tables cleanly.
- Scenario 3 (entity compilation): The backend compiles without TypeScript errors after adding entities and updating configs.
- Scenario 4 (Biome compliance): All new and modified files pass Biome check with zero issues.

## 8. Definition of Ready (to start Step 4)

- [x] Status is `Ready`.
- [x] Scope is explicit and bounded.
- [x] Required contracts are defined (API, DTO, event, UI state, schema).
- [x] Technical specification is detailed enough for independent implementation.
- [x] For data-impact tasks, table/field/type/constraint/index/migration details are fully documented.
- [x] Acceptance criteria are testable.
- [x] Open questions are resolved or captured as explicit assumptions.
- [x] All dependency tasks are `Done` (no dependencies).

## 9. Definition of Done

- [x] Status moved to `Done`.
- [x] All related tests (unit and integration) pass — no test failures allowed.
- [x] Biome reports zero warnings and zero errors on all changed files.
- [x] Acceptance criteria are validated by tests or clear verification evidence.
- [x] Edge cases listed in this task are covered.
- [x] Links to changed files/PR/tests are registered.
- [x] Feature spec and plan traceability remains intact.

## 10. Jira Mapping (Optional)

- **Issue Type**: Task
- **Summary**: Create subscription domain entities and migration
- **Description**: Create 4 TypeORM entities (Produto, Permissao, Plano, PlanoPermissao) and database migration for the subscription domain.
- **Priority**: Highest
- **Labels**: Backend, Database
- **Custom field (Tipo)**: Feature
- **Custom field (Tamanho)**: 1 Dia

## 11. Status Log

| Date       | Status      | Notes |
| ---------- | ----------- | ----- |
| 2026-04-17 | In Progress | Started implementation — creating entities, migration, and config updates |
| 2026-04-17 | Done        | All 4 entities created, migration created, app.module.ts and typeorm.config.ts updated, Biome clean, build passes |
| 2026-04-17 | Draft       | Task created from approved feature plan |
| 2026-04-17 | Ready       | Task approved for implementation |

## 12. Observations

- The migration uses `gen_random_uuid()` (built-in since PG 13) — no `uuid-ossp` extension needed.
- Entity files are placed under the module they belong to (`products/entities/` and `plans/entities/`) even though no module file is created in this task. T002 and T003 will create the NestJS modules.
- The `Plano` entity has a `@OneToMany` to `PlanoPermissao` but the reverse `@ManyToOne` from `PlanoPermissao` to `Permissao` does NOT define a back-reference on `Permissao` — this is intentional per the plan (no `planoPermissoes` nav property on `Permissao`).
