# Technical Requirements Document (TRD)
## PermitOps OS — PNM Business Operating System

**Version:** 1.0 (sanitized for public repo)
**Status:** Approved for Phase 0 planning
**Source:** Derived from the Silas-approved PNM Business Operating System TRD. All client-specific data, PII, and proprietary business logic removed.

---

## 1. Executive Summary

PermitOps OS is a secure, local-first, source-grounded business operating system for permit and liquor-license consultants. It is inspired by the CRM/pipeline/operations model of modern SaaS platforms, but it is purpose-built for the specific realities of permit consulting:

- Source-grounded case and client memory with verifiable provenance
- Disambiguation across people, entities, premises, DBAs, stores, cities, and workflows
- Forms-library currency and jurisdiction-specific requirements tracking
- Packet/final-deliverable preparation with evidence receipts
- Review flags and human approval gates before any risky action
- Daily operating views designed to reduce cognitive load

The system is deployed and managed by Silas (an AI consultant from Linear Marketing Solutions) and runs on dedicated client hardware with configurable security tiers, including full air-gap capability.

## 2. Architecture

### 2.1 Logical Layers

```
Presentation Layer
  - Jinja2/HTMX server-rendered UI
  - the permit consultant Today / Needs Review / Case Pipeline
  - Record drawers for case, client, entity, premises, city
  - CSRF tokens, CSP, secure cookies

API / Application Layer
  - FastAPI
  - Pydantic request/response validation
  - Auth middleware, RBAC, rate limiting
  - Service layer for source hierarchy and approval policy

Data Access Layer
  - SQLAlchemy 2.0 ORM
  - Alembic migrations
  - Tenant/user scoping helpers

Data Stores
  - Prototype/dev: SQLite
  - Production target: PostgreSQL
  - Obsidian vault (soft knowledge)
  - Canonical file tree
  - Quarantine/private folders excluded from ingestion

AI / Agent Layer
  - Pearl via Hermes Agent profile/runtime
  - Prompt sandbox and source-scoped retrieval
  - Output sanitizer and approval broker
  - ai_runs audit log

External Integrations (opt-in)
  - Intake relay (separate device) — encrypted client document submission
  - Google Workspace (read/draft after OAuth review)
  - Optional calendar sync
  - Future city/ABC status webhooks (signed)
  - Stripe (invoices + spend gates)
  - Google Cloud (proximity exhibits)
```

### 2.2 Multi-Agent Layer

```
Silas (LMS Hub Consultant)
  ↓ deploys / updates / reviews
Pearl (Business Orchestrator)
  ↓ delegates via Paperclip
Specialist Agents (Ruth, Marcus, Henry, Iris, Vivian, Diana, Walter, Frank)
```

### 2.3 Voice Escalation Layer

```
Inbound call → Pearl (Voice AI) → Triage
  → Routine: delegate to specialist
  → Important: message + task
  → Urgent: pass to human immediately with context
```

### 2.4 Security Tiers

| Tier | Description | Cloud APIs | Network |
|---|---|---|---|
| Hybrid (default) | Sensitive data local, non-sensitive may use cloud | Selective | Tailscale VPN |
| Local-only | All inference local, no cloud API calls | None | Tailscale for updates only |
| Full air-gap | Zero network connectivity | None | None (physical package delivery) |

## 3. Recommended Stack

| Layer | Technology | Rationale |
|---|---|---|
| Language | Python 3.11+ | Existing automation scripts, team familiarity |
| Web framework | FastAPI | Pydantic validation, OpenAPI, clean services |
| ORM / migrations | SQLAlchemy 2.0 + Alembic | Enterprise schema versioning, Postgres path |
| Frontend | HTMX + Jinja2 + vanilla JS | CPU-light, accessible, no SPA overhead |
| Database | SQLite (dev) → PostgreSQL (production) | Local-first speed + production durability |
| Auth | Local session auth (Argon2) | Required if real data present |
| Secrets | macOS Keychain / env (gitignored) | Never commit secrets |
| File serving | ID-based app route | No raw path params |
| Static analysis | ruff, bandit, semgrep, gitleaks | Security baseline |
| Testing | pytest + coverage + Playwright | Unit/integration/UI |
| Agent runtime | Hermes Agent (Nous Research) | Skills, memory, multi-agent, approval gates |
| Doc reasoning | NVIDIA Nemotron 3 Ultra | 1M context, legal domain training |
| Code/engineering | GPT-5.5 | Code generation, API integration |
| Voice | ElevenLabs | Voice synthesis for phone/demo |
| Maps/geocoding | Google Cloud Platform | Proximity exhibits, address validation |
| Payments | Stripe | Invoice creation, spend gates, audit trail |
| Infrastructure | Dedicated Mac mini + Tailscale | Isolated, reliable, private |
| Orchestration | Paperclip | Multi-agent control plane |
| Management | Silas (LMS Hub) | Package delivery, weekly reviews, security |

## 4. Data Model v0

See [architecture/data-model.md](../architecture/data-model.md) for the full sanitized ERD with 20+ tables covering: users/tenants, contacts/entities/premises/business_locations/cities/relationships, cases/pipelines/tasks/packets/approvals/review_flags, forms/form_versions/source_evidence/privacy_classifications, intakes/inbox_items/email_drafts/activity_log/ai_runs.

## 5. Source-of-Truth Hierarchy

See [architecture/evidence-hierarchy.md](../architecture/evidence-hierarchy.md) for the 7-level system. Every fact carries: source_type, source_path_or_url, captured_at, captured_by, confidence, privacy_classification.

## 6. Security Requirements

### 6.1 Secrets
- No secrets in repo, vault, logs, prompts, or frontend
- `.env` gitignored; `.env.example` placeholders only
- Pre-commit gitleaks/detect-secrets

### 6.2 Authentication
- Local session auth required if real data present
- Roles: Owner, Admin, Operator, Viewer, Pearl Service
- Default deny; route + service RBAC

### 6.3 Data Security
- DB files `0600` in SQLite mode
- FileVault + encrypted backups
- Sensitive source docs in quarantined folders; dashboard stores references only
- Privacy classification enforcement (testable)

### 6.4 AI/LLM Security
- Prompt injection controls (source isolation, escaped input, output validation)
- Least-data prompts
- Approval broker for excessive agency
- Every AI run logged with hash, scope, actor
- No auto-execution of generated code

### 6.5 Air-Gap Capability
- Configurable: hybrid → local-only → full air-gap
- Local model stack for offline inference
- Physical package delivery for updates in air-gap mode
- See [architecture/air-gapped-security.md](../architecture/air-gapped-security.md)

## 7. Phasing Plan

See [roadmap/phases.md](../roadmap/phases.md) for the full 6-phase plan:
- Phase 0: Secure foundation (1 week)
- Phase 1: Local core MVP (3-4 weeks)
- Phase 2: Data integrity & hardening (2 weeks)
- Phase 3: Auth & RBAC (2-3 weeks)
- Phase 4: Automation, AI & integrations (3-4 weeks)
- Phase 5: Production readiness (2-3 weeks)

## 8. Management & Feedback Loop

- **Silas (LMS Hub)** deploys, updates, and reviews Pearl weekly
- Weekly business review: performance, friction, errors, escalations, research requests
- Silas delivers: research packages, skill updates, security patches, strategic recommendations
- Cross-business intelligence: learnings from Pearl benefit future agents (Marvin/FieldOps, etc.)
- See [architecture/weekly-feedback-loop.md](../architecture/weekly-feedback-loop.md)

## 9. Open Questions

| Question | Required Before |
|---|---|
| SQLite-only first release or PostgreSQL? | Phase 0 exit |
| Does Phase 1 handle real data? (determines auth requirement) | Phase 0 exit |
| At-rest baseline: FileVault+0600 or SQLCipher/Postgres encryption? | Phase 0 exit |
| Which first validation case/email/form set drives the MVP? | Phase 1 start |
| Which external integrations are approved for read/draft mode? | Phase 4 start |
| Which security tier is the default for each deployment? | Per-deployment |

## 10. Decision Log

| Decision | Rationale |
|---|---|
| Custom application over GHL/open-source CRM fork | Permit workflows, evidence hierarchy, and approval gates require purpose-built system |
| GHL as clean-room UX reference only | Borrow patterns, never proprietary implementation |
| FastAPI + SQLAlchemy + HTMX | Low CPU, Python-aligned, migration path |
| Human-in-the-loop AI non-negotiable | Protects against prompt injection, PII leakage, filing errors |
| Silas hub-and-spoke management model | Enables multi-business deployment, cross-business intelligence, air-gap management |
| Air-gap capability from day one | Client data sensitivity (SSN, bank, PII) requires offline-capable architecture |
| Paperclip for multi-agent orchestration | Structured delegation, budget enforcement, heartbeat monitoring, audit trail |
