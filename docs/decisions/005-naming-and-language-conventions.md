# ADR-005: Naming and Language Conventions Baseline

## Status

Accepted

## Context

Naming conventions vary widely across projects. Some teams use English everywhere, some keep data objects in a local language, and some need separate naming rules for code, schemas, interfaces, or documentation.

Without an explicit baseline, templates and prompts end up hardcoding conventions from one project into another.

## Decision

Each consuming project must define a naming matrix during bootstrap covering, when applicable:

- source code identifiers
- documentation language
- package, module, or service names
- data object names (tables, collections, topics, indexes, constraints)
- API and event naming

The kit baseline is:

- keep naming internally consistent
- prefer descriptive names over abbreviations
- document exceptions explicitly in ADRs or architecture docs
- keep specs, ADRs, prompts, and templates in one agreed documentation language for the project

## Consequences

- The kit no longer assumes Portuguese data names or English-only code rules.
- Projects can keep strong conventions, but those conventions must be made explicit.
- Prompts and templates should refer to the project's naming convention instead of hardcoded examples.