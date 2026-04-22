# Project Spec

## Overview

This repository is a reusable Spec-Driven Development workspace. It can be instantiated as a single-product repository, a multi-application monorepo, or a mixed-language platform with shared documentation and governance.

The purpose of this file is to define the stable project baseline: what the workspace is for, how delivery decisions are made, and which documentation layers are expected to exist.

## Problem

Many projects start implementation before they establish a durable way to capture requirements, architecture decisions, validation criteria, and delivery learnings. That usually leads to:

- implicit requirements and rework during implementation
- architecture choices that are hard to trace later
- inconsistent task quality and test expectations
- stack-specific assumptions leaking into the whole workspace

## Goals

- **Spec first**: every meaningful feature or architectural change starts from a documented spec.
- **Explicit decisions**: long-lived technical choices are captured as ADRs.
- **Configurable foundations**: the same kit should work for different languages, frameworks, and repo shapes.
- **Strong execution gates**: implementation only starts from approved, implementation-ready task specs.
- **Continuous sharpening**: retrospectives feed improvements back into templates, instructions, and ADRs.

## Repository Modes

This kit supports multiple repository shapes. The active mode must be documented during initialization.

### Single Project

One application or service with one primary codebase.

### Monorepo

Multiple apps, services, packages, or libraries sharing tooling and governance.

### Mixed-Language Workspace

Different surfaces may use different languages or frameworks, as long as validation expectations are explicit per surface.

## High-Level Architecture

```text
repository/
	docs/
		architecture.md
		project.spec.md
		decisions/
		specs/
			apps/<app>/
			domains/
			features/<feature-id>/
			templates/
	source/
		apps/ or services/ or packages/ or modules/
```

The exact code layout is project-specific and should be documented in `docs/architecture.md` and in the relevant app or service architecture docs.

## Applications, Services, and Domains

- Applications or services are the deployable runtime surfaces.
- Domains are optional but recommended when the project benefits from bounded contexts or explicit business boundaries.
- Domain specs live in `docs/specs/domains/` when the project uses them.
- App or service architecture docs live in `docs/specs/apps/<app>/architecture.md`.

## Default Preferences

These are defaults that may be overridden by project-specific decisions:

- Prefer a monorepo when multiple deployable surfaces or shared libraries are expected.
- Prefer Nx when monorepo orchestration, dependency graphs, and unified task execution will add value.
- Prefer Biome where it is a good fit.
- For unsupported languages or ecosystems, document the replacement quality tool in ADR-003 or an app-level decision.

## Engineering Principles

These principles apply unless a newer ADR narrows or replaces them:

- prefer the simplest design that solves the real requirement
- favor self-descriptive code over explanatory comments
- evolve architecture incrementally based on delivery pressure
- keep boundaries clear enough for focused validation
- make cross-cutting decisions explicit instead of implicit

## Delivery Model

The project uses an iterative SDD model:

1. Draft and approve a feature spec.
2. Turn the approved scope into a technical plan.
3. Break the plan into implementation-ready task specs.
4. Implement one approved task at a time with validation evidence.
5. Run a short retrospective and sharpen the kit.

## References

- Operating rules: `.github/copilot-instructions.md`
- Architecture overview: `docs/architecture.md`
- ADRs: `docs/decisions/`
- Feature specs: `docs/specs/features/`
- App or service architecture docs: `docs/specs/apps/`
- Domain specs: `docs/specs/domains/`