# Task Spec: Docker Compose for Local Dev

- **Feature ID**: 001-admin-identity-domain
- **Task ID**: T001
- **Status**: Done
- **Type**: Task
- **Parallelizable**: Yes
- **Parallelization Notes**: No dependency on any other task. Safe to run in parallel with T000 and T002.
- **Date**: 2026-04-15
- **Owner**: TBD
- **Feature Folder**: docs/specs/features/001-admin-identity-domain/
- **Feature Spec**: docs/specs/features/001-admin-identity-domain/feature.spec.md
- **Feature Plan**: docs/specs/features/001-admin-identity-domain/plan.spec.md
- **Task File**: docs/specs/features/001-admin-identity-domain/tasks/T001-docker-compose.task.spec.md
- **Related ADRs**: N/A
- **Dependencies**: N/A

## 1. Purpose

Provide local PostgreSQL 17.4 and Redis services via Docker Compose so developers can run the backend with a real database and cache. This is an infrastructure prerequisite for all database and auth tasks.

## 2. Scope

### In Scope

- Create `docker-compose.yml` at the repository root with PostgreSQL 17.4 and Redis (latest)
- Create `.env.example` at the repository root documenting all environment variables
- Create `.env` entry in `.gitignore` (if not already present)
- Named volumes for data persistence across container restarts

### Out of Scope

- Application-level database configuration (handled in T003)
- Redis client configuration in code (handled in T002)
- CI/CD Docker configuration

## 3. Context from Feature Plan

- **Plan slice**: Slice 1 — Docker Compose for Local Dev
- **Requirement refs**: FR-020
- **Affected Paths**:
  - `docker-compose.yml` (new, repo root)
  - `.env.example` (new, repo root)
  - `.gitignore` (modify if needed)
- **Why this task exists**: PostgreSQL and Redis must be available locally before any database-dependent code can be developed or tested.

## 4. Technical Specification (Required)

### 4.1 Backend and API Contracts (when applicable)

N/A — infrastructure only.

### 4.2 Database Specification (mandatory when data impact exists)

N/A — this task creates the database server container, not the schema.

### 4.3 Frontend and UX Contracts (when applicable)

N/A.

### 4.4 Cross-Cutting Constraints

- **Compatibility constraints**: Docker Compose V2 syntax (no `version` key needed for modern Docker Compose).
- **Security**: Default credentials are for local development only. `.env` must be in `.gitignore`.

## 5. Implementation Steps

1. **Create `docker-compose.yml`** at the repo root:

```yaml
services:
  postgres:
    image: postgres:17.4
    ports:
      - "5432:5432"
    environment:
      POSTGRES_USER: ${DATABASE_USER:-satie}
      POSTGRES_PASSWORD: ${DATABASE_PASSWORD:-satie}
      POSTGRES_DB: ${DATABASE_NAME:-satie_dev}
    volumes:
      - satie_pgdata:/var/lib/postgresql/data

  redis:
    image: redis:latest
    ports:
      - "6379:6379"
    volumes:
      - satie_redisdata:/data

volumes:
  satie_pgdata:
  satie_redisdata:
```

2. **Create `.env.example`** at the repo root:

```env
# Database (PostgreSQL)
DATABASE_HOST=localhost
DATABASE_PORT=5432
DATABASE_USER=satie
DATABASE_PASSWORD=satie
DATABASE_NAME=satie_dev

# Redis
REDIS_HOST=localhost
REDIS_PORT=6379

# JWT
JWT_SECRET=change-me-in-production
JWT_ACCESS_EXPIRY=15m
JWT_REFRESH_EXPIRY=7d

# Admin Seed
ADMIN_SEED_EMAIL=admin@satie.local
ADMIN_SEED_PASSWORD=Admin@123
```

3. **Update `.gitignore`**: Add `.env` entry if not already present (to prevent committing real credentials).

4. **Verify**: Run `docker compose up -d` and confirm both services start. Test PostgreSQL connectivity with `docker compose exec postgres psql -U satie -d satie_dev -c "SELECT 1"`. Test Redis with `docker compose exec redis redis-cli ping`.

## 6. Acceptance Criteria

- AC1: `docker-compose.yml` exists at the repo root with PostgreSQL 17.4 and Redis services.
- AC2: `docker compose up -d` starts both containers without errors.
- AC3: PostgreSQL is accessible on `localhost:5432` with credentials from `.env.example` defaults.
- AC4: Redis is accessible on `localhost:6379`.
- AC5: Named volumes `satie_pgdata` and `satie_redisdata` persist data across container restarts.
- AC6: `.env.example` documents all environment variables from the plan.
- AC7: `.env` is in `.gitignore`.

## 7. Test Scenarios

- **Scenario 1 (happy path)**: `docker compose up -d` → both containers running → `docker compose exec postgres psql -U satie -d satie_dev -c "SELECT 1"` returns `1`.
- **Scenario 2 (happy path)**: `docker compose exec redis redis-cli ping` returns `PONG`.
- **Scenario 3 (persistence)**: `docker compose down && docker compose up -d` → data in PostgreSQL persists.
- **Scenario 4 (clean slate)**: `docker compose down -v` removes volumes for a fresh start.

## 8. Definition of Ready (to start Step 4)

- [x] Status is `Draft` (will be `Ready` after approval).
- [x] Scope is explicit and bounded.
- [x] Required contracts are defined — Docker Compose service specs documented.
- [x] Technical specification is detailed enough for independent implementation.
- [x] Acceptance criteria are testable.
- [x] Open questions are resolved.
- [x] Dependencies are completed or clearly sequenced — no dependencies.

## 9. Definition of Done

- [x] Status moved to `Done`.
- [x] Acceptance criteria validated.
- [x] Edge cases covered.
- [x] Links to changed files/PR/tests are registered.
- [x] Feature spec and plan traceability remains intact.

### Changed Files

- `docker-compose.yml` (created)
- `.env.example` (created)
- `.gitignore` (modified — added `.env` entry)

## 10. Jira Mapping (Optional)

- **Issue Type**: Task
- **Summary**: Docker Compose for local dev (PostgreSQL + Redis)
- **Description**: Create docker-compose.yml with PostgreSQL 17.4 and Redis for local development.
- **Priority**: Highest
- **Labels**: Infrastructure
- **Custom field (Tipo)**: Feature
- **Custom field (Tamanho)**: 1 Dia

## 11. Status Log

| Date       | Status | Notes |
| ---------- | ------ | ----- |
| 2026-04-15 | Draft  | Task created from approved feature plan |
| 2026-04-15 | Ready  | Task approved for implementation |
| 2026-04-15 | In Progress | Implementation started |
| 2026-04-15 | Done | All acceptance criteria validated — containers start, PostgreSQL and Redis accessible, volumes persist, .env in .gitignore |
