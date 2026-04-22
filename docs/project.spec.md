# Project Spec

## Overview

This monorepo hosts multiple **applications** (platforms), each with its own backend and frontend. Applications are not domains — they are products that contain multiple domains organized following Domain-Driven Design principles (see ADR-004).

The first delivery is the **Admin** platform — an internal tool for managing clients, products, features, and usage metrics across all platforms. One of the products managed through Admin is **Satie**, a centralized data platform for schools.

## Problem

### Admin Platform

There is no unified tool to manage clients, activate products and features, or monitor platform usage and metrics. Operations are manual and fragmented, making it hard to onboard clients or understand system health.

### Satie (Educational Product)

Schools deal with fragmented data across multiple systems and spreadsheets. There is no single source of truth for understanding the school's organizational structure, student performance, or operational metrics. Decision-makers lack timely, actionable insights.

## Goals

### Admin Platform (initial focus)

- **Manage clients** — activate, configure, and monitor client accounts.
- **Manage products & features** — enable/disable products and feature flags per client.
- **Visualize usage** — dashboards for navigation patterns, system usage, and operational metrics.
- **Centralize operations** — single control plane for all managed platforms and products.

### Satie (educational product, managed by Admin)

- **Centralize** school data into a single, reliable platform.
- **Visualize** the organizational structure of schools (classes, grades, teachers, students, etc.) in an intuitive way.
- **Deliver insights** through dashboards and reports that support data-driven decisions.

## High-Level Architecture

```
┌──────────────────────────────────────────────────┐
│                   Nx Monorepo                     │
│                                                   │
│  apps/                                            │
│  ├── admin/                 (Application)         │
│  │   ├── backend/           NestJS                │
│  │   ├── frontend/          Vite + React          │
│  │   └── (domains organized internally)           │
│  │                                                │
│  ├── <application>/         (Application)         │
│  │   ├── backend/           NestJS                │
│  │   ├── frontend/          Vite + React          │
│  │   └── (domains organized internally)           │
│  │                                                │
│  packages/                                        │
│  ├── shared libs, UI components, contracts        │
│                                                   │
│  Database: PostgreSQL                             │
│  Cache/Ephemeral: Redis                            │
│  CI: AWS-based                                    │
└──────────────────────────────────────────────────┘
```

## Applications

| Application | Description | Status |
|-------------|-------------|--------|
| **Admin** | Internal administrative platform — client, product, and feature management; usage dashboards and metrics. | Active (initial focus) |
| **Satie** | Centralized data platform for schools — school structure visualization, dashboards, and reports. | Planned |

Each application has its own detailed architecture spec at `docs/specs/apps/<app>/architecture.md`.

## Domains

Domains follow DDD bounded context principles and are scoped to a single application. Each domain encapsulates a coherent problem area. Domain specs are documented in `docs/specs/domains/` and linked from feature specs for development context.

> _Initial domains will be formalized as features are specced. See ADR-004 for the domain organization decision._

### Persistence Layer

PostgreSQL is the primary system of record. All entities inherit from `EntidadeBase` (audit fields + soft deletes). TypeORM with `SnakeNamingStrategy` enforces snake_case naming. Entity classes use Portuguese names. Redis is available for ephemeral data (token revocation, caching). See ADR-005.

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
- `docs/decisions/004-domain-driven-design-baseline.md`
- `docs/decisions/005-database-conventions-shared-package.md`

## Delivery Model (SDD Evolution)

The project adopts an iterative SDD model:

1. Create or update the feature spec.
2. Implement and validate against acceptance criteria.
3. Run a short retrospective focused on: what to keep, what to avoid, and what to standardize.
4. Incorporate the learning into stable specs/templates/decisions for future work.

This model intentionally evolves standards over time based on real delivery outcomes.

## References

- Stack details: see [copilot-instructions.md](../.github/copilot-instructions.md)
- Feature specs: `docs/specs/features/`
- Domain specs: `docs/specs/domains/`
- Application specs: `docs/specs/apps/`
- Architecture overview: [docs/architecture.md](./architecture.md)
- Architectural decisions: `docs/decisions/`