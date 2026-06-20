# Hermes Agent Skills — PermitOps OS

Pearl operates using **25 purpose-built Hermes Agent skills** for permit consulting workflows. These are not generic AI prompts — they are structured, versioned capability modules with defined inputs, outputs, vault reads/writes, and approval boundaries.

This directory documents each skill's purpose at a sanitized level. No client data, no source code, no proprietary business logic is included.

## Skill Catalog

### Core Operations

| Skill | Purpose |
|---|---|
| **pnm-intake** | New-client intake: extract fields from requests, disambiguate against existing cases, create vault note + folder scaffold, draft service agreement, sync databases |
| **pnm-disambiguate** | Entity/case disambiguation: match owner names, entities, DBAs, premises addresses, and store numbers to existing cases. Critical for the "4 Mohammeds" problem |
| **pnm-document-classify** | Incoming document classification: detect document type and address, match to case, file in canonical folder, update case note |
| **pnm-filing-status** | Filing status tracking: update and query status of CUP, ABC, business license, and other filings per case |
| **pnm-proximity-exhibits** | Generate branded ABC/TTB proximity screening exhibits: verify addresses, map consideration points, generate printable PDFs, preserve approval gates |

### Communication & Intake

| Skill | Purpose |
|---|---|
| **pnm-email-watcher** | Monitor inbox for new messages, classify by case relevance, draft replies (approval-gated before sending) |
| **pnm-city-outreach** | Draft zoning verification requests, track city planner/investigator contacts and responses |
| **pnm-loi-draft** | Letter of Intent drafting: generate city-specific LOI from case data, jurisdiction playbook, and boilerplate. Human approves before sending |
| **pnm-deposit-tracker** | Flag unsigned service agreements and outstanding deposits by follow-up stage |
| **pnm-google-workspace** | Google Workspace integration: read/draft emails, manage calendar, access Drive documents (OAuth-gated, read/draft only) |

### Daily Operations & Briefing

| Skill | Purpose |
|---|---|
| **pnm-morning-briefing** | Morning standup: scan vault, review queues, CRM, and watcher status to produce daily brief and dispatch autonomous work |
| **pnm-hearing-prep** | Hearing preparation: track checklist status for upcoming planning commission or ABC hearings and flag missing items |
| **pnm-remember** | Knowledge capture: detect memorable facts in conversation and write to correct vault note with source tags and timestamps |
| **pnm-weekly-review** | Weekly review: summarize the week, escalate stale items, recommend next-week priorities |

### Dashboard & Development

| Skill | Purpose |
|---|---|
| **pnm-dashboard** | Flask web dashboard: client directory, pipeline board, intake tracking, city directory |
| **pnm-dashboard-development** | Dashboard feature development: extend the Flask app with new views, routes, and data models |
| **pnm-wiki-operations** | Obsidian vault operations: manage wikilinks, frontmatter, tags, naming conventions, and vault structure |

### Project & Growth

| Skill | Purpose |
|---|---|
| **pnm-project** | Create and manage internal projects with PRD/BRD/TRD docs, goals, decisions, and session handoffs in the vault |
| **pnm-agent-orchestration** | Evaluate, plan, and integrate agent orchestration platforms (e.g., Paperclip) while preserving local-first boundaries and human approval gates |
| **pnm-autonomous-growth** | Master playbook for growing the business autonomously and alongside the human operator: daily ops, intake, research, monitoring, continuous improvement |

### Session Rituals

| Skill | Purpose |
|---|---|
| **pnm-stand-up** | Morning stand-up: orient from vault, journal, todos, blockers, and review queues |
| **pnm-journal** | Obsidian journaling: append chronological truth, ingest notes safely, preserve sources, keep candidate vs approved facts separate |
| **pnm-end-of-day** | End-of-day journaling: capture what happened, decisions made, artifacts created, and open questions |
| **pnm-wrap-up** | Close a general/day session: journaling, update open items, surface blockers, prepare tomorrow's stand-up |
| **pnm-session-handoff** | Close a work session: update journal, next-session resume, review queues, blockers, and exact next moves |

## Skill Design Pattern

Every PNM skill follows a consistent structure:

```yaml
---
name: pnm-<capability>
description: "<one-line purpose>"
version: "1.0.0"
tags: [pnm, <domain>]
---

# Skill Body
## When to Use
## Steps
## Vault Reads
## Vault Writes
## Approval Gates
## Safety Boundaries
```

Key design principles:
- **Source-scoped** — skills only see data they're permitted to access
- **Output validation** — all outputs are checked before action
- **Uncertainty labeling** — unknowns are tagged, not guessed
- **Audit trail** — every skill execution logs what was read, what was written, and why

## Custom Skill Creation Policy

New skills must be reviewed and approved before activation on Pearl's hardware. The approval process ensures:
1. No unauthorized external API access
2. No private/PII data exposure
3. Approval gates are present for risky actions
4. Vault writes follow the naming/frontmatter conventions
5. The skill is testable and reversible

## Related

- [Architecture Overview](../architecture/overview.md)
- [Workflows](../workflows/README.md)
- [Multi-Agent Architecture](../architecture/agent-roster.md)
