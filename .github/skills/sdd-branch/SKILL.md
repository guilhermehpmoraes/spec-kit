---
name: sdd-branch
description: Manage SDD-aligned git branches for features and tasks. USE WHEN creating a feature spec (Step 1), starting task implementation (Step 4), finishing a task, or finishing a feature (Step 5). Also handles releases.
---

# SDD Branch

Manage the git branching lifecycle aligned with the SDD workflow (ADR-009). Creates, switches, merges, and cleans up feature and task branches using plain `git` commands.

## Configuration

- **Integration branch**: defined by the project during bootstrap and documented in `.github/copilot-instructions.md`
- **Feature branch pattern**: `feature/<feature-id>`
- **Task branch pattern**: `task/<task-id>`
- **Release branch/trunk**: defined by the project during bootstrap

Do not assume `sandbox`, `develop`, `main`, or any other branch unless the repository has explicitly configured it.

## Operations

### 1. Create Feature Branch

1. Identify `<feature-id>` from the feature spec.
2. Ensure the working tree is clean.
3. Fetch latest from remote.
4. Checkout the configured integration branch.
5. Pull latest.
6. Create and checkout `feature/<feature-id>`.
7. Push the feature branch to remote.

### 2. Create Task Branch

1. Identify `<task-id>` from the task spec.
2. Identify `<feature-id>` from the parent feature.
3. Ensure the working tree is clean.
4. Checkout `feature/<feature-id>`.
5. Pull latest.
6. Create and checkout `task/<task-id>`.

### 3. Finish Task

1. Ensure all changes are committed.
2. Checkout `feature/<feature-id>`.
3. Pull latest.
4. Merge `task/<task-id>` with `--no-ff`.
5. Delete the task branch.
6. Push the feature branch.

### 4. Finish Feature

1. Ensure the working tree is clean.
2. Checkout the configured integration branch.
3. Pull latest.
4. Merge `feature/<feature-id>` with `--no-ff`.
5. Push the integration branch.
6. Delete the feature branch locally and remotely after user confirmation.

### 5. Release

1. Ensure the working tree is clean.
2. Checkout the configured integration branch.
3. Pull latest.
4. Run the project's documented validation pipeline.
5. If all validations pass, ask for confirmation.
6. Merge the integration branch into the configured release branch or trunk using `--no-ff`.
7. Push the release branch.

## Edge Cases

- If the working tree is dirty, stop and ask the user to resolve it first.
- If a target branch already exists, checkout it instead of recreating it.
- If merge conflicts occur, stop and let the user resolve them.
- If remote operations fail, stop and report the failure.

## Important Notes

- Never force-push.
- Never delete long-lived branches without confirmation.
- Prefer merge commits over rebasing when the project documents that policy.
- Read the configured branch names from the repository docs before acting.