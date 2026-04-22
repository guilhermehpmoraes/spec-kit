# ADR-000: Use Architectural Decision Records

## Status

Accepted

## Context

As the project grows, architectural decisions need to be documented so that current and future contributors (including AI assistants) understand _why_ things are the way they are — not just _what_ was built.

## Decision

We will use Architecture Decision Records (ADRs) to capture significant architectural decisions.

### Format

- Each ADR is a Markdown file in `docs/decisions/`.
- Files are numbered sequentially: `NNN-short-title.md` (e.g., `001-use-nestjs-backend.md`).
- Each ADR follows the template below.

### Template

```markdown
# ADR-NNN: Title

## Status

Proposed | Accepted | Deprecated | Superseded by ADR-XXX

## Context

What is the issue that we're seeing that motivates this decision?

## Decision

What is the change that we're proposing and/or doing?

## Consequences

What becomes easier or more difficult to do because of this change?
```

### Statuses

- **Proposed** — Under discussion, not yet decided.
- **Accepted** — Decided and in effect.
- **Deprecated** — No longer relevant.
- **Superseded** — Replaced by a newer ADR (link to it).

## Consequences

- Every significant architectural choice has a traceable record.
- Specs and code can reference ADR numbers for context.
- New contributors can onboard faster by reading the decision log.
