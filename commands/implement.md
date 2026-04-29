---
description: Implement planned features in isolated worktrees with dependency analysis
disable-model-invocation: true
allowed-tools: [Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion, Agent, EnterWorktree, ExitWorktree]
---

You are running the docflow implement command. This selects feature(s) to implement, analyzes dependencies, then works in isolated worktrees.

## Pre-check

1. Verify `docs/INDEX.md` exists. If not, STOP: "docflow is not initialized. Run `/docflow:init` first."
2. If `.docflow/state` exists, STOP: "A session is already active. Run `/docflow:plan` (flow) or `/docflow:commit` (free) to finish it, or delete `.docflow/state` to discard it."

## Writes

- `docs/features/*.md` — update (acceptance criteria, associated files, change history)
- `docs/INDEX.md` — update (feature status → `active`)
- Source code — create and modify (in worktree)

## Steps

### Select features

Read `docs/INDEX.md`. List all features with status `planned`:

| # | ID | Name | Status | Dependencies |
|---|-----|------|--------|-------------|
| 1 | 260430-user-login | user-login | planned | — |
| 2 | 260430-auth-middleware | auth-middleware | planned | 260430-user-login |

If no implementable features exist, STOP: "No planned features found. Run `/docflow:brainstorm` and `/docflow:plan` first."

Use `AskUserQuestion` to let the user select one or more features to implement.

### Dependency analysis

For each selected feature, read its feature doc from `docs/features/`.

Analyze across all selected features:
1. **Dependency ordering**: if feature A depends on feature B and both are selected, B must be implemented first. Flag circular dependencies.
2. **File overlap**: check `## Associated Files` across selected features. If multiple features touch the same files, warn about potential merge conflicts.
3. **Shared patterns**: identify common infrastructure or patterns that multiple features need (e.g., shared types, utility functions, database migrations).

Present the analysis:
```
Dependency order: B → A → C
File conflicts: src/auth.ts is touched by both A and B
Shared work: Both A and B need a new User type in src/types.ts
Recommendation: Implement B first, then A (they share src/auth.ts). C is independent.
```

Use `AskUserQuestion` to confirm the implementation plan.

### Implement

#### Single feature

Use `EnterWorktree` to create an isolated worktree. Then implement the feature:

- Read the feature doc thoroughly before writing any code
- Transform each acceptance criterion into a verifiable goal
- Run tests/builds after each significant change

After implementation is complete:
1. Update the feature doc: check off acceptance criteria, update associated files and hashes, add change history entry
2. Update `docs/INDEX.md`: change feature status to `active`
3. Use `ExitWorktree` with `action: "keep"` to preserve the worktree and branch

Tell the user the worktree branch name and path. Remind them to merge and clean up:
```
git merge <branch-name>
git worktree remove <worktree-path>
git branch -d <branch-name>
```

#### Multiple features

- For **independent features** (no file overlap, no dependency): launch each as a separate `Agent` with `isolation: "worktree"`. Each agent receives the feature doc content and the implementation guidelines above.
- For **dependent features**: implement them sequentially in dependency order, each in its own worktree. Complete one before starting the next.

After all agents complete, summarize results and list branches to merge.
