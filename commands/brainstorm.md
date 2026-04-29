---
description: Divergent exploration of a feature idea — discover, don't decide
disable-model-invocation: true
allowed-tools: [Read, Glob, Grep, Bash, AskUserQuestion, Write]
---

You are running the docflow brainstorm command. This openly explores a feature idea with the user — surfaces unknowns, discovers constraints, and collects possibilities without forcing decisions.

## Pre-check

1. Verify `docs/INDEX.md` exists. If not, STOP: "docflow is not initialized. Run `/docflow:init` first."
2. If `.docflow/state` exists, STOP: "A session is already active. Run `/docflow:plan` (flow) or `/docflow:commit` (free) to finish it, or delete `.docflow/state` to discard it."

## Writes

- `.docflow/brainstorm-*.md` — create
- `.docflow/state` — create

## Steps

### Explore

Ask open-ended questions to understand:
- What problem are they trying to solve? Why now?
- What does success look like from the user's perspective?
- What related code or patterns already exist in the codebase?
- What technical constraints or dependencies might apply?
- What are the possible approaches — without committing to one?

If a question can be answered by exploring the codebase (Glob, Grep, Read), explore instead of asking.

Keep the conversation divergent. When the user starts narrowing down prematurely, ask "what else could this be?" or "are there other angles we haven't considered?" Surface unknowns rather than resolving them.

### Write brainstorm

When the exploration feels thorough — the problem space is well-mapped and the key unknowns are surfaced — write the result to `.docflow/brainstorm-<yyyyMMdd-HHmmss>.md`:

```markdown
# Brainstorm: <title>

## Context
<background, motivation, why this matters>

## Discoveries
- <facts found in code, existing patterns, technical constraints>

## Related Existing Features
- <existing features from docs/INDEX.md that this idea touches, or "None">

## Ideas & Possibilities
- <functional options explored, no commitment>

## Open Questions
- <decisions to be made in plan phase>
```

### Create state

Create `.docflow/state`:
```
mode=flow
branch=<current branch from git branch --show-current>
brainstormFile=.docflow/brainstorm-<yyyyMMdd-HHmmss>.md
```

## Next

Suggest `/docflow:plan` to make decisions and create feature docs.
