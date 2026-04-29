---
description: Initialize docflow in the current project
disable-model-invocation: true
allowed-tools: [Read, Write, Glob, Grep, Bash, AskUserQuestion]
---

You are running the docflow init command. This sets up feature documentation tracking for the current project.

## Pre-check

1. Verify this is a git repository (run `git rev-parse --git-dir`). If it fails, STOP: "Not a git repository. Please run `git init` first."
2. Check the state of docflow initialization:
   - If `docs/INDEX.md` exists and is valid YAML frontmatter: STOP: "docflow is already initialized. Use `/docflow:sync` to check documentation consistency."
   - If `docs/` directory exists but `docs/INDEX.md` is missing or has invalid YAML: check whether `docs/` contains non-docflow content (files other than `INDEX.md` and `features/`). Then use `AskUserQuestion`:
     - If non-docflow content detected: "(a) Add docflow files alongside existing content in `docs/`, (b) Use a different directory for docflow (specify name), (c) Abort"
     - If only docflow artifacts (empty or only `features/`): "(a) Continue initialization (reuse existing `docs/features/` if present), (b) Clean up docflow files and start fresh (removes only `docs/features/`, `docs/INDEX.md`, and `.docflow/`)"

## Writes

- `docs/INDEX.md` — create
- `docs/features/*.md` — create
- `.docflow/` — create directory
- `.gitignore` — append `.docflow/`
- `CLAUDE.md` — create or append

## Steps

### Create directories

1. Create `docs/`, `docs/features/`, `.docflow/`
2. Add `.docflow/` to `.gitignore`: check if `.gitignore` already contains `.docflow/`. If not, append it. If `.gitignore` does not exist, create it.

### Analyze codebase

Use `git ls-files` to list tracked source files (respects `.gitignore`). Filter out common non-source files (configs, lockfiles, markdown).

- If no source code files beyond config/boilerplate: this is a **fresh project** — create an empty `docs/INDEX.md` with the following content, then skip to **Update CLAUDE.md**:
  ```yaml
  ---
  features: []
  ---
  ```
- If **existing codebase**: proceed to **Generate docs**.

### Generate docs

Read `references/conventions.md` for Feature ID format and md5 hash computation.

1. Scan the project structure and source files. Identify distinct functional modules that qualify as features. A feature is self-contained — it can be understood without reading other features.
2. Show all identified features as a summary table (ID, name, brief description). Use `AskUserQuestion` to let the user adjust (add/remove/rename/merge).
3. After confirmation, batch-generate:
   - One feature doc per feature in `docs/features/<YYMMDD-name>.md` using the template from `references/feature-template.md`
   - `docs/INDEX.md` with all features listed (status: `active`)
   - Compute md5 hashes for all associated files

### Update CLAUDE.md

Read the project's `CLAUDE.md` and apply the docflow snippet:

- If `CLAUDE.md` does not exist, create it with the snippet below.
- If `CLAUDE.md` exists and contains a `<!-- docflow:start -->` block: compare with the snippet. If different, show the diff and use `AskUserQuestion` to confirm before replacing. If identical, skip.
- If `CLAUDE.md` exists but has no `<!-- docflow:start -->` block: append the snippet.

```markdown
<!-- docflow:start -->
## Docflow

This project uses [docflow](https://github.com/linfx7/docflow) to keep feature documentation in sync with code.

- Feature docs live in `docs/features/`, indexed by `docs/INDEX.md`
- Feature docs are the source of truth for behavioral contracts
- When modifying code, check if changes affect any feature's associated files
- All code changes must include tests covering the acceptance criteria defined in the feature document
- Use `/docflow:sync` to detect documentation drift

## Coding Guidelines

- **Think before coding**: state assumptions explicitly. If uncertain, ask. If multiple interpretations exist, present them. Push back when a simpler approach exists.
- **Simplicity first**: minimum code that solves the problem. No speculative features, no abstractions for single-use code, no error handling for impossible scenarios.
- **Surgical changes**: touch only what you must. Don't improve adjacent code or formatting. Match existing style. Every changed line should trace directly to the request.
- **Goal-driven execution**: transform tasks into verifiable goals with success criteria. For multi-step tasks, state a brief plan with verification checks at each step.
<!-- docflow:end -->
```

## Next

Suggest `/docflow:brainstorm` to explore a feature idea, or `/docflow:free` to start coding and document later.
