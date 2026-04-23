# ADR-013: Database Entity Lifecycle Baseline

## Status

Accepted

## Context

Projects that persist business data usually need repeatable lifecycle metadata such as who created, changed, or deleted a record and when those actions happened. Without an explicit baseline, each app or module invents its own field set, duplicates entity boilerplate, and mixes hard delete and soft delete strategies inconsistently.

This kit already asks projects to define naming conventions during bootstrap, but it does not yet establish a reusable baseline for entity lifecycle metadata or deletion strategy.

## Decision

When a consuming project uses a database or another persistent store with entity-style records, it should define an entity lifecycle baseline during bootstrap.

### Reusable base entity

- Prefer a reusable base entity or equivalent shared persistence abstraction in a shared package, library, or module when that abstraction can be reused across multiple apps or services.
- The shared abstraction should carry the standard lifecycle metadata fields used across the project so each app does not redefine them independently.

### Required lifecycle metadata

- The project baseline should include six lifecycle fields or their semantic equivalents:
  - created by
  - created at
  - modified by
  - modified at
  - deleted by
  - deleted at
- The exact field names must follow the project's documented naming and language conventions rather than hardcoded examples from the kit.
- If the chosen persistence technology needs an equivalent representation instead of inheritance, document the project-specific pattern explicitly.

### Deletion strategy baseline

- Prefer soft delete as the default deletion strategy for persistent entities.
- Plans and tasks that model database changes must state how soft delete is represented and queried.
- If any entity or storage object must use hard delete instead, that exception must be documented explicitly in the relevant spec, plan, task, or ADR.

## Consequences

- Projects get a stable audit and deletion baseline across apps and modules.
- Shared persistence code becomes easier to reuse because lifecycle metadata is standardized.
- Plans and tasks for database work must be explicit about audit fields, soft delete behavior, and justified exceptions.
- Naming remains project-defined, so the kit does not force Portuguese or English field names.