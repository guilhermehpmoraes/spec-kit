# Satie

Monorepo for the **Admin** and **Satie** platforms. Built with Nx, NestJS, React, PostgreSQL, and Redis.

## Prerequisites

- **Node.js** (see `.nvmrc` or `package.json` for version)
- **pnpm** 10.x+
- **Docker** and **Docker Compose** (V2)

## Getting Started

### 1. Install dependencies

```bash
pnpm install
```

### 2. Set up environment variables

```bash
cp .env.example .env
```

Edit `.env` as needed. The defaults work for local development with Docker Compose.

| Variable | Default | Description |
|----------|---------|-------------|
| `DATABASE_HOST` | `localhost` | PostgreSQL host |
| `DATABASE_PORT` | `5432` | PostgreSQL port |
| `DATABASE_USER` | `satie` | PostgreSQL user |
| `DATABASE_PASSWORD` | `satie` | PostgreSQL password |
| `DATABASE_NAME` | `satie_dev` | PostgreSQL database |
| `REDIS_HOST` | `localhost` | Redis host |
| `REDIS_PORT` | `6379` | Redis port |
| `JWT_SECRET` | — | Secret key for JWT signing (required) |
| `JWT_ACCESS_EXPIRY` | `15m` | Access token lifetime |
| `JWT_REFRESH_EXPIRY` | `7d` | Refresh token lifetime |
| `ADMIN_SEED_EMAIL` | `admin@satie.local` | Super-admin seed email |
| `ADMIN_SEED_PASSWORD` | — | Super-admin seed password (required for seed migration) |
| `APP_ENVIRONMENT` | `dev` | Environment label used to derive master username and database name prefixes for educational groups (`dev` or `prod`) |

### 3. Start infrastructure (PostgreSQL + Redis)

```bash
docker compose up -d
```

Services:

| Service | Image | Port |
|---------|-------|------|
| PostgreSQL | `postgres:17.4` | `5432` |
| Redis | `redis:latest` | `6379` |

Data persists via named volumes (`satie_pgdata`, `satie_redisdata`). To reset:

```bash
docker compose down -v
```

### 4. Run database migrations

```bash
pnpm exec typeorm migration:run -d apps/admin/backend/src/config/typeorm.config.ts
```

This creates all domain tables:
- **Identity**: `usuario_admin` (+ super-admin seed)
- **Subscription**: `produto`, `permissao`, `plano`, `plano_permissao`
- **Educational Group**: `grupo_educacional`, `usuario_mestre`, `parametros_grupo_educacional`, `usuario_comum`, `assinatura`

### 5. Start the Admin backend

```bash
pnpm nx serve backend
```

The API runs at `http://localhost:3000/api`. API docs (Scalar) at `http://localhost:3000/api/docs`.

## Running Tests

```bash
# Backend unit tests
pnpm nx run backend-tests:test

# Backend integration/e2e tests (requires Docker services running)
pnpm nx run backend-tests:e2e

# All checks (Biome lint + format)
pnpm nx run-many -t check
```

## Project Structure

```
apps/
  admin/
    backend/           NestJS API (Identity, Subscription, and Educational Group domains)
    backend-tests/     Jest + Supertest tests
    frontend/          Vite + React UI
    frontend-tests/    Playwright tests
packages/
  database/            @satie/database — shared base entity, TypeORM config, Redis module
  database-tests/      Unit tests for @satie/database
  utils/               @satie/utils — slug generation, random password, environment naming helpers
  utils-tests/         Unit tests for @satie/utils
docs/
  architecture.md      Repo-wide architecture
  project.spec.md      High-level project spec
  specs/               Feature, plan, task, domain, and app specs
  decisions/           ADRs (architectural decision records)
```

## Workflow (SDD)

This project follows **Spec Driven Development**. Every feature starts from a spec before code is written. The full workflow rules live in `.github/copilot-instructions.md`, and templates in `docs/specs/templates/`.

### Delivery Flow

The SDD flow has 5 steps, each driven by a Copilot prompt:

| Step | Prompt | Input | Output |
|------|--------|-------|--------|
| 1 — Feature Spec | `/feature` | Feature description (text) or existing `feature-id` to refine | `feature.spec.md` in `Draft` |
| 2 — Feature Plan | `/plan <feature-id>` | Approved feature spec | `plan.spec.md` with technical details, DB schemas, API contracts |
| 3 — Task Breakdown | `/tasks <feature-id>` | Approved plan | One `T<NNN>-*.task.spec.md` per task, each with full implementation detail |
| 4 — Implementation | `/implement <feature-id> <task-id>` | Ready task spec | Production code, tests, task status → `Done` |
| 5 — Retrospective | `/finish <feature-id>` | All tasks `Done` | Retrospective findings, process improvements applied |

### How to Use

1. **Start a feature**: run `/feature` with a description of the problem/requirement. Review and approve the generated spec.
2. **Plan**: run `/plan <feature-id>`. Answer technical questions. Review and approve the plan.
3. **Break into tasks**: run `/tasks <feature-id>`. Confirm the proposed task list. Review generated task files.
4. **Implement**: run `/implement <feature-id> <task-id>` for each task (in dependency order). Each task produces code + tests and offers to commit.
5. **Close the feature**: run `/finish <feature-id>` after all tasks are done. Answer retrospective questions and approve process improvements.

### Artifact Structure

```
docs/specs/features/<feature-id>/
  feature.spec.md
  plan.spec.md
  tasks/
    T001-<short-title>.task.spec.md
    T002-<short-title>.task.spec.md
    ...
```

### Rules

- **No production code** in Steps 1–3 (spec/plan/tasks only).
- Tasks must have enough technical detail for any dev to implement without extra context.
- A task cannot be marked `Done` unless all tests pass and Biome reports zero issues.
- When Steps 1–2 run in plan mode, materialize approved artifacts in a separate write-enabled chat before implementing.

## Conventions

- **Database** (tables, columns, constraints, indexes): Portuguese
- **TypeORM entity classes and properties**: Portuguese (ADR-005)
- **Source code** (services, controllers, DTOs, modules, guards): English
- **Docs, specs, ADRs**: English
- **Commits**: Conventional Commits + Gitmoji
