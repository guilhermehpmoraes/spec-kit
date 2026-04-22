---
name: sdd-commit
description: Commit changes following Conventional Commits + Gitmoji standards, using context from the current task spec. USE WHEN the user wants to commit after finishing a task, or explicitly asks to commit. Trigger words - commit, commitar, save changes, git commit.
---

# SDD Commit

Commit staged or unstaged changes using **Conventional Commits** format with **Gitmoji** prefixes. The commit message is derived from the task context (task spec, changed files, and feature spec).

## When to Use

- At the end of **Step 4 — Task Implementation** when the user accepts the commit prompt.
- Whenever the user explicitly asks to commit current changes.

## Commit Message Format

```
<gitmoji> <type>(<scope>): <short description>

<optional body>

Refs: <feature-spec-path> | <task-spec-path>
```

### Rules

1. **Gitmoji** — Use the emoji (not the code) that best matches the change type.
2. **Type** — Standard Conventional Commits type: `feat`, `fix`, `chore`, `docs`, `refactor`, `test`, `ci`, `build`, `perf`, `style`.
3. **Scope** — Derived from the affected app, package, or domain (e.g., `admin-backend`, `database`, `identity`). Use the most specific scope possible.
4. **Short description** — Imperative mood, lowercase, no period, max 72 chars.
5. **Body** (optional) — Brief explanation of *what* and *why*, not *how*. Use when the subject line alone is insufficient.
6. **Refs footer** — Include the relative path to the task spec and/or feature spec being implemented.

### Gitmoji Quick Reference

| Emoji | Code | Type | Use when |
|-------|------|------|----------|
| ✨ | `:sparkles:` | feat | New feature |
| 🐛 | `:bug:` | fix | Bug fix |
| 📝 | `:memo:` | docs | Documentation only |
| ♻️ | `:recycle:` | refactor | Refactor (no feature/fix) |
| ✅ | `:white_check_mark:` | test | Adding/updating tests |
| 🔧 | `:wrench:` | chore | Config/tooling changes |
| 🏗️ | `:building_construction:` | build | Build system or dependencies |
| 🚀 | `:rocket:` | perf | Performance improvement |
| 🎨 | `:art:` | style | Code style/formatting |
| 💚 | `:green_heart:` | ci | CI/CD changes |
| 🔥 | `:fire:` | chore | Removing code/files |
| 🚚 | `:truck:` | chore | Moving/renaming files |
| 🗃️ | `:card_file_box:` | feat | Database/migration changes |
| 🔒 | `:lock:` | fix | Security fix |
| ➕ | `:heavy_plus_sign:` | build | Adding a dependency |
| ➖ | `:heavy_minus_sign:` | build | Removing a dependency |
| 🏷️ | `:label:` | chore | Types/interfaces changes |
| 🍱 | `:bento:` | chore | Assets changes |

## Commit Splitting Strategy

Not every change set needs multiple commits, and not every multi-file change should be a single commit. Use judgment:

### When to split into multiple commits

- **Different concerns in the same file or feature** — e.g., a component rewritten with both logic changes (`feat`/`refactor`) and visual/styling changes (`style`). Split by concern.
- **Changes spanning unrelated projects/packages** — e.g., backend API change + unrelated frontend config fix. Each project gets its own commit if the changes are logically independent.
- **Mixed types that would produce a misleading single message** — e.g., a new feature + a bug fix discovered along the way. Separate them so the git history is meaningful.

### When to keep as a single commit

- **Tightly coupled changes** — a new API endpoint + its corresponding frontend call + shared types are one logical unit, even across multiple projects.
- **Small scope** — if the total diff is small and cohesive, one commit is fine regardless of how many files changed.
- **Same concern, same scope** — touching 5 files to implement one feature is one commit, not five.

### Rule of thumb

> Ask: *"Would reverting part of this change leave the codebase in a broken or confusing state?"* If yes, it belongs in one commit. If parts can be reverted independently and still make sense, they can be separate commits.

**Target: 1–3 commits per task.** Most tasks produce 1 commit. Only split when there's a clear logical boundary. Never exceed 4–5 commits for a single task — if that's happening, the task itself may need splitting.

## Steps

### 1. Gather Context

Identify the **task spec** currently being implemented (from the conversation or the active editor file). Read:
- The task spec (`docs/specs/features/<feature-id>/tasks/<task-file>`) for the task title, ID, and scope.
- The feature spec (`docs/specs/features/<feature-id>/feature.spec.md`) for the feature name and ID.

### 2. Review Changes

Use `mcp_gitkraken_git_status` to see the current working tree status.
Use `mcp_gitkraken_git_log_or_diff` with action `diff` to review unstaged/staged changes.

Analyze the diff to determine:
- The **type(s)** of change (feat, fix, refactor, style, etc.)
- The **scope(s)** (which app/package/domain is affected)
- Whether the changes have **distinct logical boundaries** that warrant splitting

### 3. Plan the Commit(s)

Based on the analysis and the splitting strategy above, decide how many commits are needed.

**If single commit:** Compose one message following the format above.

**If multiple commits:** Build a **commit plan** — a numbered list of commits with:
- Which files go into each commit
- The proposed commit message for each
- Brief rationale for the split

Present the full plan to the user **before staging anything**. Example:

> **Commit plan (2 commits):**
>
> 1. `✨ feat(identity): add password hashing to user service` — files: `users.service.ts`, `users.service.spec.ts`
> 2. `🎨 style(identity): redesign login form layout and spacing` — files: `login.component.tsx`, `login.styles.css`
>
> Rationale: logic and styling are independent concerns; separating them keeps the history clean.

Wait for user approval or adjustments before proceeding.

### 4. Stage and Commit

For each commit in the plan (or the single commit):

1. Use `mcp_gitkraken_git_add_or_commit` with action `add` to stage **only the files for that specific commit**.
2. Present the commit message to the user for final approval (skip if already approved in the plan).
3. After approval, use `mcp_gitkraken_git_add_or_commit` with action `commit` with the approved message.
4. If there are more commits in the plan, repeat from sub-step 1 for the next commit.

### 5. Confirm

Report the result of all commits to the user. Summarize what was committed and how many commits were created.

## Important Notes

- **Never auto-commit without user approval** — Always show the message (or commit plan) first.
- **Keep commits atomic** — One logical change per commit. If the diff contains unrelated changes, suggest splitting.
- **Don't over-split** — 1 commit is the default. Only split when there's genuine benefit to the git history. Avoid commit spam.
- **Portuguese naming in entities is expected** — Don't flag TypeORM entity names in Portuguese as issues; that's by design (ADR-005).
- **Spec references matter** — Always include the `Refs:` footer linking to the task/feature spec in every commit.
- **Multi-line commit messages** — When passing the commit message to `mcp_gitkraken_git_add_or_commit`, use actual line breaks (newlines) in the message string, **never** use literal `\n` escape sequences. The message parameter must contain real newline characters so git receives a properly formatted multi-line message. Only break lines between logical paragraphs (subject, body, footer) — do **not** wrap or split sentences mid-phrase just to shorten lines.
- **Commit body style** — Always include a body with bullet points (`-`) listing each logical change made in the commit. Each bullet should be a concise but descriptive sentence. Keep a blank line between subject, body, and `Refs:` footer. Example:

  ```
  📝 docs(identity): add identity domain spec and update admin architecture

  - Create docs/specs/domains/identity.md with domain responsibilities, entities, modules, API surface, and related ADRs
  - Update admin architecture doc with module hierarchy, infrastructure dependencies, and shared package details
  - Link domain spec from feature spec Domain field
  - Mark T007 as Done

  Refs: docs/specs/features/001-admin-identity-domain/tasks/T007-identity-domain-docs.task.spec.md
  ```
