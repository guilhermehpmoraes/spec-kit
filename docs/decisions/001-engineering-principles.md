# ADR-001: Engineering Principles Baseline

## Status

Accepted

## Context

The project needs a simple and explicit engineering baseline that is easy to apply from day one. The team wants to avoid accidental complexity, keep code easy to read, and standardize quality expectations while the platform is still being shaped.

## Decision

We adopt the following baseline engineering principles for all code in Satie.

1. KISS first.
2. Self-descriptive code over comments.
3. Comments only when intent is not obvious from code.
4. Evolve design incrementally, avoid premature abstraction.
5. Keep units cohesive and boundaries explicit for testability.

### Practical Rules

- Prefer straightforward control flow and data structures before introducing abstractions.
- Choose clear names for modules, functions, variables, and tests.
- Keep functions and classes focused on one responsibility.
- Introduce comments only for:
    - domain/business intent that is not obvious,
    - non-trivial tradeoffs,
    - temporary constraints or workarounds.
- Avoid comments that only restate the code.

### Review Checklist

- Is this the simplest implementation that satisfies the requirement?
- Is the code understandable without additional comments?
- Are responsibilities split clearly and testable in isolation?
- Is new abstraction justified by current requirements?

## Consequences

- Code review becomes more consistent and objective.
- New contributors can follow a clear quality baseline.
- Some refactoring may be required when older code violates the baseline.
- Architecture will evolve by evidence, not by upfront complexity.
