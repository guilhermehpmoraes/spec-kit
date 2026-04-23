This file is part of a reusable kit.

If the consuming repository uses Nx, follow the Nx-specific guidance below.
If the repository does not use Nx, treat the Nx section as optional and rely on the project's actual build, task, and workspace tooling documented during bootstrap.

<!-- nx configuration start-->
<!-- Leave the start & end comments to automatically receive updates. -->

# General Guidelines for working with Nx

- For navigating/exploring the workspace, invoke the `nx-workspace` skill first - it has patterns for querying projects, targets, and dependencies
- When running tasks (for example build, lint, test, e2e, etc.), always prefer running the task through `nx` (i.e. `nx run`, `nx run-many`, `nx affected`) instead of using the underlying tooling directly
- Prefix nx commands with the workspace's package manager (e.g., `pnpm nx build`, `npm exec nx test`) - avoids using globally installed CLI
- You have access to the Nx MCP server and its tools, use them to help the user
- For Nx plugin best practices, check `node_modules/@nx/<plugin>/PLUGIN.md`. Not all plugins have this file - proceed without it if unavailable.
- NEVER guess CLI flags - always check nx_docs or `--help` first when unsure

## Scaffolding & Generators

- For scaffolding tasks (creating apps, libs, project structure, setup), ALWAYS invoke the `nx-generate` skill FIRST before exploring or calling MCP tools

## When to use nx_docs

- USE for: advanced config options, unfamiliar flags, migration guides, plugin configuration, edge cases
- DON'T USE for: basic generator syntax (`nx g @nx/react:app`), standard commands, things you already know
- The `nx-generate` skill handles generator discovery internally - don't call nx_docs just to look up generator syntax

## Additional MCP Usage

- Use Context7 MCP before writing plans, tasks, or implementation that depends on library/framework APIs, decorators, config, or CLI behavior.
- Use Playwright MCP whenever a task depends on understanding or validating real browser behavior, rendered UI states, navigation, forms, or e2e flows.
- Use GitKraken MCP for repository inspection in the SDD flow: status, diffs, branches, worktrees, pull/push operations, Launchpad, and PR triage.
- Use GitHub MCP when the SDD workflow depends on remote GitHub artifacts such as issues, pull requests, comments, reviews, releases, or assigned work.
- These MCPs complement the mandatory `sdd-branch` and `sdd-commit` skills; they do not replace them.


<!-- nx configuration end-->