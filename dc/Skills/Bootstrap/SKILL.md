---
name: bootstrap
description: |
  Scaffold the Digital Cortex vault structure on first run.
  Use when the user invokes /bootstrap, asks to "set up the vault", "initialize digital cortex",
  or when dc:core detects that 9_Meta/config.yaml is missing.
  If the vault is already bootstrapped (config.yaml exists), inform the user and exit.
---

# Digital Cortex — Bootstrap

## Flow

1. Check if `9_Meta/config.yaml` exists in the vault root.
2. If it exists → tell the user bootstrap has already been completed and stop.
3. If it does not exist → create the folder structure below, then confirm what was created.

## Folder Structure

Create every folder listed below. Skip any that already exist.

```
.claude/
0_Inbox/
1_Fleeting/
2_Drafts/
3_Projects/
4_Areas/
5_Resources/
6_Archive/
  Daily Notes/
7_Assets/
8_People/
9_Meta/
  Prompts/
  Templates/
```

## Initial Files

After creating folders, write these seed files (skip any that already exist):

### `9_Meta/config.yaml`

```yaml
# Digital Cortex Configuration
# Edit this file to customize your setup.

user:
  name:
  email:
  timezone:

sources: {}

vault:
  daily_note_format: "YYYY-MM-DD dddd"
  archive_path: "6_Archive/Daily Notes"
  stale_threshold_hours: 4

behavior:
  brief_mode: suggestion
  review_mode: interactive
```

### `9_Meta/memory.md`

Empty file.

### `9_Meta/pending.json`

```json
[]
```

### `9_Meta/state.json`

```json
{}
```

## Output

After scaffolding, print the folder tree that was created and remind the user to fill in
`9_Meta/config.yaml` with their profile (name, email, timezone) and source connections.
