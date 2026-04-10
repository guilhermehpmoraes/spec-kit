# Project Spec — [Project Name]

## Overview

[Project Name] is [one sentence describing what the product/platform is and who it serves].

## Problem

[Describe the core problem in business terms.]

## Goals

- **Goal 1**: [Primary business/technical goal]
- **Goal 2**: [Secondary goal]
- **Goal 3**: [Outcome or value goal]

## High-Level Architecture

```
┌─────────────────────────────────────────────┐
│             Repository Baseline             │
│                                             │
│  [monorepo or single-repo layout]          │
│  [service/app boundaries]                  │
│  [shared packages/modules]                 │
│                                             │
│  Data store(s): [database/cache/queues]    │
│  CI/CD: [provider/tooling baseline]        │
└─────────────────────────────────────────────┘
```

## Core Domains

> _To be defined — expand as domain specs are created in `docs/specs/domains/`._

## Non-Functional Requirements

> _To be defined — include performance, scalability, security, compliance, accessibility, and observability targets._

## Engineering Principles

These principles apply to all features and domains unless a specific ADR states otherwise.

- **KISS first**: prefer the simplest solution that fully meets the requirement.
- **Self-descriptive code**: prioritize clear naming and small cohesive units over explanatory comments.
- **Minimal comments**: use comments only for non-obvious intent, tradeoffs, or constraints that code alone cannot express.
- **Incremental design**: start with basic proven patterns and evolve when pressure is real.
- **Testable structure**: keep boundaries explicit so behavior can be validated with focused tests.

Baseline decisions are captured in:

- `docs/decisions/001-engineering-principles.md`
- `docs/decisions/002-design-patterns-baseline.md`
- `docs/decisions/003-biome-quality-tooling-baseline.md`

## Delivery Model (SDD Evolution)

This project adopts an iterative SDD model:

1. Create or update the feature spec.
2. Implement and validate against acceptance criteria.
3. Run a short retrospective focused on: what to keep, what to avoid, and what to standardize.
4. Incorporate the learning into stable specs/templates/decisions for future work.

This model intentionally evolves standards over time based on real delivery outcomes.

## References

- Agent instructions and working rules: [copilot-instructions.md](../../.github/copilot-instructions.md)
- Feature specs: `docs/specs/features/`
- Domain specs: `docs/specs/domains/`
- Architecture overview: [docs/architecture.md](../architecture.md)
- Architectural decisions: `docs/decisions/`

## Project-Specific Baselines

Fill this section when adopting the kit in a new project.

- **Domain/Product summary**: [one paragraph]
- **Repository model**: [Nx monorepo | single repo | other]
- **Stack profile**:
	- Backend: [stack]
	- Frontend: [stack]
	- Data: [stack]
	- Testing: [stack]
- **Naming conventions**:
	- Database: [language/style]
	- Source code: [language/style]
	- Docs/specs/ADRs: [language/style]
