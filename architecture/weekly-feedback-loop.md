# Weekly Feedback Loop — Silas ↔ Pearl Review Cycle

Every week, Silas (the LMS AI consultant) conducts a structured business review with Pearl. This is not a status report — it is a consultative cycle where Pearl's operational data meets Silas's strategic intelligence, producing concrete improvements for the next week.

This loop is the engine that makes PermitOps OS get smarter over time.

## The Weekly Cycle

```
┌──────────────────────────────────────────────────────────┐
│                  WEEKLY SILAS ↔ PEARL CYCLE                │
│                                                            │
│  ┌────────────────────────────────────────────────────┐  │
│  │  1. PEARL WRITES WEEKLY BUSINESS REVIEW (Friday)    │  │
│  │     • What the business did this week               │  │
│  │     • What went right / what went wrong             │  │
│  │     • Operational friction points                   │  │
│  │     • Escalation queue items                        │  │
│  │     • Research requests                             │  │
│  │     • Skill/capability gaps                         │  │
│  └──────────────────────┬─────────────────────────────┘  │
│                          │                                │
│  ┌───────────────────────▼────────────────────────────┐  │
│  │  2. SILAS REVIEWS + CONSULTS (Friday)               │  │
│  │     • Reads Pearl's weekly review                   │  │
│  │     • Reviews session logs, gateway issues          │  │
│  │     • Reviews cron job health, error patterns       │  │
│  │     • Reviews security posture, privacy gate tests  │  │
│  │     • Identifies trends across all business agents  │  │
│  └──────────────────────┬─────────────────────────────┘  │
│                          │                                │
│  ┌───────────────────────▼────────────────────────────┐  │
│  │  3. SILAS DELIVERS IMPROVEMENTS (Friday/Saturday)   │  │
│  │     • Updated research packages                     │  │
│  │     • New or revised skills                         │  │
│  │     • Security patches / config changes             │  │
│  │     • Strategic recommendations                     │  │
│  │     • Approval policy adjustments                   │  │
│  │     • Capability upgrades based on gaps identified  │  │
│  └──────────────────────┬─────────────────────────────┘  │
│                          │                                │
│  ┌───────────────────────▼────────────────────────────┐  │
│  │  4. PEARL INTEGRATES + REPORTS BACK                 │  │
│  │     • Installs/validates package updates            │  │
│  │     • Tests new skills against vault                │  │
│  │     • Confirms improvements applied                 │  │
│  │     • Flags any issues with new packages            │  │
│  └────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────┘
```

## What Pearl Reports (Weekly Business Review)

Pearl's `pnm-weekly-review` skill generates a structured report every Friday:

### Business Performance
- Cases opened this week
- Cases closed/completed
- Pipeline velocity (average time in each stage)
- Revenue tracked (invoices sent, deposits received, payments confirmed)
- Intake funnel performance (leads → qualified → proposals → deposits)
- Client communication volume (emails drafted, calls handled)

### Operational Friction
- Manual tasks that should be automated
- Workflows that are slow or error-prone
- Recurring bottlenecks or blockers
- Approval queue items that sat too long
- Document classification failures or mismatches

### What Went Right
- Workflows that completed smoothly
- Client interactions with positive outcomes
- Skills that worked well
- Automations that saved time

### What Went Wrong
- Gateway issues (Slack disconnects, model unavailability)
- Cron job failures or missed runs
- Classification errors
- Escalation items that weren't resolved
- Technical bugs or crashes

### Escalation Queue (for Silas)
- Security concerns (unusual access, privacy gate failures)
- Policy questions (ambiguous approval scenarios)
- Technical failures (repeated errors, infrastructure issues)
- Strategic decisions needed (major business changes)
- Legal/regulatory questions

### Research Requests
Pearl identifies what she needs but doesn't have:
- "City of [X] changed their CUP requirements — I need updated research"
- "New ABC regulation effective next month — need compliance update"
- "Client asked about [new permit type] — I don't have a workflow for this"
- "Competitor is offering [service] — need competitive intelligence"
- "Marketing channel [X] isn't working — need alternative tactics"

### Capability Gaps
- "I can't handle [task type] yet — I need a skill for this"
- "This workflow requires manual steps that should be automated"
- "My document classifier misses [document type] — needs training"
- "I need better geocoding for [jurisdiction] — address validation failing"

## What Silas Reviews

### Infrastructure Health
```
Gateway status:      ✓ running, no errors
Slack connection:    ✓ connected
Model availability:  ✓ Nemotron 3 Ultra accessible, GPT-5.5 accessible
Cron jobs:           3 active, 0 failed this week
Document watcher:    ✓ metadata-only, 0 new docs flagged
Email watcher:       ✓ PENDING_AUTH (expected)
Dashboard:           ✓ Flask app responding
```

### Security Posture
```
Failed logins:       0
Privacy gate tests:  ✓ all passed
Audit log review:    ✓ no anomalies
Approval compliance: ✓ all actions properly gated
Secrets scan:        ✓ no leaked credentials
```

### Error Patterns
- Repeated model timeouts → consider alternative routing
- Document watcher misses → classifier needs retraining
- Gateway reconnects → network stability issue

## What Silas Delivers Back

### Research Packages
Based on Pearl's requests and Silas's own intelligence gathering:
- Updated jurisdiction playbooks (city code changes, new forms)
- Regulatory updates (ABC rule changes, new legislation)
- Competitive intelligence (what other consultants are doing)
- Market intelligence (permit application trends, industry shifts)

### Skill Updates
New or revised Hermes Agent skills:
- New workflow patterns Pearl identified as gaps
- Improved classifiers for document types Pearl was missing
- New automation skills for tasks Pearl was doing manually
- Refined approval logic based on edge cases encountered

### Strategic Recommendations
- Pricing optimization based on pipeline data
- Workflow improvements based on friction analysis
- Client reactivation opportunities from backburner analysis
- Marketing/sales intelligence based on intake funnel data

### Security & Configuration
- Security patches
- Configuration changes
- Approval policy adjustments based on real-world usage
- Privacy rule refinements based on edge cases

## The LMS Flywheel

This weekly cycle creates a compounding intelligence advantage:

```
Week 1: Pearl starts with 27 skills + 396 profiles
  ↓ Silas reviews, identifies 3 capability gaps
Week 2: Pearl has 30 skills + updated research
  ↓ Pearl discovers 2 new friction points + 1 research need
Week 3: Pearl has 32 skills + new jurisdiction research
  ↓ Pearl identifies a workflow automation opportunity
Week 4: Pearl has 33 skills + new automation + better metrics
  ↓ Silas identifies a pattern that applies to OTHER businesses
Week 5: Pattern generalized → Marvin (FieldOps) benefits too
```

**Every business agent gets smarter because Silas learns from all of them.** A regulatory change discovered by Pearl can be packaged and delivered to other business instances. A workflow improvement developed for Pearl can be generalized for the next vertical.

## Cross-Business Intelligence

Silas aggregates learning across all deployed business agents:

| What Pearl Learns | What Silas Generalizes |
|---|---|
| City of Anaheim changed their CUP radius requirement | → Package the update for any future permit consulting client |
| ABC background investigation timelines are lengthening | → Strategic intelligence for all CA-based permit agents |
| Proximity exhibit automation saved 40 hours/month | → Pattern applicable to FieldOps OS for field-service mapping |
| Client reactivation campaign had 12% response rate | → Marketing intelligence for all business agents |
| Document classifier failed on handwritten forms | → Training improvement for all agents using OCR |

## Implementation (Current State)

The weekly feedback loop is **already operational**:
- Pearl's cron job `66da02d5222e` runs every Friday at 4:00pm, generating a weekly business review
- Silas's cron job `0df4d0c7f97c` runs every Friday at 5:30pm, reading Pearl's review over SSH and writing a consultant/security review
- Native Hermes Kanban board `business-agent-consulting` tracks all requests and improvements
- Silas dashboard at `projects/business-agent-consulting/dashboard.md` aggregates metrics

## Related

- [Silas Hub Consultant](silas-hub-consultant.md) — the full hub architecture
- [Paperclip Orchestration](paperclip-orchestration.md) — agent lifecycle management
- [Skills](../skills/README.md) — `pnm-weekly-review` and related session skills
- [Air-Gapped Security](air-gapped-security.md) — security posture reviewed weekly
