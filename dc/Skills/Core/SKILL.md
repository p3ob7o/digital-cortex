---
name: core
description: |
  Digital Cortex orchestrator — personal operating system built on Claude Code and Obsidian.
  Activate when the user interacts with their Obsidian vault, invokes Digital Cortex commands
  (/brief, /review, /weekly, /process, /capture, /task, /reminder, /today, /tasks, /projects,
  /status, /sync, /help), uses natural language that maps to these commands ("start my day",
  "let's do the morning brief", "what's on today", "evening review time", "show me my tasks"),
  or works with vault notes, daily notes, inbox processing, or personal task management.
  This skill loads the system configuration from 9_Meta/config.yaml, understands the vault
  structure, and dispatches to the appropriate dc:* feature skill.
---

# Digital Cortex — Core

## Vision

A personal operating system that reduces friction in capturing ideas, tasks, and information
while proactively synthesizing inputs from work tools into actionable daily guidance. Implements
PARA for organization and GTD for processing, with Claude as an intelligent layer that prepares
briefs, facilitates reviews, and maintains the system.

## Core Principles

1. **Frictionless capture** — any thought enters the system with minimal effort
2. **Async processing** — captures accumulate; Claude processes them on schedule
3. **Proactive synthesis** — Claude pulls from external sources and prepares actionable briefs
4. **Explicit memory** — all configuration, prompts, and preferences are visible and editable
5. **Graceful degradation** — system works offline and tolerates partial failures
6. **Portability** — no hardcoded personal data; everything configurable via `9_Meta/config.yaml`

## Architecture

```
CAPTURE LAYER (Share Extension, Quick Capture, Voice Memo)
        │
        ▼
OBSIDIAN VAULT (0_Inbox → 6_Archive, organized by PARA)
        │
        ▼
CLAUDE CODE LAYER (Rituals, Processing, Queries)
        │
        ▼
EXTERNAL SOURCES (Gmail, Calendar, Slack, P2, Linear, Reminders)
```

## Vault Structure

| Folder | Purpose |
|--------|---------|
| `0_Inbox/` | Daily notes + unprocessed captures |
| `1_Fleeting/` | Processed notes without a home yet |
| `2_Drafts/` | Work-in-progress posts (folders) |
| `3_Projects/` | Active projects (folders with index) |
| `4_Areas/` | Ongoing responsibilities (files) |
| `5_Resources/` | Reference material |
| `6_Archive/` | Completed/processed items; `Daily-Notes/YYYY/MM/` |
| `7_Assets/` | Linked files (images, PDFs, etc.) |
| `8_People/` | One markdown file per person (PRM) |
| `9_Meta/` | Config, Prompts, memory, state |

## Configuration

On activation, read `9_Meta/config.yaml` to understand:
- **User profile**: name, email, timezone
- **Enabled sources**: which external services are connected
- **Vault settings**: daily note format, archive path, stale thresholds
- **Behavior modes**: suggestion vs autonomous, retry settings

Resolve `{{variables}}` in templates using config values and computed dates.
See [References/config-schema.md](References/config-schema.md) for the full schema,
variable syntax, state tracking, and template format.

## Feature Skills

Each feature is a separate `dc:*` skill. Dispatch to the appropriate one based on context.

| Skill | Feature | Handles |
|-------|---------|---------|
| `dc:vault-structure` | F01 | Vault scaffolding, folder creation, structure validation |
| `dc:capture` | F02 | Share extension, quick capture, voice memo |
| `dc:daily-note` | F03 | Daily note creation, structure, archival |
| `dc:frontmatter` | F04 | Frontmatter schemas for all entity types |
| `dc:morning-brief` | F05 | Morning brief orchestration and source synthesis |
| `dc:evening-review` | F06 | Interactive daily reflection and processing |
| `dc:weekly-review` | F07 | Projects review, fleeting notes, week-ahead planning |
| `dc:inbox` | F08 | On-demand inbox triage with suggestions |
| `dc:tasks` | F09 | Linear + Apple Reminders integration |
| `dc:sources` | F10 | External source connections and sync |
| `dc:commands` | F11 | Command parsing, help, error messages |
| `dc:errors` | F12 | Graceful degradation, offline mode, retry queue |
| `dc:templates` | F13 | Prompt templates and configuration management |
| `dc:bootstrap` | F14 | First-run setup and reconfiguration |
| `dc:people` | F15 | Personal relationship management |

## Command Routing

When the user invokes a command (slash, CLI, or natural language), route to the right skill:

### Rituals

| Command | Aliases | Natural Language | Skill |
|---------|---------|------------------|-------|
| `/brief` | `/morning`, `/am` | "start my day", "morning brief" | `dc:morning-brief` |
| `/review` | `/evening`, `/pm` | "evening review time" | `dc:evening-review` |
| `/weekly` | `/week` | "weekly review" | `dc:weekly-review` |
| `/process` | `/inbox` | "let's process the inbox" | `dc:inbox` |

### Capture

| Command | Natural Language | Skill |
|---------|------------------|-------|
| `/capture <text>` | (direct capture) | `dc:capture` |
| `/task <text>` | "create a task to..." | `dc:tasks` |
| `/reminder <text>` | "remind me to..." | `dc:tasks` |
| `/shopping <text>` | "I need to buy..." | `dc:tasks` |

### Query

| Command | Natural Language | Skill |
|---------|------------------|-------|
| `/today` | "what's on today?" | `dc:morning-brief` |
| `/tasks` | "show me my tasks" | `dc:tasks` |
| `/projects` | "what projects are active?" | `dc:frontmatter` |
| `/blocked` | "what's blocked?" | `dc:frontmatter` |
| `/people <name>` | "tell me about Jane" | `dc:people` |

### System

| Command | Skill |
|---------|-------|
| `/status` | `dc:errors` |
| `/sync` | `dc:sources` |
| `/help` | `dc:commands` |

## Error Handling

Apply these principles across all operations:

1. **Partial success over total failure** — if a source fails, continue with what works
2. **Transparency** — always show what's missing or stale
3. **No data loss** — queue failed writes in `9_Meta/pending.json`, retry automatically
4. **Local-first** — core vault operations work without network
5. **Graceful recovery** — self-heal when connectivity returns

Source freshness thresholds:
- < 30 min: fresh
- 30 min – 4 hours: usable, show "synced X ago"
- \> 4 hours: stale, warn prominently

## Interaction Modes

Two modes defined in config under `behavior`:

- **Suggestion** (default for briefs): Claude prepares, user approves each item
- **Interactive** (default for reviews): Claude guides a step-by-step conversation

Never auto-file, auto-create tasks, or auto-send without explicit user confirmation
unless the user has set `autonomous` mode for that specific operation.

## Implementation Phases

Features are built incrementally. When a feature skill doesn't exist yet, acknowledge it
and offer to help with what's available:

1. **Foundation**: F01 (vault), F14 (bootstrap), F13 (templates)
2. **Capture & Processing**: F02 (capture), F03 (daily note), F04 (frontmatter), F08 (inbox)
3. **Rituals**: F05 (morning brief), F06 (evening review), F07 (weekly review)
4. **Integration**: F10 (sources), F09 (tasks), F15 (people)
5. **Polish**: F11 (commands), F12 (errors), F14 (bootstrap complete)
