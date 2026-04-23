# ADR-008: Repository and Project Naming Conventions

## Status

Accepted

## Context

Repositories with multiple applications, services, modules, packages, or deployable surfaces need predictable names. Without that, commands, docs, specs, and automation become ambiguous.

## Decision

Each consuming project must define naming conventions for repository-level artifacts during bootstrap.

The documented convention should cover, when applicable:

- applications or services
- packages or libraries
- modules or bounded contexts
- test projects or suites
- build, release, and automation identifiers

The kit baseline is:

- names should clearly indicate purpose and ownership
- naming should scale as the repository grows
- the same concept should not have multiple names across docs, code, and automation unless documented

## Consequences

- The kit no longer assumes Nx project names, npm scopes, or one application naming pattern.
- Branch, commit, and prompt flows can derive scope from project-defined names instead of sample values.
- Bootstrap must capture the repository's naming rules explicitly.