# ADR-006: Single Version Dependency Policy

## Status

Accepted

## Context

Integrated monorepos often drift into inconsistent dependency management when individual projects declare overlapping third-party dependencies independently.

This caused:

- **Version drift** across workspace projects.
- **Tooling ambiguity** about where new dependencies belong.
- **Maintenance overhead** when multiple manifests need to stay aligned.

For integrated monorepos where applications share substantial tooling and are evolved together, a single version policy is a strong default.

## Decision

Adopt a **single version policy** for all third-party dependencies:

1. **Declare third-party dependencies in one documented workspace-level location** whenever the ecosystem supports it.
2. **Project-local manifests should avoid duplicating shared third-party dependencies** unless the package manager or build tool requires it.
3. **Workspace package references should use the native workspace linking model** of the chosen tool.
4. **Exceptions must be documented** when a subproject legitimately needs independent dependency ownership.

## Consequences

### Positive

- Single source of truth for all dependency versions — no drift or conflicts.
- Simpler onboarding with one primary install entry point.
- Easier upgrades: update a version once, all projects use it.
- Smaller dependency diffs on updates.

### Negative

- All teams must coordinate on dependency upgrades (acceptable for current team size).
- Adding a new dependency may require editing the workspace-level manifest or dependency file.

### Neutral

- This ADR is most applicable to integrated monorepos. Single-project repositories may not need it.

## References

- [Nx Dependency Management Strategies](https://nx.dev/docs/concepts/decisions/dependency-management)
