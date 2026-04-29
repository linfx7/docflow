# Docflow

A Claude Code plugin that keeps feature documentation in sync with code — making "change code" and "change docs" two sides of the same action.

## Why

Code evolves continuously; docs don't. Stale docs mislead both humans and LLMs. Docflow ties documentation updates to the coding workflow so they can't fall out of sync.

## Install

```
/plugin marketplace add linfx7/docflow
/plugin install docflow
```

## Commands

### Setup

| Command | Description |
|---------|-------------|
| `/docflow:init` | Initialize docflow in the current project |

### Flow Mode

Structured cycle: brainstorm → plan → implement.

| Command | Description |
|---------|-------------|
| `/docflow:brainstorm` | Divergent exploration — discover, don't decide (triggers flow state) |
| `/docflow:plan` | Grill-me style convergence — force every decision, then create feature docs (requires flow state) |

### Free Mode

Code first, document later: free → (code freely) → commit.

| Command | Description |
|---------|-------------|
| `/docflow:free` | Enter free mode — records branch and commit for later diff (triggers free state) |
| `/docflow:commit` | Detect changes since free mode started and update feature docs (requires free state) |

### Standalone

| Command | Description |
|---------|-------------|
| `/docflow:implement` | Implement planned features in isolated worktrees with dependency analysis |
| `/docflow:sync` | Detect drift between code and docs, resolve interactively |

## State Management

`.docflow/state` tracks the current mode. Two modes are mutually exclusive:

- **flow**: triggered by `brainstorm`, only `plan` can run (clears on completion)
- **free**: triggered by `free`, only `commit` can run (clears on completion)

When a state is active, all other commands are rejected. Finish the session or delete `.docflow/state` to discard it. The state file also records the branch name to prevent cross-branch corruption.

## Data Model

```
CLAUDE.md                # Appended with docflow constraints
docs/
  INDEX.md               # Feature index (YAML frontmatter)
  features/
    YYMMDD-name.md       # One doc per feature
.docflow/                # Temp directory (gitignored)
```

**INDEX.md** — feature list with IDs, status, and dependencies. No file-level detail.

**Feature doc** — the smallest self-contained unit (readable without other features):

- **Behavioral Contract** — observable behavior, not implementation
- **Key Implementation Notes** — technical choices that affect behavior
- **Associated Files** — source files with md5 hashes for drift detection
- **Acceptance Criteria** — testable conditions
- **Change History** — chronological record of updates

Feature lifecycle: `planned` → `active` → `deprecated`.

Feature IDs use `YYMMDD-name` format (e.g., `260430-user-login`). Name is a slug: lowercase letters, digits, hyphens.

## Plugin Structure

```
docflow/
  .claude-plugin/
    plugin.json
    marketplace.json
  commands/
    init.md, brainstorm.md, plan.md, implement.md
    free.md, commit.md, sync.md
  references/
    feature-template.md   # Shared by init, plan, commit, sync
    conventions.md         # Feature ID format, md5 computation
```

## Design Principles

- Feature docs are the source of truth for behavioral contracts
- One feature = one self-contained unit
- Flow and free modes are mutually exclusive
- All command prompts in English; user interaction follows user's language
- All commands set `disable-model-invocation: true` — user-invoked only

## Acknowledgements

- `/docflow:plan` adopts the interview technique from [grill-me](https://github.com/mattpocock/skills) by Matt Pocock
- Coding guidelines in CLAUDE.md are adapted from [andrej-karpathy-skills](https://github.com/forrestchang/andrej-karpathy-skills) by Jiayuan Zhang
