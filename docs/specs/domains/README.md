# Domain Specs

This directory is optional. Use it when the project benefits from explicit domain, bounded-context, capability-area, or module-level documentation.

Each spec is a Markdown file named `<domain-or-area>.md` and may document:

- responsibilities and boundaries
- core entities, aggregates, or business concepts
- invariants and rules
- owning product, surface, or application
- upstream and downstream integrations

Feature specs can reference these documents through the `Domain/Area` field.

If the project does not use domain-oriented documentation, this directory may stay empty.
