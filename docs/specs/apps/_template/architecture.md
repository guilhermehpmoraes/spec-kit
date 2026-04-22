# [App Or Service Name] Architecture

## Purpose

Describe the concrete architecture of this app or service: stack, main boundaries, runtime shape, validation approach, and local conventions.

## Context

- What this app or service does
- Who uses it
- Why it exists in the workspace

## Technology Profile

- Language(s): [TypeScript, Java, Go, Python, etc.]
- Framework(s): [NestJS, Spring Boot, React, Next.js, etc.]
- Runtime: [Node.js, JVM, browser, mobile, other]
- Data stores: [PostgreSQL, MySQL, MongoDB, Redis, S3, N/A]
- Test tooling: [Jest, Vitest, JUnit, Playwright, etc.]
- Quality tooling: [Biome, ESLint, Spotless, Checkstyle, ktlint, etc.]

## Source Layout

```text
[document the actual folders used by this app or service]
```

## Main Boundaries

- Modules, packages, layers, or bounded contexts
- How responsibilities are split
- What is shared and what stays local

## Contracts And Interfaces

- Public APIs, events, schemas, UI contracts, or integration points
- Validation and versioning rules

## Local Conventions

- Naming conventions
- Testing strategy
- Build and run strategy
- Observability expectations

## Related Decisions

- List app-specific ADRs here when needed