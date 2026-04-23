# ADR-006: Dependency Management Policy Baseline

## Status

Accepted

## Context

Different repositories manage dependencies in different ways. Some centralize versions at the root, some use per-module manifests, and some rely on build-tool-native mechanisms such as version catalogs or BOMs.

The kit should not force one dependency strategy, but it must require that the strategy be explicit.

## Decision

Each consuming project must define one documented dependency management policy during bootstrap.

That policy should answer:

- where dependency versions are declared
- how shared/internal packages or modules are referenced
- how upgrades are coordinated
- how drift is prevented or detected
- which package manager or build tool owns dependency resolution

The kit baseline is to prefer **one clear source of truth per repository**, even if the actual mechanism differs by stack.

## Consequences

- The kit stays compatible with JavaScript, Java, and other ecosystems.
- Dependency ownership becomes auditable.
- Prompts should consult the documented project policy instead of assuming root-level JavaScript workspaces.