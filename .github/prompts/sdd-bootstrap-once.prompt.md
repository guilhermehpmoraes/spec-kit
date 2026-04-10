---
mode: ask
model: GPT-5
description: "One-shot SDD bootstrap for a new project; creates initial spec context and then deletes this prompt file"
---

# One-Shot SDD Bootstrap (Self-Destruct)

You are initializing SDD in a fresh project using this spec-kit as baseline.

## Inputs to collect from the user

1. Project name and one-paragraph product overview.
2. Repository model (Nx monorepo, single repo, or other).
3. Stack profile:
- Backend
- Frontend
- Database/data stores
- Test stack
- Package/build manager
- CI/CD provider
4. Naming conventions for:
- Source code
- Database
- Docs/specs/ADRs
5. Initial domains expected for the first release.

## Required actions

1. Update `docs/specs/project.spec.md` with the project-specific context and stack profile.
2. Update `docs/architecture.md` with project context and architectural boundaries.
3. Keep `.github/copilot-instructions.md` aligned with the selected stack profile.
4. Do not implement production code during bootstrap.

## Validation checklist

- Project context is filled with concrete information.
- Stack profile is explicit (no TBD values left unless justified).
- Naming conventions are documented.
- At least one initial domain is listed.

## Self-destruct step (mandatory)

After finishing and showing a summary of changes, delete this file:

- `.github/prompts/sdd-bootstrap-once.prompt.md`

If deletion fails, explicitly warn the user to remove it manually to avoid unnecessary context load in future chats.
