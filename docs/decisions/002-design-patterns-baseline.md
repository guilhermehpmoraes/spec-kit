# ADR-002: Design Patterns Baseline

## Status

Accepted

## Context

Projects need guidance on pattern selection, but the kit must avoid prescribing framework-specific architectures that only fit one stack.

## Decision

The baseline is to prefer a **small, pragmatic set of patterns** that match the project's actual architecture and delivery needs.

### Default Guidance

- Use clear module, package, or service boundaries.
- Separate orchestration from core business rules when the codebase benefits from that split.
- Use adapters for external systems or unstable integration points.
- Prefer composition and small reusable units over inheritance-heavy hierarchies.
- Introduce advanced patterns only when complexity justifies them.

### Explicit Non-Goals

- No required framework pattern such as controller-service-repository, CQRS, or hook-based state management unless the project chooses it.
- No pattern adoption purely for theoretical purity.

## Consequences

- The kit stays useful across multiple stacks.
- Projects can document stack-specific patterns later through additional ADRs.
- Reviewers can challenge unnecessary complexity without blocking pragmatic architecture choices.