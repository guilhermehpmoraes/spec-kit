# ADR-011: Frontend Design System and Pencil Workflow Baseline

## Status

Accepted

## Context

Projects that ship user interfaces need a repeatable way to manage design artifacts, reusable UI code, tokens, and implementation flow. Without an explicit baseline, visual decisions drift into production code, prototypes become stale, and small component variations multiply into avoidable UI fragmentation.

This kit needs a frontend workflow that remains generic across stacks while still enforcing a strong design-first rule when a change has meaningful visual or UX impact.

## Decision

When a consuming project includes frontend work, it should define a shared frontend baseline during bootstrap.

### Baseline structure

- Keep design artifacts, prototypes, tokens, and related references under a top-level `design/` area or an equivalent project-documented location.
- Keep reusable frontend implementation code in shared packages, libraries, or modules rather than duplicating near-identical components inside feature code.
- Prefer highly reusable component APIs over multiple one-off variants with only minor visual differences.
- Define a global theme baseline covering items such as color roles, typography, spacing, and other shared tokens.
- Treat mobile-first behavior as the default responsive design posture unless the consuming project documents an explicit exception.

### Pencil as design source of truth

- Use Pencil for frontend or UI work when the feature changes visual behavior, interaction structure, or user-facing layout in a meaningful way.
- During planning, frontend or UI features with relevant visual impact must produce or update the Pencil prototype before plan approval.
- UI tasks should not transition to `Ready` unless the relevant Pencil prototype is approved and referenced in the plan or task artifact.
- If implementation uncovers a new visual or UX change that materially affects the approved prototype, update Pencil first and then replicate that change in code.
- Code must not become the source of truth for unresolved visual decisions.

### Scope guard

- Purely technical frontend tasks with no meaningful visual or UX impact may proceed without Pencil, as long as the task spec makes that explicit.

## Consequences

- Design and implementation stay synchronized for UI work.
- Task readiness becomes stricter for frontend changes that affect user experience.
- Projects need to maintain their design artifacts deliberately instead of treating them as disposable.
- The kit gains a frontend baseline without hardcoding a specific frontend framework.