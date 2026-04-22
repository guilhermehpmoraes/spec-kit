---
name: sdd-branch
description: Manage SDD-aligned git branches for features and tasks. USE WHEN creating a feature spec (Step 1), starting task implementation (Step 4), finishing a task, or finishing a feature (Step 5). Also handles releases to main. Trigger words - feature branch, task branch, finish task, finish feature, merge task, merge feature, branch, branching, release, deploy to main.
---

# SDD Branch

Manage the git branching lifecycle aligned with the SDD workflow (ADR-009). Creates, switches, merges, and cleans up feature and task branches using plain `git` commands.

## Configuration

- **Integration branch**: `sandbox` (will switch to `develop` when the project graduates from bootstrap phase)
- **Feature branch pattern**: `feature/<feature-id>`
- **Task branch pattern**: `task/<task-id>`

To change the integration branch, update the value above.

## Branch Naming

- `<feature-id>` — matches the feature folder name under `docs/specs/features/` (e.g., `001-admin-identity-domain`).
- `<task-id>` — matches the task spec filename prefix without the `.task.spec.md` suffix (e.g., `T001-setup-typeorm` from `T001-setup-typeorm.task.spec.md`).

## Operations

### 1. Create Feature Branch (Step 1 — Feature Spec)

**When**: The user starts a new feature (e.g., via `/feature` or Step 1 of the SDD flow).

**Steps**:

1. Identify the `<feature-id>` from the feature spec being created.
2. Ensure the working tree is clean (`git status --porcelain`). If dirty, warn the user and stop.
3. Fetch latest from remote: `git fetch origin`.
4. Checkout the integration branch: `git checkout sandbox` (or `develop`).
5. Pull latest: `git pull origin sandbox`.
6. Create and checkout the feature branch: `git checkout -b feature/<feature-id>`.
7. Push the branch to remote: `git push -u origin feature/<feature-id>`.
8. Confirm to the user: "Created and checked out `feature/<feature-id>` from `sandbox`."

**Important**: Feature branches are expected to be published immediately when they are created. If the feature branch already exists (e.g., resuming work), just check it out instead of creating it. If remote publishing fails because the remote is unavailable, stop the flow and tell the user the branch was not published.

### 2. Create Task Branch (Step 4 — Task Implementation)

**When**: The user starts implementing a task (e.g., via `/implement` or Step 4).

**Steps**:

1. Identify the `<task-id>` from the task spec being implemented.
2. Identify the `<feature-id>` from the task spec's parent feature.
3. Ensure the working tree is clean. If dirty, warn the user and stop.
4. Checkout the feature branch: `git checkout feature/<feature-id>`.
5. Pull latest: `git pull origin feature/<feature-id>`.
6. Create and checkout the task branch: `git checkout -b task/<task-id>`.
7. Confirm to the user: "Created and checked out `task/<task-id>` from `feature/<feature-id>`."

**Important**: Task branches are local-only by default (no push to remote unless the user requests it). If the task branch already exists, just check it out.

### 3. Finish Task (End of Step 4)

**When**: After the task is committed (via `sdd-commit`) and the user confirms the task is done.

**Steps**:

1. Ensure all changes are committed (working tree clean).
2. Checkout the feature branch: `git checkout feature/<feature-id>`.
3. Pull latest feature branch: `git pull origin feature/<feature-id>`.
4. Merge the task branch with no-fast-forward: `git merge --no-ff task/<task-id> -m "🔀 merge(task): T0XX <task-short-description>"`.
   - The merge message should reference the task ID and a brief description.
5. Delete the task branch: `git branch -d task/<task-id>`.
6. Push the feature branch: `git push origin feature/<feature-id>`.
7. Confirm to the user: "Merged `task/<task-id>` into `feature/<feature-id>` and cleaned up task branch."

### 4. Finish Feature (Step 5 — Feature Finish)

**When**: All tasks for the feature are `Done` and the user triggers Step 5.

**Steps**:

1. Ensure the working tree is clean.
2. Checkout the integration branch: `git checkout sandbox`.
3. Pull latest: `git pull origin sandbox`.
4. Merge the feature branch with no-fast-forward: `git merge --no-ff feature/<feature-id> -m "🔀 merge(feature): <feature-id> — <feature-title>"`.
   - The merge message should include the feature ID and title from the feature spec.
5. Push the integration branch: `git push origin sandbox`.
6. Delete the feature branch locally and remotely:
   - `git branch -d feature/<feature-id>`
   - `git push origin --delete feature/<feature-id>`
7. Confirm to the user: "Merged `feature/<feature-id>` into `sandbox` and cleaned up feature branch."

**Important**: Always ask the user for confirmation before pushing to the integration branch and deleting the feature branch.

### 5. Release to Main

**When**: The user triggers a release (e.g., via `/release`). This validates the integration branch and promotes it to `main`.

**Steps**:

1. Ensure the working tree is clean.
2. Checkout the integration branch: `git checkout sandbox`.
3. Pull latest: `git pull origin sandbox`.
4. **Run full validation pipeline** — execute each step sequentially and stop on first failure:
   a. Typecheck: `pnpm nx run-many -t typecheck`
   b. Lint (Biome): `pnpm nx run-many -t lint`
   c. Build: `pnpm nx run-many -t build`
   d. Unit tests: `pnpm nx run-many -t test` (if targets exist)
   e. E2E tests: `pnpm nx run-many -t e2e` (if targets exist)
5. If **any step fails**, stop immediately. Report the failure details to the user and do NOT proceed with the merge.
6. If **all steps pass**, inform the user of the results and ask for confirmation to merge into `main`.
7. After confirmation:
   a. Checkout main: `git checkout main`
   b. Pull latest: `git pull origin main`
   c. Merge with no-fast-forward: `git merge --no-ff sandbox -m "🚀 release: merge sandbox into main"`
   d. Push main: `git push origin main`
   e. Checkout back to integration branch: `git checkout sandbox`
8. Confirm to the user: "Release complete — `sandbox` merged into `main`."

**Important**:
- The integration branch is **NOT** deleted after release — it continues to receive new features.
- Always require explicit user confirmation before pushing to `main`.
- If the validation pipeline takes too long, inform the user and provide intermediate progress.

## Edge Cases

### Dirty working tree
If the working tree has uncommitted changes when a branch operation is requested, warn the user and suggest either committing or stashing before proceeding. Never force-checkout or discard changes.

### Branch already exists
- For feature branches: if `feature/<feature-id>` already exists locally, checkout and pull instead of creating.
- For task branches: if `task/<task-id>` already exists locally, checkout instead of creating.

### Merge conflicts
If a merge produces conflicts:
1. Inform the user which files conflict.
2. Do NOT auto-resolve. Let the user decide how to handle conflicts.
3. After the user resolves conflicts, continue with `git add` and `git commit`.

### Remote not available
If `git fetch` or `git push` fails due to network issues, stop the flow and inform the user that remote sync could not be completed.

## Integration with Other Skills

- **`sdd-commit`**: After committing on a task branch, the `sdd-branch` skill handles merging the task back into the feature branch.
- The branch operations should be offered **automatically** at the right SDD steps — not just when the user explicitly asks.

## Important Notes

- **Never force-push** — All operations use regular push. If a push is rejected, pull and merge first.
- **Never delete branches without confirmation** — Always ask the user before deleting feature branches (task branches are ephemeral and can be cleaned up automatically after merge).
- **Integration branch is configurable** — Currently `sandbox`, will become `develop`. Only update the Configuration section above when switching.
- **No rebase** — Merges only, to preserve clear audit trail aligned with spec references.
