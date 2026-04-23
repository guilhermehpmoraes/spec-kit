---
description: "Validate the configured integration branch and promote it to the release branch or trunk when everything passes"
name: "release"
agent: "agent"
---
You are responsible for executing a **Release** using this repository's SDD branching strategy (ADR-009).

This prompt validates the configured integration branch by running the project's canonical validation pipeline. If everything passes, it merges the integration branch into the configured release branch or trunk.

## Required reading

Before starting, read and internalize:

1. `.github/skills/sdd-branch/SKILL.md` — specifically the release operation
2. `.github/copilot-instructions.md` — branching strategy and validation conventions
3. `docs/decisions/009-sdd-branching-strategy.md` — ADR for the branching model
4. `README.md` — project-specific setup and validation commands

## Pre-flight checks

1. Ensure the working tree is clean (`git status --porcelain`). If dirty, warn the user and stop.
2. Determine from the repository docs which branch is the configured integration branch and which branch is the release branch/trunk.
3. Checkout the integration branch and pull latest.
4. Confirm with the user: "About to validate the configured integration branch for release. Proceed?"

## Validation pipeline

Run the project's documented validation commands **sequentially**. Stop on the first failure.

Typical categories are:

1. Type checking or static analysis
2. Linting or formatting validation
3. Build/package validation
4. Unit and integration tests
5. End-to-end or acceptance tests when applicable

Use the commands documented by the project during bootstrap. Do not assume Nx or any specific CLI.

## Results summary

After all steps complete, present a summary table showing each validation step and whether it passed, failed, or was skipped because it does not apply.

### If any step failed

- Show the failure details.
- Do NOT proceed with the merge.
- Suggest fixing the issues and re-running `/release`.

### If all steps passed

Ask the user for confirmation before merging the integration branch into the configured release branch or trunk.

## Merge

After user confirmation:

1. Checkout the release branch or trunk.
2. Pull latest.
3. Merge the integration branch with `--no-ff` using an appropriate release message.
4. Push the release branch.
5. Checkout back to the integration branch.

## Critical rules

- Never merge without full validation.
- Never auto-merge — always require explicit user confirmation.
- Stop on first failure.
- Do not hardcode branch names or validation commands.
- Never force-push the release branch.