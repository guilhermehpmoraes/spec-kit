# ADR-012: Nx App and Test Layout Baseline

## Status

Accepted

## Context

Some consuming projects will use Nx to organize full-stack applications. In those repositories, a predictable folder structure for app surfaces and tests reduces navigation cost, keeps backend and frontend boundaries aligned under the same application root, and avoids test placement drifting across projects.

The kit should not force Nx on every repository, but when a project adopts Nx with paired backend and frontend surfaces per app, the baseline layout should be explicit.

## Decision

If a consuming project uses Nx and organizes work by application, the default layout is:

```text
apps/
  <app>/
    backend/
    frontend/
```

### Backend test placement

- Unit tests stay colocated with the implementation files.
- Integration tests may also stay colocated when they validate a specific module or controller boundary.
- End-to-end backend tests live under the backend root test area.

Example:

```text
apps/<app>/backend/
  src/
    modules/
      users/
        users.service.ts
        users.service.spec.ts
        users.controller.ts
        users.controller.spec.ts
        users.integration.spec.ts
  test/
    e2e/
      users.e2e-spec.ts
    jest-e2e.json
```

### Frontend test placement

- Unit tests stay colocated with components and other local implementation units.
- Page or route integration tests may stay colocated with the page or feature they validate.
- Frontend end-to-end tests live under the frontend root e2e area.

Example:

```text
apps/<app>/frontend/
  src/
    components/
      UserCard/
        UserCard.tsx
        UserCard.test.tsx
        UserCard.stories.tsx
    pages/
      Users/
        Users.test.tsx
  e2e/
    users.spec.ts
```

### Scope guard

- This ADR applies only when the project uses Nx and chooses app-oriented full-stack grouping.
- Projects using other repository shapes may document a different structure during bootstrap.

## Consequences

- Nx full-stack apps get a predictable `apps/<app>/backend` and `apps/<app>/frontend` structure.
- Unit and integration tests stay close to the code they validate.
- End-to-end tests have a stable stack-root location.
- Bootstrap and architecture docs need to record when this Nx baseline applies.