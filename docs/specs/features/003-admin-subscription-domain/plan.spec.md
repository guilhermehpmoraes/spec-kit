# Feature Plan: Admin Subscription Domain

- **Feature ID**: 003-admin-subscription-domain
- **Plan ID**: PLAN-003-admin-subscription-domain
- **Status**: Completed
- **Date**: 2026-04-17
- **Last Updated**: 2026-04-17
- **Owner**: Guilherme Moraes
- **Feature Folder**: docs/specs/features/003-admin-subscription-domain/
- **Feature Spec**: docs/specs/features/003-admin-subscription-domain/feature.spec.md
- **Plan File**: docs/specs/features/003-admin-subscription-domain/plan.spec.md
- **Task Output Folder**: docs/specs/features/003-admin-subscription-domain/tasks/
- **Related ADRs**: ADR-004, ADR-005, ADR-010

## 1. Planning Goal

Define a complete, implementation-ready plan for the Admin Subscription Domain: four database tables (`produto`, `permissao`, `plano`, `plano_permissao`), TypeORM entities inheriting `EntidadeBase`, read-only endpoints for products/permissions, full CRUD for plans with atomic permission management, cross-product validation, cascade soft-delete, JWT-protected endpoints, Swagger/Scalar documentation, and unit + integration tests — with enough detail to generate self-sufficient task specs.

## 2. Summary

- **Feature objective**: Establish the subscription domain for the Admin application — a product/permission catalog with plan composition, enabling future client-plan subscription management.
- **Why now**: The subscription domain is a prerequisite for client subscription management. Plans define what product features a client can access, and this catalog needs to exist before any client-plan linking can be built.
- **Expected outcome**: A fully functional Products (read-only) + Plans (CRUD) API with atomic permission management, cross-product validation, cascade soft-delete, JWT auth, Swagger/Scalar docs, and comprehensive tests — all following existing codebase patterns from the Identity domain.
- **Implementation direction**: Bottom-up — data layer first (entities + migration), then read-only API (products/permissions), then full CRUD API (plans with permissions), then integration tests.

## 3. Technical Context

### Stack Baseline (Satie)

- **Monorepo**: Nx + pnpm
- **Backend**: NestJS
- **Frontend**: Vite + React + TanStack Router + TanStack Query + Tailwind CSS (out of scope)
- **Database**: PostgreSQL 17.4
- **Cache/Ephemeral**: Redis (existing, used by Identity domain for token revocation)
- **Tests**: Jest + Supertest (backend), Vitest + React Testing Library (frontend), Playwright (e2e)
- **Quality**: Biome (formatter + linter)

### Feature-Specific Context

- **Touched apps/packages**: `apps/admin/backend/` (new subscription module + entities + migration), `apps/admin/backend-tests/` (new unit + integration tests)
- **New dependencies**: None — all required NestJS, TypeORM, class-validator, Swagger dependencies already exist from the Identity domain.
- **Data impact**: 4 new PostgreSQL tables (`produto`, `permissao`, `plano`, `plano_permissao`) with foreign key relationships. No Redis usage in this feature.
- **Constraints**: Portuguese naming for entities/DB (ADR-005), English for all other code. Soft deletes mandatory (ADR-005). KISS-first (ADR-001). Controller-Service-Repository pattern (ADR-002). Scalar API documentation with decorator standard (ADR-010).

### Constraints and Assumptions

- **Inputs**: HTTP requests (JSON bodies) for plan CRUD; no input for products/permissions (read-only).
- **Outputs**: JSON API responses with nested DTOs; 204 for soft-delete.
- **Boundary conditions**: Non-existent product/plan IDs, cross-product permission validation, invalid color format, empty permission arrays, duplicate permission assignments.
- **Assumptions**:
  - Products and permissions are catalog data inserted via migrations (not via API) — this feature does NOT seed any data (deferred to a future task when actual product/permission data is defined).
  - No pagination, filtering, or sorting on listing endpoints — catalog sizes are small.
  - `JwtAuthGuard` from the Identity domain is available and exported via `IdentityModule → AuthModule`.
  - All existing dependencies (TypeORM, class-validator, @nestjs/swagger, etc.) are already installed.
  - Docker Compose with PostgreSQL and Redis is already configured and operational.

### Resolved Open Questions

| Question | Resolution |
|----------|------------|
| Pagination on listing endpoints? | No — simple listing without pagination for now (small catalogs). |
| Soft-delete cascade for plano_permissao? | Yes — cascade soft-delete: when a plan is soft-deleted, all its plano_permissao records are also soft-deleted within the same transaction. |
| EntidadeBase on plano_permissao? | Yes — keep EntidadeBase for consistency with project conventions (audit trail on junction table). |
| GET /produtos/:id response shape? | Include nested permissions in the product response. |
| GET /planos list response depth? | Include permissions only (no product nesting). Detail endpoint (GET /planos/:id) includes both product and permissions. |
| UUID generation function? | `gen_random_uuid()` — matches existing codebase pattern, built-in since PG 13, no extension needed. |

## 3.1 Technical Design Baseline (Required)

### Database Design

#### Tables and Actions

| Schema | Table | Action | Purpose |
| ------ | ----- | ------ | ------- |
| public | `produto` | Create | Platform product catalog (e.g., Satie) |
| public | `permissao` | Create | Product-scoped permission/capability catalog |
| public | `plano` | Create | Subscription plan tier for a specific product |
| public | `plano_permissao` | Create | Junction table linking plans to permissions |

#### Columns Specification

**Table: `produto`**

| Campo | Tipo SQL | Nullable | Default | PK | FK Ref | Unique | Index | Notes |
| ----- | -------- | -------- | ------- | -- | ------ | ------ | ----- | ----- |
| `id_produto` | UUID | No | `gen_random_uuid()` | Yes | N/A | Yes | PK | Primary key |
| `descricao` | VARCHAR(255) | No | — | No | N/A | No | — | Product description |
| `codigo_produto` | VARCHAR(100) | No | — | No | N/A | Yes | `idx_produto_codigo_produto` | Unique product code |
| `criado_por` | VARCHAR(255) | No | — | No | N/A | No | — | EntidadeBase |
| `modificado_por` | VARCHAR(255) | No | — | No | N/A | No | — | EntidadeBase |
| `criado_as` | TIMESTAMP | No | `NOW()` | No | N/A | No | — | EntidadeBase |
| `modificado_as` | TIMESTAMP | No | `NOW()` | No | N/A | No | — | EntidadeBase |
| `deletado_as` | TIMESTAMP | Yes | NULL | No | N/A | No | — | EntidadeBase |
| `deletado_por` | VARCHAR(255) | Yes | NULL | No | N/A | No | — | EntidadeBase |

**Table: `permissao`**

| Campo | Tipo SQL | Nullable | Default | PK | FK Ref | Unique | Index | Notes |
| ----- | -------- | -------- | ------- | -- | ------ | ------ | ----- | ----- |
| `id_permissao` | UUID | No | `gen_random_uuid()` | Yes | N/A | Yes | PK | Primary key |
| `codigo_permissao` | VARCHAR(100) | No | — | No | N/A | Yes | `idx_permissao_codigo_permissao` | Unique permission code |
| `id_produto` | UUID | No | — | No | `produto(id_produto)` | No | `idx_permissao_id_produto` | FK to product |
| `descricao` | VARCHAR(255) | No | — | No | N/A | No | — | Permission description |
| `criado_por` | VARCHAR(255) | No | — | No | N/A | No | — | EntidadeBase |
| `modificado_por` | VARCHAR(255) | No | — | No | N/A | No | — | EntidadeBase |
| `criado_as` | TIMESTAMP | No | `NOW()` | No | N/A | No | — | EntidadeBase |
| `modificado_as` | TIMESTAMP | No | `NOW()` | No | N/A | No | — | EntidadeBase |
| `deletado_as` | TIMESTAMP | Yes | NULL | No | N/A | No | — | EntidadeBase |
| `deletado_por` | VARCHAR(255) | Yes | NULL | No | N/A | No | — | EntidadeBase |

**Table: `plano`**

| Campo | Tipo SQL | Nullable | Default | PK | FK Ref | Unique | Index | Notes |
| ----- | -------- | -------- | ------- | -- | ------ | ------ | ----- | ----- |
| `id_plano` | UUID | No | `gen_random_uuid()` | Yes | N/A | Yes | PK | Primary key |
| `nome` | VARCHAR(150) | No | — | No | N/A | No | — | Plan name |
| `descricao` | TEXT | Yes | NULL | No | N/A | No | — | Plan description |
| `cor` | VARCHAR(7) | No | — | No | N/A | No | — | Hex color (#RRGGBB) |
| `ativo` | BOOLEAN | No | `true` | No | N/A | No | — | Active flag |
| `id_produto` | UUID | No | — | No | `produto(id_produto)` | No | `idx_plano_id_produto` | FK to product (immutable after creation) |
| `criado_por` | VARCHAR(255) | No | — | No | N/A | No | — | EntidadeBase |
| `modificado_por` | VARCHAR(255) | No | — | No | N/A | No | — | EntidadeBase |
| `criado_as` | TIMESTAMP | No | `NOW()` | No | N/A | No | — | EntidadeBase |
| `modificado_as` | TIMESTAMP | No | `NOW()` | No | N/A | No | — | EntidadeBase |
| `deletado_as` | TIMESTAMP | Yes | NULL | No | N/A | No | — | EntidadeBase |
| `deletado_por` | VARCHAR(255) | Yes | NULL | No | N/A | No | — | EntidadeBase |

**Table: `plano_permissao`**

| Campo | Tipo SQL | Nullable | Default | PK | FK Ref | Unique | Index | Notes |
| ----- | -------- | -------- | ------- | -- | ------ | ------ | ----- | ----- |
| `id_plano_permissao` | UUID | No | `gen_random_uuid()` | Yes | N/A | Yes | PK | Primary key |
| `id_plano` | UUID | No | — | No | `plano(id_plano)` | No | `idx_plano_permissao_id_plano` | FK to plan |
| `id_permissao` | UUID | No | — | No | `permissao(id_permissao)` | No | `idx_plano_permissao_id_permissao` | FK to permission |
| `criado_por` | VARCHAR(255) | No | — | No | N/A | No | — | EntidadeBase |
| `modificado_por` | VARCHAR(255) | No | — | No | N/A | No | — | EntidadeBase |
| `criado_as` | TIMESTAMP | No | `NOW()` | No | N/A | No | — | EntidadeBase |
| `modificado_as` | TIMESTAMP | No | `NOW()` | No | N/A | No | — | EntidadeBase |
| `deletado_as` | TIMESTAMP | Yes | NULL | No | N/A | No | — | EntidadeBase |
| `deletado_por` | VARCHAR(255) | Yes | NULL | No | N/A | No | — | EntidadeBase |

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

#### Migration Strategy

- **Forward migration**: Single migration file creates all 4 tables in dependency order: `produto` → `permissao` → `plano` → `plano_permissao`. Foreign keys and indexes are created with each table.
- **Rollback strategy**: Drop tables in reverse order: `plano_permissao` → `plano` → `permissao` → `produto`.
- **Backfill strategy**: N/A — tables are created empty. Seed data (actual products/permissions) will be a separate future task.
- **Compatibility window**: No existing code references these tables. Migration is purely additive.

### API and Integration Contracts

#### Products API (read-only)

**GET /api/produtos** — List all products

| Field | Type | Notes |
|-------|------|-------|
| Response `200` | `ProductResponseDto[]` | Array of products |

**ProductResponseDto:**

| Field | Type | Nullable | Source |
|-------|------|----------|--------|
| `idProduto` | string (UUID) | No | `produto.id_produto` |
| `codigoProduto` | string | No | `produto.codigo_produto` |
| `descricao` | string | No | `produto.descricao` |
| `criadoAs` | Date | No | `produto.criado_as` |
| `modificadoAs` | Date | No | `produto.modificado_as` |

**GET /api/produtos/:id** — Get product by ID with permissions

| Field | Type | Notes |
|-------|------|-------|
| Response `200` | `ProductDetailResponseDto` | Product with nested permissions |
| Response `404` | Error | Product not found |

**ProductDetailResponseDto** (extends ProductResponseDto):

| Field | Type | Nullable | Source |
|-------|------|----------|--------|
| `idProduto` | string (UUID) | No | `produto.id_produto` |
| `codigoProduto` | string | No | `produto.codigo_produto` |
| `descricao` | string | No | `produto.descricao` |
| `criadoAs` | Date | No | `produto.criado_as` |
| `modificadoAs` | Date | No | `produto.modificado_as` |
| `permissoes` | `PermissionResponseDto[]` | No | Related permissions |

**GET /api/produtos/:idProduto/permissoes** — List permissions for a product

| Field | Type | Notes |
|-------|------|-------|
| Response `200` | `PermissionResponseDto[]` | Array of permissions for the product |
| Response `404` | Error | Product not found |

**PermissionResponseDto:**

| Field | Type | Nullable | Source |
|-------|------|----------|--------|
| `idPermissao` | string (UUID) | No | `permissao.id_permissao` |
| `codigoPermissao` | string | No | `permissao.codigo_permissao` |
| `descricao` | string | No | `permissao.descricao` |
| `criadoAs` | Date | No | `permissao.criado_as` |
| `modificadoAs` | Date | No | `permissao.modificado_as` |

#### Plans API (CRUD)

**POST /api/planos** — Create a plan with product and permissions

Request body — `CreatePlanDto`:

| Field | Type | Required | Validation | Notes |
|-------|------|----------|------------|-------|
| `nome` | string | Yes | `@IsString()`, `@IsNotEmpty()`, `@MaxLength(150)` | Plan name |
| `descricao` | string | No | `@IsOptional()`, `@IsString()` | Plan description |
| `cor` | string | Yes | `@IsString()`, `@Matches(/^#[0-9A-Fa-f]{6}$/)` | 7-char hex color |
| `ativo` | boolean | No | `@IsOptional()`, `@IsBoolean()` | Defaults to `true` |
| `idProduto` | string (UUID) | Yes | `@IsUUID()` | FK to product |
| `permissoes` | string[] (UUIDs) | Yes | `@IsArray()`, `@IsUUID("4", { each: true })`, `@ArrayNotEmpty()` | Permission UUIDs |

| Response | Type | Notes |
|----------|------|-------|
| `201` | `PlanDetailResponseDto` | Created plan with product and permissions |
| `400` | Error | Validation error (invalid color, missing fields, cross-product permission violation) |
| `404` | Error | Product not found |

**GET /api/planos** — List all plans with permissions

| Response | Type | Notes |
|----------|------|-------|
| `200` | `PlanListResponseDto[]` | Plans with permissions (no product nesting) |

**PlanListResponseDto:**

| Field | Type | Nullable | Source |
|-------|------|----------|--------|
| `idPlano` | string (UUID) | No | `plano.id_plano` |
| `nome` | string | No | `plano.nome` |
| `descricao` | string | Yes | `plano.descricao` |
| `cor` | string | No | `plano.cor` |
| `ativo` | boolean | No | `plano.ativo` |
| `idProduto` | string (UUID) | No | `plano.id_produto` |
| `criadoAs` | Date | No | `plano.criado_as` |
| `modificadoAs` | Date | No | `plano.modificado_as` |
| `permissoes` | `PermissionResponseDto[]` | No | Related permissions via plano_permissao |

**GET /api/planos/:id** — Get plan detail with product and permissions

| Response | Type | Notes |
|----------|------|-------|
| `200` | `PlanDetailResponseDto` | Plan with nested product and permissions |
| `404` | Error | Plan not found |

**PlanDetailResponseDto:**

| Field | Type | Nullable | Source |
|-------|------|----------|--------|
| `idPlano` | string (UUID) | No | `plano.id_plano` |
| `nome` | string | No | `plano.nome` |
| `descricao` | string | Yes | `plano.descricao` |
| `cor` | string | No | `plano.cor` |
| `ativo` | boolean | No | `plano.ativo` |
| `criadoAs` | Date | No | `plano.criado_as` |
| `modificadoAs` | Date | No | `plano.modificado_as` |
| `produto` | `ProductResponseDto` | No | Related product |
| `permissoes` | `PermissionResponseDto[]` | No | Related permissions via plano_permissao |

**PATCH /api/planos/:id** — Update a plan (product is immutable)

Request body — `UpdatePlanDto`:

| Field | Type | Required | Validation | Notes |
|-------|------|----------|------------|-------|
| `nome` | string | No | `@IsOptional()`, `@IsString()`, `@IsNotEmpty()`, `@MaxLength(150)` | Plan name |
| `descricao` | string | No | `@IsOptional()`, `@IsString()` | Plan description |
| `cor` | string | No | `@IsOptional()`, `@Matches(/^#[0-9A-Fa-f]{6}$/)` | 7-char hex color |
| `ativo` | boolean | No | `@IsOptional()`, `@IsBoolean()` | Active flag |
| `permissoes` | string[] (UUIDs) | No | `@IsOptional()`, `@IsArray()`, `@IsUUID("4", { each: true })` | When provided, replaces all current permissions |

| Response | Type | Notes |
|----------|------|-------|
| `200` | `PlanDetailResponseDto` | Updated plan with product and permissions |
| `400` | Error | Validation error (invalid color, cross-product permission violation) |
| `404` | Error | Plan not found |

**Note on UpdatePlanDto**: This is NOT `PartialType(CreatePlanDto)` because `idProduto` must be excluded entirely (immutable after creation) and `permissoes` has different validation semantics (optional on update, can be empty array to clear all permissions). This will be a standalone DTO class.

**DELETE /api/planos/:id** — Soft-delete a plan

| Response | Type | Notes |
|----------|------|-------|
| `204` | void | Plan and its plano_permissao records soft-deleted |
| `404` | Error | Plan not found |

#### Error Contracts

| Status | Condition | Error Message Pattern |
|--------|-----------|----------------------|
| `400` | Invalid color format | "A cor deve ser um código hexadecimal de 7 caracteres (ex: #FF5733)" |
| `400` | Missing required fields | Standard class-validator messages |
| `400` | Cross-product permission violation | "As permissões informadas não pertencem ao produto do plano" |
| `401` | Missing/invalid JWT | Standard JwtAuthGuard response |
| `404` | Product not found | "Produto não encontrado" |
| `404` | Plan not found | "Plano não encontrado" |

### Module Architecture

```
apps/admin/backend/src/
├── subscription/
│   ├── subscription.module.ts           # Domain boundary — imports Products + Plans modules
│   ├── products/
│   │   ├── products.module.ts           # TypeOrmModule.forFeature([Produto, Permissao])
│   │   ├── products.controller.ts       # GET /produtos, GET /produtos/:id, GET /produtos/:idProduto/permissoes
│   │   ├── products.service.ts          # Read-only operations
│   │   ├── entities/
│   │   │   ├── produto.entity.ts        # Produto entity
│   │   │   └── permissao.entity.ts      # Permissao entity
│   │   └── dto/
│   │       ├── product-response.dto.ts          # ProductResponseDto (list item)
│   │       ├── product-detail-response.dto.ts   # ProductDetailResponseDto (with permissions)
│   │       └── permission-response.dto.ts       # PermissionResponseDto
│   └── plans/
│       ├── plans.module.ts              # TypeOrmModule.forFeature([Plano, PlanoPermissao]), imports ProductsModule
│       ├── plans.controller.ts          # POST/GET/PATCH/DELETE /planos
│       ├── plans.service.ts             # CRUD with atomic permission management
│       ├── entities/
│       │   ├── plano.entity.ts          # Plano entity
│       │   └── plano-permissao.entity.ts # PlanoPermissao entity
│       └── dto/
│           ├── create-plan.dto.ts       # CreatePlanDto (rich payload)
│           ├── update-plan.dto.ts       # UpdatePlanDto (standalone, excludes idProduto)
│           ├── plan-list-response.dto.ts    # PlanListResponseDto (with permissions, no product)
│           └── plan-detail-response.dto.ts  # PlanDetailResponseDto (with product + permissions)
├── app/
│   └── app.module.ts                    # Add SubscriptionModule import + new entities to TypeORM config
├── config/
│   └── typeorm.config.ts                # Add new entities to DataSource entity array
└── migrations/
    └── <timestamp>-CreateSubscriptionTables.ts  # All 4 tables in one migration
```

### TypeORM Entity Design

**Produto** (`apps/admin/backend/src/subscription/products/entities/produto.entity.ts`):

- `@Entity("produto")`
- Extends `EntidadeBase`
- `@PrimaryGeneratedColumn("uuid") idProduto: string`
- `@Column({ type: "varchar", length: 255 }) descricao: string`
- `@Column({ type: "varchar", length: 100, unique: true }) codigoProduto: string`
- `@OneToMany(() => Permissao, (p) => p.produto) permissoes: Permissao[]`
- `@OneToMany(() => Plano, (p) => p.produto) planos: Plano[]`

**Permissao** (`apps/admin/backend/src/subscription/products/entities/permissao.entity.ts`):

- `@Entity("permissao")`
- Extends `EntidadeBase`
- `@PrimaryGeneratedColumn("uuid") idPermissao: string`
- `@Column({ type: "varchar", length: 100, unique: true }) codigoPermissao: string`
- `@Column({ type: "uuid" }) idProduto: string`
- `@Column({ type: "varchar", length: 255 }) descricao: string`
- `@ManyToOne(() => Produto, (p) => p.permissoes) @JoinColumn({ name: "id_produto" }) produto: Produto`

**Plano** (`apps/admin/backend/src/subscription/plans/entities/plano.entity.ts`):

- `@Entity("plano")`
- Extends `EntidadeBase`
- `@PrimaryGeneratedColumn("uuid") idPlano: string`
- `@Column({ type: "varchar", length: 150 }) nome: string`
- `@Column({ type: "text", nullable: true }) descricao: string | null`
- `@Column({ type: "varchar", length: 7 }) cor: string`
- `@Column({ type: "boolean", default: true }) ativo: boolean`
- `@Column({ type: "uuid" }) idProduto: string`
- `@ManyToOne(() => Produto, (p) => p.planos) @JoinColumn({ name: "id_produto" }) produto: Produto`
- `@OneToMany(() => PlanoPermissao, (pp) => pp.plano) planoPermissoes: PlanoPermissao[]`

**PlanoPermissao** (`apps/admin/backend/src/subscription/plans/entities/plano-permissao.entity.ts`):

- `@Entity("plano_permissao")`
- `@Unique(["idPlano", "idPermissao"])`
- Extends `EntidadeBase`
- `@PrimaryGeneratedColumn("uuid") idPlanoPermissao: string`
- `@Column({ type: "uuid" }) idPlano: string`
- `@Column({ type: "uuid" }) idPermissao: string`
- `@ManyToOne(() => Plano, (p) => p.planoPermissoes) @JoinColumn({ name: "id_plano" }) plano: Plano`
- `@ManyToOne(() => Permissao) @JoinColumn({ name: "id_permissao" }) permissao: Permissao`

### Key Implementation Patterns

#### Transaction Management

The Plans service uses `DataSource.transaction()` for atomic operations involving multiple entities:

- **Create plan**: Create `Plano` + create all `PlanoPermissao` records in one transaction.
- **Update plan (with permissions)**: Soft-delete existing `PlanoPermissao` records + create new ones + save plan in one transaction.
- **Delete plan**: Soft-delete all `PlanoPermissao` records + soft-delete `Plano` in one transaction.

#### Cross-Product Permission Validation

Before creating or updating a plan's permissions, the service validates that ALL provided permission UUIDs belong to the plan's product:

1. Query `Permissao` where `idProduto = plan.idProduto AND idPermissao IN (requestedIds)`.
2. Compare count of found permissions vs. count of requested IDs.
3. If mismatch → throw `BadRequestException("As permissões informadas não pertencem ao produto do plano")`.

#### Permission Replacement Strategy

On plan update, when `permissoes` is provided:

1. Soft-delete ALL existing `PlanoPermissao` for the plan (set `deletadoPor` + `softRemove`).
2. Create NEW `PlanoPermissao` records for the new permission set.
3. This creates a clean audit trail of permission changes over time.
4. **Trade-off**: Accumulates soft-deleted junction records. Acceptable for current catalog sizes.

#### Cascade Soft-Delete

On plan deletion:

1. Find all `PlanoPermissao` records for the plan (non-deleted).
2. Set `deletadoPor` on each, then `softRemove()` each.
3. Set `deletadoPor` on the plan, then `softRemove()` the plan.
4. All within a single transaction.

#### Guard Reuse

All subscription controllers apply `@UseGuards(JwtAuthGuard)` and `@ApiBearerAuth()` at the class level, reusing the guard from the Identity domain. The `AuthModule` exports `JwtAuthGuard` implicitly through `JwtStrategy`, and the `PlansModule`/`ProductsModule` do NOT need to import `AuthModule` directly — the guard works via Passport's global strategy registration.

## 4. Planning Gates (Step 2)

- [x] Feature spec status is `Approved`.
- [x] Feature scope is explicit and bounded.
- [x] Required contracts are defined or referenced.
- [x] Acceptance criteria are testable.
- [x] Dependencies are completed or properly sequenced.
- [x] Risks and edge cases are reviewed.
- [x] Technical design baseline is complete for all impacted layers.

### Engineering Gates

- [x] KISS-first approach is documented for major slices.
- [x] Pattern choice aligns with ADR baseline or deviation is justified.
- [x] Comment strategy is documented where comments are truly needed.
- [x] Testability is confirmed for slice boundaries.
- [x] Complexity hotspots are identified with mitigation.

**KISS notes**: Follow existing Identity domain patterns exactly — same module structure, controller decorators, service injection, DTO patterns, response mapping. No new patterns introduced. Transaction management with `DataSource.transaction()` is the simplest approach for multi-entity atomicity.

**Complexity hotspot**: Plans service handles cross-product validation + atomic permission replacement + cascade soft-delete. Mitigation: each operation is isolated into a clear method, and transactions wrap the multi-step logic.

## 5. Scope-to-Execution Mapping

### Slice 1 — Database Entities and Migration

- **Goal**: Create all 4 TypeORM entities and the database migration. Register entities in app module and TypeORM CLI config.
- **Files to create**:
  - `apps/admin/backend/src/subscription/products/entities/produto.entity.ts`
  - `apps/admin/backend/src/subscription/products/entities/permissao.entity.ts`
  - `apps/admin/backend/src/subscription/plans/entities/plano.entity.ts`
  - `apps/admin/backend/src/subscription/plans/entities/plano-permissao.entity.ts`
  - `apps/admin/backend/src/migrations/<timestamp>-CreateSubscriptionTables.ts`
- **Files to modify**:
  - `apps/admin/backend/src/app/app.module.ts` — add entities to `createTypeOrmConfig()`
  - `apps/admin/backend/src/config/typeorm.config.ts` — add entities to DataSource
- **Requirements covered**: FR-011, FR-012 (entity-level), FR-013, FR-014, FR-015
- **Acceptance covered**: SC-001
- **Expected outputs**: 4 entity files, 1 migration file, 2 modified config files

### Slice 2 — Products and Permissions Read-Only API

- **Goal**: Implement read-only endpoints for products and permissions, including the subscription module boundary, products module, controller, service, and response DTOs. Unit tests for the products service.
- **Files to create**:
  - `apps/admin/backend/src/subscription/subscription.module.ts`
  - `apps/admin/backend/src/subscription/products/products.module.ts`
  - `apps/admin/backend/src/subscription/products/products.controller.ts`
  - `apps/admin/backend/src/subscription/products/products.service.ts`
  - `apps/admin/backend/src/subscription/products/dto/product-response.dto.ts`
  - `apps/admin/backend/src/subscription/products/dto/product-detail-response.dto.ts`
  - `apps/admin/backend/src/subscription/products/dto/permission-response.dto.ts`
  - `apps/admin/backend-tests/src/unit/products.service.spec.ts`
- **Files to modify**:
  - `apps/admin/backend/src/app/app.module.ts` — import `SubscriptionModule`
- **Requirements covered**: FR-001, FR-002, FR-010
- **Acceptance covered**: AC S1.1, AC S1.2
- **Expected outputs**: Module + controller + service + 3 DTOs + unit tests

### Slice 3 — Plans CRUD API with Permission Management

- **Goal**: Implement full CRUD for plans with rich create/update payloads, cross-product permission validation, cascade soft-delete, and atomic transactions. Unit tests for the plans service.
- **Files to create**:
  - `apps/admin/backend/src/subscription/plans/plans.module.ts`
  - `apps/admin/backend/src/subscription/plans/plans.controller.ts`
  - `apps/admin/backend/src/subscription/plans/plans.service.ts`
  - `apps/admin/backend/src/subscription/plans/dto/create-plan.dto.ts`
  - `apps/admin/backend/src/subscription/plans/dto/update-plan.dto.ts`
  - `apps/admin/backend/src/subscription/plans/dto/plan-list-response.dto.ts`
  - `apps/admin/backend/src/subscription/plans/dto/plan-detail-response.dto.ts`
  - `apps/admin/backend-tests/src/unit/plans.service.spec.ts`
- **Files to modify**:
  - `apps/admin/backend/src/subscription/subscription.module.ts` — import `PlansModule`
- **Requirements covered**: FR-003, FR-004, FR-005, FR-006, FR-007, FR-008, FR-010, FR-012
- **Acceptance covered**: AC S2.1–S2.4, AC S3.1–S3.3, AC S4.1, AC S5.1
- **Expected outputs**: Module + controller + service + 4 DTOs + unit tests

### Slice 4 — Integration Tests

- **Goal**: End-to-end API testing for all subscription endpoints, covering all acceptance scenarios from the feature spec.
- **Files to create**:
  - `apps/admin/backend-tests/src/backend/produtos.spec.ts`
  - `apps/admin/backend-tests/src/backend/planos.spec.ts`
- **Requirements covered**: All FRs (validation through integration)
- **Acceptance covered**: All ACs (S1.1–S5.1)
- **Expected outputs**: 2 integration test files covering all acceptance scenarios
- **Prerequisites**: Seed data must be inserted via test setup (beforeAll) since no seed migration exists for products/permissions. Tests will insert test products and permissions directly into the database or via migration-like setup queries.

**Note on integration test data**: Since products and permissions are catalog data with no create API, integration tests need to seed test data before running. Two approaches:
1. **Direct database insertion** in `beforeAll` (using a separate DataSource or TypeORM repository).
2. **Temporary seed migration** for testing purposes.

Approach 1 (direct insertion) is recommended — it keeps test data isolated and doesn't pollute the migration pipeline. The existing integration test pattern uses axios against a running server, so tests will need a setup step that inserts test data via raw SQL or a shared test helper.

## 6. Task Generation Matrix

| Task ID | Title | Type | Plan Slice | Parallelizable | Dependencies | Requirement Refs | Technical Scope | Output File |
| ------- | ----- | ---- | ---------- | -------------- | ------------ | ---------------- | --------------- | ----------- |
| T001 | Database entities and migration | Task | Slice 1 | No | N/A | FR-011, FR-013, FR-014, FR-015, SC-001 | 4 entities, 1 migration, config updates | `docs/specs/features/003-admin-subscription-domain/tasks/T001-database-entities-and-migration.task.spec.md` |
| T002 | Products and permissions read API | Task | Slice 2 | No | T001 | FR-001, FR-002, FR-010, SC-002, AC S1.1, S1.2 | Module + controller + service + DTOs + unit tests | `docs/specs/features/003-admin-subscription-domain/tasks/T002-products-permissions-read-api.task.spec.md` |
| T003 | Plans CRUD with permission management | Task | Slice 3 | No | T001, T002 | FR-003–FR-008, FR-010, FR-012, SC-003–SC-006, AC S2.1–S5.1 | Module + controller + service + DTOs + unit tests | `docs/specs/features/003-admin-subscription-domain/tasks/T003-plans-crud-permission-management.task.spec.md` |
| T004 | Integration tests | Task | Slice 4 | No | T001, T002, T003 | All FRs, SC-007, SC-008, all ACs | 2 integration test files + test data seeding | `docs/specs/features/003-admin-subscription-domain/tasks/T004-integration-tests.task.spec.md` |

## 7. Task File Generation Rules

- Create one file per task under `docs/specs/features/003-admin-subscription-domain/tasks/`.
- Use naming pattern: `TXXX-<short-title>.task.spec.md`.
- Use `docs/specs/templates/task.spec.md` for every task file.
- Each task must reference the same feature spec and this feature plan.
- Start each task with status `Draft`; set to `Ready` after task-level approval.
- If a task cannot be delivered safely in one branch cycle, split it before approval.
- For data-impact tasks, include explicit table, field, SQL type, nullability, default, PK/FK, constraints, indexes, and migration notes.
- For API/UI-impact tasks, include explicit request/response/UI state contracts.

## 8. Repository Impact

```text
apps/
  admin/
    backend/
      src/
        subscription/           # NEW — entire module tree
        app/app.module.ts       # MODIFIED — import SubscriptionModule + entities
        config/typeorm.config.ts # MODIFIED — add entities
        migrations/             # NEW — subscription tables migration
    backend-tests/
      src/
        unit/                   # NEW — products.service.spec.ts, plans.service.spec.ts
        backend/                # NEW — produtos.spec.ts, planos.spec.ts
docs/
  specs/
    apps/admin/architecture.md  # MODIFIED — add Subscription domain entry
```

### Planned Changes by Area

- **Backend**: New `subscription/` module tree with products (read-only) and plans (CRUD) submodules. Each submodule has module, controller, service, entities, and DTOs. App module updated with new entities and domain module import.
- **Database**: 1 migration creating 4 tables with FK relationships, unique constraints, and indexes.
- **QA**: 2 unit test files (products service, plans service) + 2 integration test files (products API, plans API).
- **Docs**: Admin architecture spec updated with Subscription domain entry.

## 9. Validation Strategy

- **Unit tests**:
  - Products service: `findAll()`, `findOne()`, `findPermissionsByProduct()` with mocked repositories. Test not-found cases.
  - Plans service: `create()`, `findAll()`, `findOne()`, `update()`, `remove()` with mocked repositories and DataSource. Test cross-product permission validation, cascade soft-delete, transaction behavior, not-found cases.

- **Integration tests**:
  - Products API (`produtos.spec.ts`): List products, get product by ID (with permissions), get product permissions, 404 for non-existent product, 401 without JWT.
  - Plans API (`planos.spec.ts`): Create plan (with product + permissions), list plans (with permissions), get plan detail (with product + permissions), update plan (with permission replacement), deactivate plan (preserves permissions), cross-product permission rejection, non-existent product rejection, invalid color rejection, soft-delete (plan + cascade to plano_permissao), 404 for non-existent plan, 401 without JWT.

- **Non-functional checks**: Biome compliance on all changed files (`pnpm nx run-many -t check`).

## 10. Rollout and Safety

- **Feature flags**: N/A — backend-only addition of new endpoints.
- **Backward compatibility**: Fully additive — no existing endpoints or tables are modified. New module, new tables, new routes.
- **Monitoring/observability**: Standard NestJS logging. No additional monitoring needed at this stage.
- **Rollback plan**: Drop the 4 new tables via migration rollback. Remove `SubscriptionModule` import from `AppModule`. No data loss risk (new tables only).

## 11. Step 2 Completion Checklist

- [x] Plan is approved.
- [x] Technical design baseline is complete for all impacted layers.
- [x] Scope-to-execution mapping covers all feature requirements.
- [x] Task generation matrix is defined and consistent with scope.
- [x] Dependencies between planned slices are explicit.
- [x] Feature spec status is updated to `Planned`.
- [x] Plan is ready to hand off to Step 3 (Task Breakdown).

## 12. Documentation Impact

### Admin Architecture Spec

Update `docs/specs/apps/admin/architecture.md`:
- Add **Subscription** domain entry under "## Domains" section with reference to domain spec and feature spec.
- Add module hierarchy entries for `subscription/` under "### Module Hierarchy".
- No new ADRs needed — all patterns follow existing ADR baseline.

This documentation update will be included as part of T002 (the first task that creates the module structure and subscription module boundary).

## 13. Post-Implementation Feedback

Complete after feature delivery to sharpen future planning.

### What worked in planning

- [To be filled after feature completion]

### What caused friction

- [To be filled after feature completion]

### Planning standard updates

- [To be filled after feature completion]
