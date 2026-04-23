# ADR-010: Interface Documentation Standard

## Status

Accepted

## Context

Many projects expose interfaces that need durable documentation: HTTP APIs, events, commands, SDKs, CLIs, or internal integration contracts. The original kit hardcoded one API documentation tool from one stack, which made reuse difficult.

## Decision

Projects using this kit must choose and document a standard way to describe externally consumed interfaces.

Possible approaches include:

- OpenAPI or generated API reference UIs
- AsyncAPI or event contract docs
- SDK/reference documentation
- Markdown-based interface references
- CLI help and command reference generation

Whatever mechanism is chosen, the project should document:

- the source of truth for the interface definition
- the publication or access surface
- the required metadata quality for documented interfaces

## Consequences

- The kit remains useful beyond HTTP/NestJS projects.
- Interface documentation becomes an explicit architecture concern.
- Consuming projects should add stack-specific ADRs if they standardize on a specific tool.