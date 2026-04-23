# Project Spec

## Overview

This repository provides a reusable **Spec Kit** for teams adopting **Spec Driven Development (SDD)**.

The kit is meant to be copied into new or existing repositories and then adapted during an initial bootstrap flow. Its purpose is to give every project a stable foundation for:

- documenting product and technical intent before implementation
- turning approved specs into implementation plans and task specs
- keeping architectural decisions explicit through ADRs
- evolving process rules based on delivery feedback

The kit is intentionally **stack-agnostic**. It should support monorepos and single-repository setups, backend-only systems, frontend-only systems, full-stack applications, libraries, services, and multi-application platforms.

## Problem

Projects often start coding before three foundational questions are answered clearly:

- what is being built and why now
- how scope, architecture, and implementation work should be documented
- how delivery decisions and learnings should be preserved for future work

Without a reusable baseline, every new repository rebuilds its delivery process from scratch. That creates inconsistent specs, weak traceability, missing architectural rationale, and avoidable rework.

## Goals

- **Standardize SDD adoption** with a repeatable workflow for specs, plans, tasks, implementation, and retrospectives.
- **Keep setup flexible** so the same kit works for different stacks, architectures, package managers, and repository layouts.
- **Capture project context early** through a guided bootstrap/init flow that collects stack, topology, conventions, and delivery rules.
- **Preserve architectural intent** via ADRs and foundational documentation.
- **Improve over time** by feeding delivery learnings back into templates, prompts, instructions, and decisions.

## Non-Goals

- Enforcing a single application stack, framework, or language.
- Requiring a monorepo, Nx, pnpm, or any specific toolchain.
- Defining a universal domain model for all projects.
- Replacing project-specific architecture, setup, or operational documentation.

## High-Level Architecture

The kit is organized around a small set of reusable documentation and automation surfaces.

```text
docs/
	project.spec.md                Project-level context and SDD baseline
	architecture.md                Repository/workspace architecture baseline
	decisions/                     ADRs and foundational decisions
	specs/
		templates/                   Reusable templates for feature, plan, and task specs
		features/                    Generated feature work packages
		domains/                     Optional domain or bounded-context specs
		apps/                        Optional per-application architecture specs

.github/
	copilot-instructions.md        Agent behavior and project rules
	prompts/                       Step-oriented SDD prompts
	skills/                        Reusable operational skills
	agents/                        Optional supporting agents
```

The concrete code layout of a consuming repository may vary. Common patterns include:

- monorepo with `apps/`, `packages/`, `libs/`, or `services/`
- polyrepo or single-application repository with `src/`
- backend and frontend split repositories
- platform repositories with multiple deployable applications

## Operating Model

The kit assumes a five-step SDD lifecycle:

1. Draft the feature spec.
2. Produce the feature plan.
3. Break the plan into task specs.
4. Implement approved tasks.
5. Finish the feature with a retrospective and process sharpening.

The exact branch strategy, validation commands, and architecture conventions are configured per project during bootstrap.

## Project Bootstrap Principle

Before regular feature work begins, the consuming repository should run a one-time **SDD bootstrap/init flow** that captures:

- project name and purpose
- repository topology and layout
- stacks, frameworks, and key libraries
- package manager and task runner
- architectural conventions
- testing and quality gates
- branching and release conventions
- naming rules and documentation boundaries

The result of that bootstrap is persisted into the repository's foundational docs and instructions. After the bootstrap is complete, the bootstrap-only prompt or instruction should be removed or retired so the repository continues with the normal SDD flow only.

## Engineering Principles

These principles apply to the kit unless a project-specific ADR overrides them.

- **KISS first**: prefer the simplest process and structure that fully supports delivery.
- **Explicit decisions**: important technical or workflow choices should be documented.
- **Self-sufficient artifacts**: specs, plans, and tasks should minimize hidden context.
- **Validation-driven delivery**: plans and tasks must define how behavior will be checked.
- **Iterative standardization**: adopt standards from proven delivery patterns, not speculation.

Baseline decisions are captured in `docs/decisions/`.

## Delivery Model

The kit evolves through use:

1. Start from the base templates and instructions.
2. Adapt them during project bootstrap.
3. Deliver features through the SDD workflow.
4. Capture what worked, what failed, and what should become standard.
5. Update the kit deliberately so the next project starts from a stronger baseline.

## References

- Agent instructions: `/.github/copilot-instructions.md`
- Architecture baseline: `docs/architecture.md`
- ADRs: `docs/decisions/`
- Templates: `docs/specs/templates/`
- Feature work packages: `docs/specs/features/`
- Optional domain specs: `docs/specs/domains/`
- Optional application specs: `docs/specs/apps/`