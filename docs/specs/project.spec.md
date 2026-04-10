# Project Spec — Satie

## Overview

Satie is a centralized data platform for schools. It enables organized visualization of school structure and delivers insights through dashboards and reports.

## Problem

Schools deal with fragmented data across multiple systems and spreadsheets. There is no single source of truth for understanding the school's organizational structure, student performance, or operational metrics. Decision-makers lack timely, actionable insights.

## Goals

- **Centralize** school data into a single, reliable platform.
- **Visualize** the organizational structure of schools (classes, grades, teachers, students, etc.) in an intuitive way.
- **Deliver insights** through dashboards and reports that support data-driven decisions.

## High-Level Architecture

```
┌─────────────────────────────────────────────┐
│                 Nx Monorepo                  │
│                                              │
│  apps/                                       │
│  ├── <domain>/                               │
│  │   ├── backend/     (NestJS)               │
│  │   └── frontend/    (Vite + React)         │
│  │                                           │
│  packages/                                   │
│  ├── shared libs, UI components, utils       │
│                                              │
│  Database: PostgreSQL                        │
│  CI: AWS-based                               │
└─────────────────────────────────────────────┘
```

## Core Domains

> _To be defined — this section will be expanded as domain specs are created in `docs/specs/domains/`._

## Non-Functional Requirements

> _To be defined — performance targets, scalability, security, accessibility, etc._

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

Satie adopts an iterative SDD model:

1. Create or update the feature spec.
2. Implement and validate against acceptance criteria.
3. Run a short retrospective focused on: what to keep, what to avoid, and what to standardize.
4. Incorporate the learning into stable specs/templates/decisions for future work.

This model intentionally evolves standards over time based on real delivery outcomes.

## References

- Stack details: see [copilot-instructions.md](../../.github/copilot-instructions.md)
- Feature specs: `docs/specs/features/`
- Domain specs: `docs/specs/domains/`
- Architecture overview: [docs/architecture.md](../architecture.md)
- Architectural decisions: `docs/decisions/`
