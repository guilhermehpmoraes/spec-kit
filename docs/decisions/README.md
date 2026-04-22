# ADR Scope Guide

Use this folder for decisions that should apply across the whole workspace or that represent durable defaults of the kit.

## Keep In `docs/decisions/`

- SDD workflow and branching rules
- engineering principles
- default design pattern baselines
- workspace-wide quality tooling strategy
- repo-wide dependency strategy
- documentation governance

## Move To `docs/specs/apps/<app>/decisions/`

- framework-specific backend choices
- frontend framework choices
- API documentation UI choices
- ORM, migration, or schema tooling choices that apply to only one app
- deployment details that belong to one service or application

## Promotion Rule

Promote a decision from app-level to repo-wide only when you expect it to hold across most future projects that reuse this kit.