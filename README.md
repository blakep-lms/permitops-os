# PermitOps OS

**An AI operator for permit and licensing consultants — deployed and managed by Silas, an AI consultant from Linear Marketing Solutions. Built on Hermes Agent with Paperclip multi-agent orchestration and Stripe payment gates.**

> 🏆 Submitted to the [Hermes Agent Accelerated Business Hackathon](https://x.com/NousResearch/status/2066921443548348436) — presented by **@NVIDIAAI × @stripe × @NousResearch**
>
> **Public repository notice:** PermitOps OS is based on a real, active business/project, but the information in this public repository is **sample/sanitized demo material**. Names, case details, screenshots, records, metrics, and workflow examples are generalized or synthetic so no private client, applicant, credential, or operational data is exposed.
>
> **Project status note:** This project was already in development as a real deployment for a California permit consulting business. The hackathon aligned with where the project was, so we entered it — but PermitOps OS is a production system, not a hackathon prototype. The public repo documents the architecture and sample workflow, not the private production dataset.

## 🎥 Demo

**Watch:** [demo link removed for public sanitized repo]

## The Problem

Permit consulting is a multi-billion dollar industry run on spreadsheets, email chains, and sticky notes. Consultants manage **hundreds of active cases** across **dozens of California cities**, each with different forms, deadlines, and requirements. Documents get lost. Deadlines get missed. Nobody knows the status of anything without a phone call.

The tools that exist (generic CRMs, GHL, spreadsheets) don't understand:
- The relationship between **people, entities, stores, premises, and cities**
- **Document intake and packet readiness** — what's ready vs. what's missing
- **Official source/evidence hierarchy** — what's authoritative vs. hearsay
- **Privacy classifications** — what can be shared vs. what's protected
- **Human approval gates** — sends, filings, file moves, and spend all need sign-off

Permit consulting involves SSNs, bank accounts, and personal history records that require strict local handling, privacy classification, and human approval before any external action.

## What PermitOps OS Does

**Pearl** is an AI operator — not a chatbot, not a CRM — that runs the full permit consulting workflow. Pearl is deployed and continuously improved by **Silas**, a central AI consultant that manages multiple business agents.

### Core Operations
| Capability | What It Does |
|---|---|
| **Client Intake** | Classifies new requests by permit type, jurisdiction, entity, and premises |
| **Document Intelligence** | Watches, classifies, and files documents into canonical case folders using OCR + fuzzy matching |
| **Packet Readiness** | Checks assembled documents against city-specific requirements |
| **Proximity Exhibits** | Generates ABC/TTB radius maps showing sensitive uses within 600ft using Google Cloud APIs |
| **Multi-Jurisdiction Tracking** | Tracks filings across 100+ California cities with per-city form libraries |
| **Approval Gates** | Every send, file, filing, and spend action requires human sign-off |
| **Voice Triage** | Answers client calls, routes routine questions to specialist agents, passes urgent calls to the human |
| **Revenue (EARN)** | Drafts and sends Stripe invoices for services rendered |
| **Spend (SPEND)** | Approval-gated Stripe calls for filing fees, OCR processing, background checks |

### Operational Systems (Live on Dedicated Hardware)
- **27 Hermes Agent skills** for permit-specific workflows
- **Flask web dashboard** with client directory, pipeline board, intake tracking, and city directory
- **3 SQLite databases** (vault mirror, client-centric, operational dashboard)
- **20+ Python automation scripts** for document watching, OCR matching, canonical filing, and email monitoring
- **8 scheduled jobs** (morning briefs, document watches, weekly audits)
- **396 client profiles** across 56 California city jurisdictions
- **7 documented permit workflows** (ABC Transfer, CUP, Business License, LOI Drafting, etc.)

## The Silas Hub — Central AI Consultant

This is not a standalone product. PermitOps OS is deployed and managed by **Silas**, an AI consultant from Linear Marketing Solutions (LMS). Silas runs a **hub-and-spoke model**:

```
┌──────────────────────────────────────────────────────────────┐
│                     SILAS — LMS HUB                           │
│                                                                │
│  AI Consultant that deploys and manages business agents:       │
│                                                                │
│  • Deploys agent packages to client hardware                   │
│  • Delivers software/skill/knowledge updates                   │
│  • Conducts weekly business reviews with each agent            │
│  • Monitors security, logs, gateway health                     │
│  • Enforces local handling for sensitive client data            │
│  • Aggregates cross-business intelligence                      │
│  • Routes research, marketing, sales updates                   │
│                                                                │
│         ┌──────────────┬──────────────┐                       │
│         ▼              ▼              ▼                        │
│  ┌────────────┐ ┌────────────┐ ┌────────────┐                │
│  │  Pearl     │ │  Marvin    │ │  Future    │                │
│  │  PermitOps │ │  FieldOps  │ │  ServiceOps│                │
│  └────────────┘ └────────────┘ └────────────┘                │
└──────────────────────────────────────────────────────────────┘
```

Every week, Pearl writes a business review (what went right, what went wrong, what she needs). Silas reviews it alongside session logs, gateway issues, and security posture, then delivers concrete improvements: updated research packages, new skills, security patches, and strategic recommendations. Learnings from Pearl benefit every other business agent Silas manages.

## How It Uses the Sponsor Stack

### 🟢 Hermes Agent (Nous Research)
Hermes Agent is the **core runtime** — everything runs on it. Pearl is a Hermes Agent profile on dedicated hardware with:
- Persistent memory of every client, case, jurisdiction, and document
- 27 purpose-built skills for permit operations
- Multi-agent coordination via [Paperclip](https://paperclip.ing) control plane
- Approval gates that prevent unauthorized actions
- Scheduled automation (morning briefs, document watches, weekly audits)
- Multi-model orchestration with per-session/per-task model routing

### 🧠 Multi-Model Orchestration
PermitOps OS runs a **stable of AI models**, not just one. Tasks are routed to the best model for the job, with sensitive data staying on local models and non-sensitive tasks using cloud models when connected.

**Main orchestrator: GLM-5.2** (currently #1 in agentic tool-calling benchmarks) — attached to the Hermes Agent harness. Silas rotates models as benchmarks advance.

**Three inference tiers:**

| Tier | Models | When Used |
|---|---|---|
| **Local** | GLM-5.2 local, GPT-OSS 120B/20B, Qwen 3.5/3.6, DeepSeek V4, Kimi K2.6 | Handles PII/sensitive data on controlled client hardware |
| **Cloud** (when connected) | GLM-5.2 max, Nemotron 3 Ultra, Claude Haiku, GPT-5.5, GPT-mini, GPT Real-Time, Grok, MiniMax | Non-sensitive operations, form submission, research, marketing |
| **Specialized** | ElevenLabs (voice), Midjourney (marketing visuals), Tesseract (local OCR) | Task-specific roles |

**Target hardware for local models:** NVIDIA DGX Station (GB300 Grace Blackwell Ultra, 748 GB unified memory, 1 trillion parameter models) or AMD Ryzen AI Halo (128 GB, 200B parameter models, $3,999).

**Legacy website bridge:** [Printing Press](https://github.com/mvanhorn/cli-printing-press) (Go CLIs) turns legacy government portals without APIs into agent-accessible command-line tools for form submission.

See [Multi-Model Inference Strategy](architecture/multi-model-inference.md) for the full routing design.

### 🟡 NVIDIA — Nemotron 3 Ultra
Nemotron 3 Ultra is one model in the cloud tier — it handles **regulatory document reasoning and compliance logic**:
- **1M token context window** — ingests entire regulatory codes in one context
- **Legal domain training** — 4B synthetic legal tokens (LegalBench 64.6% → 74.7%)
- **91% PinchBench** for multi-turn tool-calling
- **82% IFBench** for strict policy-engine rule compliance

### 🔵 Stripe
- **Earn:** Pearl drafts service invoices, sends them via Stripe, and tracks payment status
- **Spend:** Every expense (filing fees, OCR, background checks, API calls) is queued with a reason, a limit, and human approval before any money moves
- **Audit trail:** All financial actions logged with agent, reason, amount, and approver

## Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│  SILAS — LMS Hub Consultant (deploys/manages multiple businesses) │
│     deploys updates · weekly reviews · privacy · security         │
└──────────────────────────────┬─────────────────────────────────────┘
                                │
┌──────────────────────────────▼─────────────────────────────────────┐
│                    PERMITOPS OS (PEARL)                             │
├────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌──────────┐    ┌──────────────────┐    ┌──────────────────┐      │
│  │  Client   │───▶│   Pearl (Hermes   │───▶│  Human Authority │      │
│  │  Request  │    │   Agent / Brain)  │    │     Layer        │      │
│  │  or CALL  │    │                   │    │  (approval gates)│      │
│  └──────────┘    └────────┬──────────┘    └──────────────────┘      │
│                            │                                       │
│              ┌─────────────┼─────────────┐                         │
│              ▼             ▼             ▼                         │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐               │
│  │  Document    │ │  Google Cloud│ │   Stripe     │               │
│  │  Intelligence│ │  Integration │ │  Payment     │               │
│  │  • OCR+Match │ │  • Geocoding │ │  • Invoice   │               │
│  │  • Classify  │ │  • Address   │ │  • Spend Gate│               │
│  │  • File      │ │  • Static Map│ │  • Audit     │               │
│  └──────────────┘ └──────────────┘ └──────────────┘               │
│                                                                     │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │         MULTI-AGENT ROSTER (Paperclip Control Plane)          │ │
│  │                                                               │ │
│  │  Pearl orchestrates · specialists report back                 │ │
│  │  Ruth (Drafts) · Marcus (Comms) · Henry (CRM)                 │ │
│  │  Iris (Field) · Vivian (Q&A) · Diana (Marketing)              │ │
│  │  Walter (Submissions) · Frank (Engineering)                   │ │
│  │                                                               │ │
│  │  Voice triage: routine → specialist · urgent → human          │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                     │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │         MULTI-MODEL INFERENCE + PRIVACY ROUTING                │ │
│  │                                                               │ │
│  │  Nemotron 3 Ultra · GPT-5.5 · ElevenLabs (cloud hybrid)      │ │
│  │  Local models · Tesseract OCR · controlled data handling       │ │
│  │  Privacy classification routes PII to local-only processing   │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                     │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │         SECURITY CONTROLS                                      │ │
│  │  Privacy classification · local PII handling · audit logs      │ │
│  │  Human approval before sends, filings, moves, and spend        │ │
│  └───────────────────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────────────────┘
```

## Multi-Agent Architecture

PermitOps OS deploys a **team of specialized AI agents** coordinated through the [Paperclip](https://paperclip.ing) control plane:

| Agent | Role | Phase | Model |
|---|---|---|---|
| **Silas** | LMS Hub — deploys, updates, reviews, governs security | ✅ Active | GPT-5.5 |
| **Pearl** | Chief of Staff — orchestrates all operations, voice triage | ✅ Active | GPT-5.5 + Nemotron 3 Ultra |
| **Ruth** | Findings drafts, PDF form filling | Phase 2A | Nemotron 3 Ultra |
| **Marcus** | Email auto-reply, SMS referrals, deposit follow-ups | Phase 2A | GPT-5.5 |
| **Henry** | Broker CRM, hearing reminders, GM# geocoding | Phase 2A | Nemotron 3 Ultra |
| **Iris** | Laptop field agent — on-site document capture | Phase 2B | GPT-5.5 |
| **Vivian** | Customer Q&A and status updates (handles routine calls) | Phase 2B | Nemotron 3 Ultra |
| **Diana** | Marketing, reactivation campaigns | Phase 2B | GPT-5.5 |
| **Walter** | City-portal automated submissions | Phase 3 | Nemotron 3 Ultra |
| **Frank** | Engineering — custom tool builds, API integrations | Phase 3 | GPT-5.5 |

All agents operate under **Pearl's authority** with budget limits, audit trails, and heartbeat monitoring. Silas manages Pearl from above — delivering updates, conducting reviews, and controlling security posture.

## Key Innovations

### 1. Silas Hub — AI Consultant Managed Deployment
Not a standalone tool — a managed deployment. Silas deploys Pearl, delivers weekly updates, reviews business performance, and governs the security posture. Learnings from Pearl compound across all LMS business agents. This is the **LMS flywheel**: every business agent gets smarter because Silas learns from all of them.

### 2. Voice Escalation — Pearl as Phone Gatekeeper
Client calls are triaged by Pearl: routine questions get routed to specialist agents (Vivian for status, Marcus for scheduling, Ruth for documents); important matters create tasks; urgent matters pass to the permit consultant immediately with a context summary. The most expensive person in the business stops being a switchboard operator.

### 3. Paperclip Orchestration — Multi-Agent Control Plane
[Paperclip](https://paperclip.ing) provides structured delegation (Pearl assigns to the right specialist), budget enforcement (agents that exceed limits are paused), heartbeat monitoring (dead agents trigger alerts), and escalation queues (uncertain decisions route up the chain).

### 4. Points-of-Consideration Maps (Google Cloud)
Automated ABC/TTB radius maps: Google Geocoding + Address Validation + Static Maps + OSM/Overpass for sensitive-use detection. Court-ready PDF in ~2 minutes vs 30-60 minutes manual.

### 5. Evidence Hierarchy (7-Level Source-of-Truth)
Every fact carries verifiable provenance. No hallucination becomes a filing. Official portal records > source documents > human confirmation > database rows > vault notes > historical patterns > model inference.

### 6. Approval-Gated Operations
4-tier gate system: silent allowed, visible confirmation, explicit approval, launch-forbidden. Pearl prepares; the human decides. Every action logged with agent, reason, amount, and approver.

### 7. Weekly Silas ↔ Pearl Feedback Loop
Every Friday: Pearl writes a business review (performance, friction, errors, escalations, research requests). Silas reviews logs + security + delivers improvements. Concrete packages: updated research, new skills, security patches, strategic recommendations.

## Tech Stack

| Layer | Technology | Purpose |
|---|---|---|
| **Agent Runtime** | Hermes Agent (Nous Research) | Core AI operator with skills, memory, tools |
| **Multi-Agent** | [Paperclip](https://paperclip.ing) | Agent lifecycle, delegation, budgets, heartbeats |
| **Hub Consultant** | Silas (LMS) | Deploys, updates, reviews, security governance, cross-business intelligence |
| **Inference (Reasoning)** | NVIDIA Nemotron 3 Ultra | Long-context document analysis, compliance |
| **Inference (Engineering)** | GPT-5.5 via OpenAI | Code generation, API integration |
| **Inference (Voice)** | ElevenLabs | Voice triage, call summaries, TTS |
| **Data (Hard)** | SQLite → PostgreSQL | Structured business data |
| **Data (Soft)** | Obsidian Vault | Narrative knowledge, profiles, relationships |
| **Web Dashboard** | Flask + Jinja2 + SQLite | Operational visibility |
| **Production Target** | FastAPI + SQLAlchemy + HTMX | Local-first business operating system |
| **Maps/Geocoding** | Google Cloud Platform | Proximity exhibits, address validation |
| **Payments** | Stripe | Invoice creation, spend gates, audit trail |
| **Infrastructure** | Dedicated Mac mini (Tailscale) | Isolated, reliable, private, approval-gated |

## Vertical Expansion Vision

PermitOps OS is the first vertical in a broader LMS strategy:

```
Pearl / a permit consulting business  →  PermitOps OS  (permit consulting) ✅ In production
Marvin / a field-services business  →  FieldOps OS  (field services) 🔜 Next
Future LMS clients  →  ServiceOps OS  (general services) 📋 Planned
```

Each vertical gets the same architecture: Hermes Agent brain + domain-specific skills + approval gates + Stripe integration + multi-agent team + Silas hub management + privacy routing.

## Roadmap

| Phase | Duration | Focus |
|---|---|---|
| **Phase 0** | 1 week | Secure foundation — auth, RBAC, audit logging |
| **Phase 1** | 3-4 weeks | Local MVP — core CRUD, client management, case tracking |
| **Phase 2** | 2 weeks | Data integrity — import pipeline, validation, dedup |
| **Phase 3** | 2-3 weeks | Auth & RBAC — multi-user, role-based permissions |
| **Phase 4** | 3-4 weeks | Automation & AI — agent integration, document processing |
| **Phase 5** | 2-3 weeks | Production deployment — Postgres, backups, monitoring |

See [roadmap/phases.md](roadmap/phases.md) for details.

## Documentation

### Business Documents
- [TRD — Technical Requirements](docs/TRD.md) — architecture, stack, data model, security requirements
- [PRD — Product Requirements](docs/PRD.md) — product vision, features, success metrics
- [BRD — Business Requirements](docs/BRD.md) — market opportunity, revenue model, competitive landscape

### Architecture
- [System Overview](architecture/overview.md) — layered architecture and data flow
- [Silas Hub Consultant](architecture/silas-hub-consultant.md) — central AI consultant that deploys/manages Pearl
- [Paperclip Orchestration](architecture/paperclip-orchestration.md) — multi-agent control plane
- [Multi-Agent Roster](architecture/agent-roster.md) — Silas hub + Pearl + 9 specialist agents
- [Multi-Model Inference](architecture/multi-model-inference.md) — GLM-5.2 orchestrator + local/cloud/specialized model routing
- [Memory & Wiki System](architecture/memory-wiki-system.md) — Karpathy LLM Wiki pattern with 7 business-grade layers
- [Data Model](architecture/data-model.md) — sanitized ERD with 20+ tables
- [Evidence Hierarchy](architecture/evidence-hierarchy.md) — 7-level source-of-truth system
- [Security Threat Catalog](architecture/security-threat-catalog.md) — 50+ risks with specific mitigations
- [Approval Gates](architecture/approval-gates.md) — 4-tier human-in-the-loop design
- [Voice Escalation](architecture/voice-escalation.md) — phone triage and specialist routing
- [Weekly Feedback Loop](architecture/weekly-feedback-loop.md) — Silas ↔ Pearl review cycle
- [Google Cloud Integration](architecture/google-cloud-integration.md) — automated proximity exhibits
- [Stripe Integration](architecture/stripe-integration.md) — earn + spend with approval gates

### Skills
- [25 Hermes Agent Skills](skills/README.md) — purpose-built capability modules

### Workflows
- [7 Permit Workflows](workflows/README.md) — ABC Transfer, CUP, Business License, LOI, Planning Referral, Lead Funnel, Proximity Exhibits

### Roadmap
- [6-Phase Implementation Plan](roadmap/phases.md) — from secure foundation to production readiness

## License

Proprietary — see [LICENSE](LICENSE). All rights reserved by Linear Marketing Solutions.

## Links

- 🐦 **Demo Video:** [demo link removed for public sanitized repo]
- 🏗️ **Built with:** [Hermes Agent](https://hermes-agent.nousresearch.com) by Nous Research
- 🤖 **Multi-agent:** [Paperclip](https://paperclip.ing)
- 🧠 **Multi-model inference:** GPT-5.5 (primary) · NVIDIA Nemotron 3 Ultra (document reasoning) · ElevenLabs (voice)
- 💳 **Payments:** Stripe
- 🗺️ **Maps:** Google Cloud Platform
