# ADR-005: Database Conventions and Shared Database Package

## Status

Accepted

## Context

The monorepo hosts multiple applications (Admin, Satie, and future ones), all sharing PostgreSQL as the primary database. Without a shared baseline, each application would independently define:

- Entity base classes (audit fields, soft deletes)
- Naming strategies (snake_case enforcement)
- TypeORM configuration patterns

This leads to inconsistency, duplicated boilerplate, and divergent conventions across applications.

Additionally, the project follows a Portuguese naming convention for database objects (tables, columns) while keeping source code in English for logic. Entity classes need a clear rule for naming to avoid constant translation friction between code and database.

## Decision

### Shared Database Package

- A shared package `@satie/database` lives at `packages/database/`.
- This package exports the base entity, TypeORM configuration utilities, and naming strategy setup.
- All applications import `@satie/database` for database foundations. Application-specific entities and migrations remain in each app.

### Auditable Base Entity (`EntidadeBase`)

All database entities MUST inherit from `EntidadeBase`, which provides:

| Field | Type | Description |
|-------|------|-------------|
| `criado_por` | VARCHAR(255), NOT NULL | Identifier of who created the record (user ID or `'system'`) |
| `modificado_por` | VARCHAR(255), NOT NULL | Identifier of who last modified the record |
| `criado_as` | TIMESTAMP, NOT NULL, DEFAULT NOW() | Creation timestamp (immutable after insert) |
| `modificado_as` | TIMESTAMP, NOT NULL, DEFAULT NOW() | Last modification timestamp (auto-updated) |
| `deletado_as` | TIMESTAMP, NULL | Soft delete marker (TypeORM `@DeleteDateColumn`) |
| `deletado_por` | VARCHAR(255), NULL | Identifier of who performed the soft delete |

### Soft Deletes

- All tables use soft deletes by default. Records are never physically deleted in normal operations.
- Soft delete is implemented via TypeORM's `@DeleteDateColumn` on `deletado_as`.
- The `deletado_por` field is set alongside `deletado_as` to track who performed the deletion.
- Queries automatically exclude soft-deleted records via TypeORM's built-in filtering.

### Audit Field Types

- `criado_por` and `modificado_por` use `VARCHAR(255)` (not FK) to allow flexible identifiers: user UUIDs, `'system'`, `'migration'`, `'seed'`, etc.
- This avoids circular FK dependencies during bootstrap and supports system-level operations.

### Snake Case Naming Strategy

- The package `typeorm-naming-strategies` with `SnakeNamingStrategy` is the standard naming strategy for all TypeORM connections.
- This ensures all table names and column names are automatically converted to `snake_case` in PostgreSQL, regardless of the casing used in TypeScript entity classes.
- The naming strategy is configured once in the shared package and applied at connection level.

### Entity Class Naming Convention

- TypeORM entity classes MUST be named in Portuguese to be consistent with database table and column names.
- Examples: `UsuarioAdmin`, `TokenRevogado`, `EntidadeBase`.
- Entity property names follow Portuguese as well: `idUsuarioAdmin`, `email`, `senha`, `criadoPor`, `modificadoPor`, etc.
- This eliminates the mental translation layer between entity code and database schema, since `typeorm-naming-strategies` converts `criadoPor` -> `criado_por` automatically.

### What Stays in English

- Service classes, controllers, DTOs, modules, guards, and all non-entity code remain in English.
- Variable names outside of entity definitions remain in English.
- Only entity classes and their properties use Portuguese names.

## Consequences

- All applications share a single source of truth for audit fields and soft deletes.
- New entities get audit + soft-delete behavior by inheriting `EntidadeBase` — no manual field setup.
- Snake case is enforced automatically; developers don't need to manually specify column names.
- Entity classes are immediately readable alongside their database schema.
- The `@satie/database` package becomes a foundational dependency for all backends.
- Adding new shared database utilities (e.g., custom TypeORM decorators, migration helpers) has a clear home.
- Portuguese entity naming is scoped strictly to TypeORM entities; the rest of the codebase remains in English.
