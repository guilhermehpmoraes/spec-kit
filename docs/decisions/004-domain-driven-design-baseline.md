# ADR-004: Solution Boundary Documentation Baseline

## Status

Accepted

## Context

Some projects using this kit will organize work around DDD bounded contexts. Others will use modules, capabilities, services, or product areas. The kit needs a way to document solution boundaries without forcing one architecture style.

## Decision

The kit supports **boundary-oriented documentation** with optional domain specs.

### Baseline Rules

- A project may document boundaries as domains, modules, capability areas, services, or another clear unit.
- `docs/specs/domains/` is available for this purpose but is optional.
- Feature specs may reference a `Domain/Area` when that adds clarity.
- Per-app or per-surface architecture specs are optional and should be added only when repo-wide docs are not enough.

### If the project uses DDD

- Domain specs represent bounded contexts.
- Feature specs should reference the relevant domain.

### If the project does not use DDD

- The same directory can document modules, services, or capability areas instead.

## Consequences

- The kit remains compatible with DDD-oriented and non-DDD repositories.
- Teams still have a consistent place to document boundary ownership.
- Projects must be explicit during bootstrap about which boundary model they actually use.