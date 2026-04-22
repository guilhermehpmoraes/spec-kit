# Domain specs

This directory contains domain specs following DDD bounded context principles.

Each domain spec is a Markdown file named `<domain-name>.md` that documents:

- The bounded context and its responsibilities
- Core entities, value objects, and aggregates
- Domain rules and invariants
- Which application the domain belongs to

Domain specs are referenced from feature specs via the `Domain` field.

See [ADR-004](../../decisions/004-domain-driven-design-baseline.md) for the domain organization decision.
