# ADR-000: Use Architectural Decision Records

## Status

Accepted

## Context

Projects adopting this Spec Kit need a durable way to record why long-lived technical and process decisions were made. Without explicit decision records, future contributors lose the rationale behind architecture, tooling, branching, naming, and documentation rules.

## Decision

We use Architectural Decision Records (ADRs) to document significant technical and workflow decisions.

### Format

- Each ADR is a Markdown file in `docs/decisions/`.
- Files are numbered sequentially: `NNN-short-title.md`.
- ADRs describe the problem, the chosen decision, and the resulting consequences.

### Minimum Structure

```markdown
# ADR-NNN: Title

## Status

Proposed | Accepted | Deprecated | Superseded by ADR-XXX

## Context

Why is this decision needed?

## Decision

What was chosen?

## Consequences

What gets easier, harder, or more constrained because of this?
```

## Consequences

- Architectural and workflow decisions become traceable.
- Specs, prompts, and code can reference stable decision numbers.
- Future projects can reuse the kit baseline and replace ADRs selectively during bootstrap.