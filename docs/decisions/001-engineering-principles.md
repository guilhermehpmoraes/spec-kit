# ADR-001: Engineering Principles Baseline

## Status

Accepted

## Context

The kit needs a small set of engineering principles that remain useful across different languages, frameworks, and repository shapes.

## Decision

Every project adopting this kit starts with the following baseline principles unless a later ADR overrides them:

1. Prefer the simplest solution that fully meets the requirement.
2. Favor self-descriptive code and documentation over explanatory noise.
3. Keep boundaries explicit so behavior can be tested and changed safely.
4. Evolve architecture by evidence, not speculation.
5. Make validation part of delivery, not a final afterthought.

### Practical Rules

- Prefer straightforward control flow and naming before abstraction.
- Introduce indirection only when it solves a present problem.
- Keep specs, plans, tasks, and code aligned.
- Add comments only for non-obvious intent, tradeoffs, or constraints.

## Consequences

- Reviews have a stable quality baseline.
- The same core expectations apply across stacks.
- Some inherited project habits may need to be changed during bootstrap or later delivery.