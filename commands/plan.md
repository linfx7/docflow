---
description: Convergent decision-making — grill the user until every detail is nailed down, then create feature docs
disable-model-invocation: true
allowed-tools: [Read, Write, Glob, Grep, Bash, AskUserQuestion]
argument-hint: [brainstorm-filename (optional)]
---

You are running the docflow plan command. This reads the brainstorm output and forces every open question to a decision through relentless interviewing, then creates feature documents.

## Pre-check

1. Verify `docs/INDEX.md` exists. If not, STOP: "docflow is not initialized. Run `/docflow:init` first."
2. `.docflow/state` must exist with `mode=flow`. If missing, STOP: "No active flow session. Run `/docflow:brainstorm` first." If mode is not `flow`, STOP: "Current session is `<mode>`, not flow. Run `/docflow:commit` to finish it, or delete `.docflow/state` to discard it."
3. Verify branch: run `git branch --show-current`. If current branch != `branch` in state file, STOP: "You started this flow session on branch `<state.branch>` but are now on `<current>`. Switch back or delete `.docflow/state` to discard the session."

## Writes

- `docs/features/*.md` — create or update
- `docs/INDEX.md` — update
- `.docflow/state` — delete
- `.docflow/brainstorm-*.md` — delete (optional, user confirms)

## Steps

### Decide

Read inputs:
- If an argument was provided, use it as the brainstorm filename (look in `.docflow/`)
- Otherwise, use `brainstormFile` from `.docflow/state`. If the file does not exist, STOP: "Brainstorm file not found: <path>. Delete `.docflow/state` and re-run `/docflow:brainstorm`."
- Read `docs/INDEX.md` to understand existing features
- Read relevant existing feature docs if the brainstorm touches existing functionality

Interview the user relentlessly about every aspect of this plan until reaching shared understanding. Walk down each branch of the design tree, resolving dependencies between decisions one-by-one. For each question, provide your recommended answer. Ask the questions one at a time. If a question can be answered by exploring the codebase, explore the codebase instead.

Do NOT proceed to the next step until every open question is resolved.

If the user expresses intent to abandon (e.g., "never mind", "let's not do this"), confirm with `AskUserQuestion`: (a) discard and delete state + brainstorm file, (b) pause and come back later (keep files). Then STOP.

### Split into features

Read `references/conventions.md` for Feature ID format.

Propose a feature split. For each feature specify:
- Whether it is NEW or an UPDATE to an existing feature
- Feature ID: `YYMMDD-name` format
- Scope: what behavioral contracts it covers
- Dependencies on other features
- Acceptance criteria

Present this as a clear summary. Use `AskUserQuestion` to let the user adjust the split.

### Write docs

After user confirms, for each feature:
- **New feature**: create `docs/features/<YYMMDD-name>.md` using the template from `references/feature-template.md`
- **Updated feature**: read the existing feature doc. Verify it contains the expected section headers (`## Behavioral Contract`, `## Key Implementation Notes`, `## Associated Files`, `## Acceptance Criteria`, `## Change History`). If any section is missing, add it. Then add new behavioral contracts, acceptance criteria, and a change history entry.
- Update `docs/INDEX.md` with any new features (status: `planned`), dependency changes, or status updates

### Clean up

Delete `.docflow/state` (clears flow state).

Use `AskUserQuestion` to ask the user whether to delete the consumed brainstorm file. If yes, delete it from `.docflow/`.

## Next

List all implementable features (status `planned` or just updated) and suggest `/docflow:implement`.
