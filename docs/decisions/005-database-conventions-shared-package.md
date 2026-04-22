# ADR-005: Persistence Conventions and Shared Data Foundations

## Status

Accepted

## Context

Some projects using this kit will have multiple applications or services that share persistence patterns. Without a shared baseline, each codebase can independently redefine:

- audit fields and soft delete behavior
- naming conventions for schema or persistent objects
- ORM, query, migration, or repository setup patterns

This leads to inconsistency, duplicated boilerplate, and divergent conventions across applications.

Additionally, different projects may choose different naming languages and casing rules for source code and persistence objects. Those choices must be explicit rather than implicit.

## Decision

### Shared Data Foundation

- If multiple applications or services share persistence conventions, create a shared package or module for common persistence foundations.
- That shared foundation may include base entities, audit traits, migration helpers, repository utilities, schema conventions, or connection setup.
- Application-specific entities, schemas, and migrations remain local unless deliberately shared.

### Audit And Deletion Strategy

Projects should decide explicitly whether audit fields and soft deletes are part of the baseline. When they are, use a shared foundation to keep the rules consistent.

Recommended baseline when auditability matters:

- created by
- modified by
- created at
- modified at
- deleted at
- deleted by

### Naming Conventions

- Persistence naming conventions must be documented explicitly.
- The language used for tables, collections, entities, documents, fields, or columns is project-specific.
- The casing strategy for persistence objects is also project-specific.
- Tool-specific naming strategies belong in the owning app or service architecture doc unless they are truly repo-wide.

### Tooling Scope

- ORM and migration choices are project-specific.
- If one ORM or migration tool is used across most projects that reuse this kit, promote that decision explicitly in a future ADR.

## Consequences

- Shared persistence behavior becomes easier to reuse and audit.
- Naming decisions stop being accidental and become part of the project contract.
- Different projects can choose different persistence tooling without breaking the kit.
- Projects that do not need shared persistence foundations can keep this ADR lightweight.
