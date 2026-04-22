# ADR-010: Scalar as Standard API Documentation UI

## Status

Deprecated

## Context

This ADR captures a framework-specific documentation UI choice from a reference project.

Keeping this kind of decision in the core ADR set makes the kit less reusable across different stacks and languages.

## Decision

API documentation UI choices must live in the owning app or service architecture docs or app-level ADRs, not in the core kit ADR set.

This file is retained only as a historical reference example of the type of decision that should be scoped locally.

## Consequences

- Core ADRs stay reusable across different stacks.
- Framework-specific API documentation decisions become easier to scope correctly.
- Existing projects that rely on this choice should move it to an app-level decision if it remains active.
