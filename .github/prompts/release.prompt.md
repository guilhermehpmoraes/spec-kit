---
description: "Validate the integration branch (build, lint, test, e2e) and merge into main if everything passes"
name: "release"
agent: "agent"
---
You are responsible for executing a **Release to Main** using this repository's SDD branching strategy (ADR-009).

This prompt validates the integration branch by running the full build and test pipeline. If everything passes, it merges the integration branch into `main`.

## Required reading

Before starting, read and internalize:

1. `.github/skills/sdd-branch/SKILL.md` — specifically **Operation 5: Release to Main**
2. `.github/copilot-instructions.md` — branching strategy and conventions
3. `docs/decisions/009-sdd-branching-strategy.md` — ADR for the branching model

## Pre-flight checks

1. Ensure the working tree is clean (`git status --porcelain`). If dirty, warn the user and stop.
2. Checkout the integration branch (`sandbox`) and pull latest:
   ```
   git checkout sandbox
   git pull origin sandbox
   ```
3. Confirm with the user: "About to validate `sandbox` for release to `main`. Proceed?"

## Validation pipeline

Run each step **sequentially**. Stop on the first failure.

### Step 1 — Typecheck
```
pnpm nx run-many -t typecheck
```
If fails → report errors and stop. Do NOT proceed to the next step.

### Step 2 — Lint (Biome)
```
pnpm nx run-many -t lint
```
If fails → report errors and stop.

### Step 3 — Build
```
pnpm nx run-many -t build
```
If fails → report errors and stop.

### Step 4 — Unit tests
```
pnpm nx run-many -t test
```
If no `test` targets exist, skip and note it. If fails → report errors and stop.

### Step 5 — E2E tests
```
pnpm nx run-many -t e2e
```
If no `e2e` targets exist, skip and note it. If fails → report errors and stop.

## Results summary

After all steps complete, present a summary table:

| Step | Status |
|------|--------|
| Typecheck | ✅ / ❌ |
| Lint | ✅ / ❌ |
| Build | ✅ / ❌ |
| Unit tests | ✅ / ⏭️ skipped |
| E2E tests | ✅ / ⏭️ skipped |

### If any step failed

- Show the failure details.
- Do NOT proceed with the merge.
- Suggest fixing the issues and re-running `/release`.

### If all steps passed

Ask the user for confirmation: "All validations passed. Merge `sandbox` into `main`?"

## Merge to main

After user confirmation:

```
git checkout main
git pull origin main
git merge --no-ff sandbox -m "🚀 release: merge sandbox into main"
git push origin main
git checkout sandbox
```

## Post-release confirmation

Report to the user:
- "Release complete — `sandbox` merged into `main`."
- Summary of what was validated.

## Critical rules

- **Never merge without full validation** — all pipeline steps must pass.
- **Never auto-merge** — always require explicit user confirmation before pushing to `main`.
- **Stop on first failure** — do not continue running validation steps after a failure.
- **Integration branch survives** — do NOT delete `sandbox` after release. It continues receiving new features.
- **Never force-push to main** — if push is rejected, pull and resolve first.
