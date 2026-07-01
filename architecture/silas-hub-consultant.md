# Silas Hub — Central AI Consultant Architecture

Silas is the AI consultant for Linear Marketing Solutions (LMS). Silas deploys, manages, updates, and reviews business-specific AI agents across multiple client businesses. Pearl is the first deployed agent; PermitOps OS is the first vertical.

This document describes the hub-and-spoke model where Silas serves as the central intelligence that keeps every business agent current, secure, and improving.

## The Model

```
┌──────────────────────────────────────────────────────────────┐
│                      LINEAR MARKETING SOLUTIONS               │
│                                                                │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  SILAS — AI Consultant (Hub)                             │ │
│  │                                                          │ │
│  │  Capabilities:                                           │ │
│  │  • Deploys business agents to client hardware            │ │
│  │  • Delivers software/skill/knowledge package updates     │ │
│  │  • Conducts weekly business reviews with each agent      │ │
│  │  • Provides research, marketing, sales intelligence      │ │
│  │  • Monitors security, logs, gateway health               │ │
│  │  • Can local-only isolation any business from the internet            │ │
│  │  • Manages escalation queues from business agents        │ │
│  └──────────────────────────────────────────────────────────┘ │
│                          │ │ │ │                                │
│           ┌──────────────┘ │ │ └──────────────┐                 │
│           ▼                ▼ ▼                ▼                 │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐            │
│  │  Pearl      │  │  Marvin     │  │  Future     │            │
│  │  PNM        │  │  Install    │  │  Agents     │            │
│  │  (the permit consultant)  │  │  Pros       │  │  (TBD)      │            │
│  │             │  │             │  │             │            │
│  │  PermitOps  │  │  FieldOps   │  │  ServiceOps │            │
│  │  9-agent    │  │  TBD-agent  │  │  TBD-agent  │            │
│  │  team       │  │  team       │  │  team       │            │
│  └─────────────┘  └─────────────┘  └─────────────┘            │
│                                                                │
│  Each business instance:                                       │
│  • Runs on dedicated client hardware                          │
│  • Has its own vault, memory, skills, and identity            │
│  • Is isolated from other businesses                          │
│  • Reports up to Silas for review/updates                     │
└──────────────────────────────────────────────────────────────┘
```

## What Silas Does for Pearl

### 1. Package Deployment & Updates

Silas delivers updates to Pearl on client hardware (the permit consultant's Mac mini):

- **Skill updates** — new or revised Hermes Agent skills (e.g., regulatory changes, new workflow patterns)
- **Knowledge packages** — updated research on jurisdictions, city requirements, market intelligence, competitive landscape
- **Software patches** — dashboard updates, security fixes, automation improvements
- **Model configuration** — inference routing changes (e.g., Nemotron 3 Ultra availability changes)

Updates flow through a **controlled delivery pipeline**, not live internet access:
```
Silas builds package → validates → transfers via secure channel
  → Pearl mini receives → Silas verifies installation → Pearl confirms
```

### 2. Weekly Business Review (Silas ↔ Pearl)

Every week, Silas conducts a structured review with Pearl:

**What Silas reviews:**
- Session logs — what Pearl did, what went well, what went wrong
- Gateway issues — Slack/email connectivity, model availability, API errors
- Cron job health — morning briefs, document watchers, weekly audits
- Escalation queue — items Pearl flagged for Silas/the permit consultant attention
- Error patterns — repeated failures, stuck workflows, blocked approvals
- Security posture — any unauthorized access attempts, privacy gate tests

**What Pearl reports:**
- Business performance — cases opened/closed, revenue tracked, pipeline velocity
- Operational friction — what's slow, what's manual, what needs automation
- Skill gaps — what Pearl can't do yet but needs to
- Research requests — jurisdiction changes, new permit types, market intelligence needed
- Marketing/sales intelligence — lead funnel performance, reactivation opportunities, competitive threats
- Client feedback patterns — what clients ask for, what frustrates them

**What Silas delivers back:**
- Updated research packages (city code changes, new forms, regulatory updates)
- New or revised skills based on identified gaps
- Strategic recommendations (pricing optimization, workflow improvements)
- Security patches or configuration changes
- Approval policy adjustments

See [Weekly Feedback Loop](weekly-feedback-loop.md) for the full cycle.

### 3. Security & Privacy Governance

Silas is responsible for the security posture of every business agent deployment. This includes the ability to **fully local-only isolation** a business from the internet when client data sensitivity requires it.

#### Local Sensitive-Data Capability

Permit consulting involves extremely sensitive data:
- Social Security Numbers (SSN)
- Bank account and financial information
- Tax identification numbers (TIN/EIN)
- Personal history records (ABC Form 407)
- Fingerprint card data
- Business ownership documents

For maximum security, Silas can configure a business instance to run **completely offline**:

```
┌──────────────────────────────────────────────────────┐
│  CONTROLLED BUSINESS INSTANCE                         │
│                                                       │
│  ┌──────────────────────────────────────────────┐    │
│  │  Dedicated Hardware (Mac mini)               │    │
│  │                                               │    │
│  │  • Local LLM inference (no cloud API calls)  │    │
│  │  • Local document storage (no cloud sync)    │    │
│  │  • No internet connection required           │    │
│  │  • Silas delivers updates via physical       │    │
│  │    transfer or brief managed connection      │    │
│  └──────────────────────────────────────────────┘    │
│                                                       │
│  Silas updates:                                       │
│  • Periodic physical USB package delivery             │
│  • Or brief authenticated Tailscale connection        │
│    for verified package transfer only                 │
│  • No persistent internet exposure                    │
└──────────────────────────────────────────────────────┘
```

#### Local Inference for Local-Only Mode

In fully local-only isolationped mode, PermitOps OS falls back to local models:
- **Nemotron 3 Ultra (local)** — if NVIDIA hardware is available locally
- **Local Mamba/Transformer models** — for document reasoning without internet
- **Reduced capability mode** — the system degrades gracefully: all approval gates still work, all vault operations work, but inference quality may be lower than cloud-routed models

Silas manages the tradeoff: **cloud inference = better quality, local inference = better security**. For the most sensitive client data (SSNs, bank accounts, personal history records), the system defaults to local processing only, even when internet is available.


### 4. Escalation Routing

When a business agent encounters something beyond its authority, it escalates to Silas:

```
Pearl encounters:
  • Security concern (potential data breach, unauthorized access)
  • Policy question (ambiguous approval gate scenario)
  • Technical failure (repeated gateway crashes, model unavailability)
  • Strategic decision (major business change needed)
  • Legal question (regulatory uncertainty)
  ↓
Pearl routes to Silas escalation queue
  ↓
Silas reviews, decides, and routes back:
  (a) Resolution instructions to Pearl
  (b) Escalation to the LMS owner (LMS owner) if human decision needed
  (c) Emergency local-only isolation if security breach suspected
```

### 5. Multi-Business Intelligence

Silas aggregates learning across all deployed business agents:

- A regulatory change discovered by Pearl in one jurisdiction can be packaged and delivered to other business instances
- A workflow improvement developed for Pearl can be generalized for Marvin/FieldOps
- Marketing intelligence from one business informs strategy recommendations for others
- Security threats detected anywhere trigger protective measures everywhere

This is the **LMS flywheel**: every business agent gets smarter because Silas learns from all of them.

## Deployment Model

```
Silas (LMS Hub) — runs on the LMS owner's primary infrastructure
  │
  ├── Deploys Pearl → the permit consultant's Mac mini (Tailscale-connected)
  │     ├── PNM-Vault (Obsidian)
  │     ├── 27 skills
  │     ├── 9-agent Paperclip roster (phased)
  │     ├── Document/Email watchers
  │     ├── Flask dashboard
  │     └── Weekly cron: report to Silas
  │
  ├── Deploys Marvin → a field-services business hardware (future)
  │     ├── IPS-Vault
  │     ├── Field-service skills
  │     └── Weekly cron: report to Silas
  │
  └── Future agents → Future client hardware
        └── Same pattern
```

## Trust Boundaries

| Layer | Can Access | Cannot Access |
|---|---|---|
| **Silas** | Package delivery, log review, security posture, aggregate metrics | Client PII, case details, financial data (unless escalated) |
| **Pearl** | All PNM business data within approved scope | Other businesses' data, Silas infrastructure |
| **Specialists** | Scoped task data only | Full case history, other specialists' work, raw PII |
| **Client Hardware** | Local files, local models, local vault | Internet (in local-only mode), Silas infrastructure |

## Related

- [Paperclip Orchestration](paperclip-orchestration.md) — the control plane Silas uses
- [Weekly Feedback Loop](weekly-feedback-loop.md) — the structured Silas ↔ Pearl review cycle
- [Agent Roster](agent-roster.md) — the full team Silas manages
- [Vertical Expansion Vision](../README.md#vertical-expansion-vision) — PermitOps → FieldOps → ServiceOps
