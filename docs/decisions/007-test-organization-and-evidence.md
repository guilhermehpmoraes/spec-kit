# ADR-007: Test Organization and Evidence Baseline

## Status

Accepted

## Context

The kit already requires validation and test evidence in task specs, but the baseline needs an ADR that explains where tests should live, how they relate to implementation scopes, and why evidence is mandatory.

## Decision

Every project adopting this kit must define its test organization and validation evidence rules during bootstrap.

At minimum, the project should document:

- which test layers exist (unit, integration, e2e, contract, performance, etc.)
- where those tests live
- what environment dependencies they require
- which commands are authoritative for each test layer
- how test execution evidence is recorded for completed tasks

The task-spec `Test Evidence` section remains mandatory for implemented work.

## Consequences

- Completed work must show real validation evidence.
- Projects can choose colocated tests, dedicated test modules, or separate repositories, as long as the choice is explicit.
- Prompts and implementation flows can stay flexible without losing rigor.