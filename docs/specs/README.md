# Specs Directory Guide

This directory contains both reusable templates and project-specific spec artifacts.

## Stable Structure

- `templates/` holds the canonical templates used by the kit.
- `apps/` holds app or service architecture docs.
- `domains/` holds optional domain specs.
- `features/` holds feature packages with feature, plan, and task specs.

## During Initialization

When adapting this kit to a new project:

- keep `templates/`
- create or update only the app, domain, and feature docs that actually belong to the new project
- archive or remove stale reference material from previous projects

## Reference Material

If this repository contains app, domain, or feature specs from an older project, treat them as examples unless they are explicitly part of the active project.