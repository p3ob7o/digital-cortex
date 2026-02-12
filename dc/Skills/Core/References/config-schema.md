# Configuration Schema Reference

## Files in 9_Meta/

| File | Purpose |
|------|---------|
| `config.yaml` | User configuration (profile, sources, vault, behavior) |
| `memory.md` | Claude's learned preferences (readable, editable) |
| `state.json` | Sync timestamps, last ritual times, weekly intentions |
| `pending.json` | Queued write operations that failed |
| `prompts/` | Template files for rituals (morning-brief.md, etc.) |

## config.yaml Schema

```yaml
user:
  name: string              # Display name
  email: string             # Primary email
  timezone: string          # IANA timezone (e.g. Europe/Vienna)

sources:
  gmail:
    enabled: boolean
    exclude_categories: [string]   # e.g. [PROMOTIONS, UPDATES]

  calendar:
    enabled: boolean
    calendars: string | [string]   # "all" or list of calendar IDs

  slack:
    enabled: boolean
    mcp: string                    # MCP server name (e.g. context-a8c)

  p2:
    enabled: boolean
    mcp: string

  linear:
    enabled: boolean
    personal_team: string          # Team name for personal tasks (e.g. Totoro)
    personal_team_id: string       # Team UUID

  reminders:
    enabled: boolean
    lists: [string]                # e.g. [Reminders, Shopping]

vault:
  daily_note_format: string        # e.g. "YYYY-MM-DD dddd"
  archive_path: string             # e.g. "6_Archive/Daily Notes"
  stale_threshold_hours: number    # Default: 4

behavior:
  brief_mode: string               # "suggestion" | "autonomous"
  review_mode: string              # "interactive" | "suggestion"
  auto_retry_minutes: number       # Default: 5
  max_retries: number              # Default: 5
```

## Variable Syntax

Templates in `9_Meta/prompts/` use `{{variable}}` placeholders resolved at runtime.

### User Variables
```
{{user.name}}        → config.user.name
{{user.email}}       → config.user.email
{{user.timezone}}    → config.user.timezone
```

### Date Variables (computed at runtime)
```
{{date.today}}       → YYYY-MM-DD
{{date.weekday}}     → Monday, Tuesday, ...
{{date.month}}       → January, February, ...
{{date.day}}         → 1–31
{{date.year}}        → YYYY
{{date.iso}}         → ISO 8601 with timezone
```

### Source Variables
```
{{sources.linear.personal_team}}   → config value
{{sources.reminders.lists}}        → config value
```

### Computed Variables (from state.json)
```
{{last_brief}}          → ISO timestamp of last morning brief
{{days_since_review}}   → integer days since last weekly review
```

## state.json Schema

```json
{
  "last_sync": {
    "gmail": "ISO timestamp",
    "calendar": "ISO timestamp",
    "slack": "ISO timestamp",
    "p2": "ISO timestamp",
    "linear": "ISO timestamp",
    "reminders": "ISO timestamp"
  },
  "last_brief": "ISO timestamp",
  "last_review": "ISO timestamp",
  "last_weekly": "ISO timestamp",
  "weekly_intentions": ["string"]
}
```

## pending.json Schema

```json
{
  "pending": [
    {
      "id": "op-NNN",
      "operation": "create_task | create_reminder",
      "target": "linear | reminders",
      "payload": {},
      "created_at": "ISO timestamp",
      "failed_at": "ISO timestamp",
      "retries": 0,
      "last_error": "string"
    }
  ]
}
```

Retry schedule: immediate → 5 min → 15 min → 1 hour → manual (`/sync`).

## Template Format

Templates are markdown with YAML frontmatter:

```markdown
---
name: morning-brief
mode: suggestion
triggers:
  - /brief
  - /morning
sources:
  - gmail
  - calendar
  - slack
  - p2
  - linear
  - reminders
---

## Purpose
Prepare {{user.name}} for the day...

## Output Structure
Write to today's daily note...

## Behavior
- Present in terminal for review
- Wait for approval on each suggestion
```

## Validation

On load, validate config.yaml:
- All required fields present (user.name, user.email, user.timezone)
- Source configs have `enabled` boolean
- Timezone is valid IANA identifier
- Show clear error with line number and expected type on failure
