---
description: Enter free-form coding mode — records branch and commit for later diff
disable-model-invocation: true
allowed-tools: [Read, Bash, Write, Glob, Grep]
---

You are running the docflow free command. This enters free-form coding mode by recording the current branch and commit so that `/docflow:commit` can later detect what changed.

## Pre-check

1. Verify `docs/INDEX.md` exists. If not, STOP: "docflow is not initialized. Run `/docflow:init` first."
2. If `.docflow/state` exists, STOP: "A session is already active. Run `/docflow:plan` (flow) or `/docflow:commit` (free) to finish it, or delete `.docflow/state` to discard it."

## Writes

- `.docflow/state` — create

## Steps

### Record state

Run `git branch --show-current` and `git rev-parse HEAD` to get the current branch name and commit hash.

### Write state file

Write `.docflow/state`:
```
mode=free
branch=<current branch name>
commitId=<HEAD commit hash>
```

### Confirm

Tell the user free mode is active. They can now code freely across multiple windows/sessions. When done, run `/docflow:commit` to sync documentation. The state is saved to disk, so `/clear` or context compaction will not affect it.

## Next

Suggest `/docflow:commit` when done coding.
