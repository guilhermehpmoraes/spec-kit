# Project Init Spec: [Project Name]

- **Status**: Draft | Approved | Materialized
- **Created**: [YYYY-MM-DD]
- **Last Updated**: [YYYY-MM-DD]
- **Owner**: [Auto-filled from `git config user.name`]
- **Project Name**: [Repository or product name]
- **Repository Shape**: [Nx monorepo | monorepo without Nx | multi-repo | other]
- **Architecture Mode**: [Keep baseline | Adapt baseline | Replace baseline]
- **Primary Applications / Products**: [List or N/A]
- **Related ADRs**: [ADR-00X or N/A]

## 1. Project Summary

Describe what the project is, who it serves, and why it exists.

## 2. Stack Profile

Record the chosen stack by layer.

| Layer | Selected Stack | Notes |
| ----- | -------------- | ----- |
| Backend | [Java / NestJS / Go / N/A] | [details] |
| Frontend | [Angular / React / Vue / N/A] | [details] |
| Mobile | [Flutter / React Native / N/A] | [details] |
| Data | [PostgreSQL / MySQL / MongoDB / Redis / N/A] | [details] |
| Testing | [Jest / Vitest / JUnit / Playwright / Cypress / N/A] | [details] |
| CI/CD | [GitHub Actions / AWS / Jenkins / N/A] | [details] |

## 3. Repository and Architecture Choices

- **Nx baseline**: [Keep | Adapt | Do not use]
- **Repo layout notes**: [apps/, packages/, services/, modules/, etc.]
- **Architecture notes**: [layers, modules, domains, services, apps]
- **Shared package strategy**: [UI, contracts, utils, infrastructure, or N/A]
- **Naming conventions**: [code, database, docs, branches, packages]

## 4. Frontend Baseline (fill when applicable)

- **Frontend exists**: Yes | No
- **Mobile first**: [rules or N/A]
- **Shared component library**: [location and ownership or N/A]
- **Reusable/dynamic component rules**: [how components should be configured]
- **Global CSS/theme layer**: [tokens, colors, spacing, typography]
- **Pencil component catalog**: [`.pen` file path or planned location]
- **Pencil planning rule**: [how UI features must be validated during Step 2]

## 5. Materialization Outputs

List the files that project init must create or update.

- `docs/project.spec.md`
- `docs/architecture.md`
- `docs/specs/apps/<app>/architecture.md` when needed
- [Any additional baseline docs]
- [Any Pencil files or design folders]

## 6. Init Acceptance Checklist

- [ ] Project summary is explicit.
- [ ] Stack profile is complete enough to plan future features without guessing.
- [ ] Repository shape is defined.
- [ ] Architecture mode is defined.
- [ ] Shared package strategy is defined.
- [ ] Frontend baseline is documented when UI work exists.
- [ ] Pencil usage is documented when frontend/UI work exists.
- [ ] Materialization outputs are listed.