---
description: Detect code changes since free mode started and sync feature documentation
disable-model-invocation: true
allowed-tools: [Read, Write, Bash, Glob, Grep, AskUserQuestion]
---

You are running the docflow commit command. This detects what changed since `/docflow:free` was started and updates feature documentation accordingly.

## Pre-check

1. Verify `docs/INDEX.md` exists. If not, STOP: "docflow is not initialized. Run `/docflow:init` first."
2. `.docflow/state` must exist with `mode=free`. If missing, STOP: "No active free mode session. Run `/docflow:free` first." If mode is not `free`, STOP: "Current session is `<mode>`, not free. Run `/docflow:plan` to finish it, or delete `.docflow/state` to discard it."
3. Verify branch: run `git branch --show-current`. If current branch != `branch` in state file, STOP: "You started free mode on branch `<state.branch>` but are now on `<current>`. Switch back or delete `.docflow/state` to discard the session."

## Writes

- `docs/features/*.md` — create or update
- `docs/INDEX.md` — update
- `.docflow/state` — delete

## Steps

### Detect changes

Read `commitId` from `.docflow/state`.

Run `git diff <commitId> --stat` to get a summary of changed files. Then run `git diff <commitId>` to get the full diff.

If there are no changes, tell the user "No changes detected since free mode started." Then delete `.docflow/state` and STOP.

### Analyze and match

Read `references/conventions.md` for Feature ID format and md5 hash computation.

Read `docs/INDEX.md` and all existing feature docs in `docs/features/`.

For each changed file, determine which existing feature(s) it relates to by checking `## Associated Files` sections. Group changes by feature.

If any changes don't match existing features, flag them as untracked and ask the user:
- Assign to an existing feature
- Create a new feature (generate ID as `YYMMDD-name`, use template from `references/feature-template.md`)
- Skip (don't document)

### Update documentation

For each affected feature:
1. Read the feature doc
2. Update `## Associated Files` — add new files, update md5 hashes for changed files
3. If behavioral contracts changed, update `## Behavioral Contract`
4. If implementation approach changed, update `## Key Implementation Notes`
5. Update `## Acceptance Criteria` — check off completed items
6. Add a `## Change History` entry: `- <YYYY-MM-DD>: <brief description>`

Update `docs/INDEX.md` if feature statuses changed.

### Clean up

Delete `.docflow/state` (clears free mode state).

Summarize what was updated.
