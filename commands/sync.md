---
description: Check consistency between code and feature documentation
disable-model-invocation: true
allowed-tools: [Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion]
---

You are running the docflow sync command. This detects inconsistencies between code and feature documentation, then helps the user resolve them.

## Pre-check

1. Verify `docs/INDEX.md` exists. If not, STOP: "docflow is not initialized. Run `/docflow:init` first."
2. If `.docflow/state` exists, STOP: "A session is already active. Run `/docflow:plan` (flow) or `/docflow:commit` (free) to finish it, or delete `.docflow/state` to discard it."

## Writes

- `docs/features/*.md` — create, update, or delete
- `docs/INDEX.md` — update

## Steps

### Read current state

- Read `docs/INDEX.md` and parse its YAML frontmatter. If malformed, STOP and show the parse error.
- Read all feature docs from `docs/features/`
- Collect all associated file paths and recorded hashes

### Detect inconsistencies

Read `references/conventions.md` for md5 hash computation.

Run all checks below, then present a summary table. If nothing found, tell the user everything is in sync and STOP.

**File-level checks:**

1. **Hash mismatch**: compute current md5 for each associated file, compare with recorded hash.
2. **Missing files**: associated files that no longer exist. If only some files of a feature are deleted, flag as partial (may indicate incomplete refactoring).
3. **Possible renames**: cross-reference missing files with untracked files — match by similar name or identical content hash. Flag as rename rather than separate delete + add.
4. **Untracked files**: source files not associated with any feature. Use `git ls-files --cached --others --exclude-standard`, exclude non-source patterns: `docs/**`, `.docflow/**`, `.*`, `*.md`, `*.json`, `*.yaml`, `*.yml`, `*.toml`, `*.lock`, `*.config.*`, `Makefile`, `Dockerfile`, `LICENSE*`, `CLAUDE.md`.

**Feature-level checks:**

5. **Status mismatch**: `active` features whose files are all deleted, or `deprecated` features with live code.
6. **Stale planned**: `planned` features with no associated files and no change history updates in 30+ days.

**Index integrity checks:**

7. **INDEX/doc mismatch**: docs in `docs/features/` without INDEX entry, or INDEX entries without a doc file.
8. **ID consistency**: verify ID matches across filename, doc YAML frontmatter, and INDEX.md.
9. **Duplicate IDs**: same ID used by multiple features in INDEX.md.

### Resolve

For each inconsistency, use `AskUserQuestion` to let the user choose:

| Issue | Options |
|-------|---------|
| Hash mismatch | (a) Update doc to reflect code (b) Skip |
| Missing files | (a) Remove from associations (b) Deprecate feature (c) Skip |
| Possible rename | (a) Update path + refresh hash (b) Treat as delete + add (c) Skip |
| Untracked file | (a) Associate with existing feature (b) Create new feature (use `references/feature-template.md`) (c) Ignore |
| Status mismatch | (a) Update status (b) Skip |
| Stale planned | (a) Keep (b) Remove feature entirely (c) Skip |
| Orphan doc | (a) Add to INDEX.md (b) Delete doc (c) Skip |
| Missing doc | (a) Create from `references/feature-template.md` (b) Remove INDEX entry (c) Skip |
| ID mismatch | (a) Fix all to INDEX.md ID (b) Fix all to filename ID (c) Skip |
| Duplicate IDs | (a) Reassign one to new ID (b) Skip |

Apply all confirmed resolutions. Refresh md5 hashes for updated associations.

## Next

List any features with status `planned` and suggest `/docflow:implement`.
