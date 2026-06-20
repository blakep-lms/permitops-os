# Roadmap — 6-Phase Implementation Plan

Derived from the Silas-approved PNM Business Operating System TRD. Each phase has explicit security gates that must pass before the next phase begins.

## Phase Overview

| Phase | Duration | Focus | Status |
|---|---|---|---|
| **Phase 0** | 1 week | Secure foundation — auth, RBAC, audit logging | In planning |
| **Phase 1** | 3-4 weeks | Local MVP — core CRUD, client management, case tracking | Pending Phase 0 |
| **Phase 2** | 2 weeks | Data integrity — import pipeline, validation, dedup | Pending Phase 1 |
| **Phase 3** | 2-3 weeks | Auth & RBAC — multi-user, role-based permissions | Pending Phase 2 |
| **Phase 4** | 3-4 weeks | Automation & AI — agent integration, document processing | Pending Phase 3 |
| **Phase 5** | 2-3 weeks | Production deployment — Postgres, backups, monitoring | Pending Phase 4 |

**Current state:** Pearl is active on dedicated hardware with 27 skills, a Flask dashboard, 396 client profiles, operational proximity exhibits, and automated document/email watchers. The TRD defines the path from "lean local prototype" to "secure local-first business operating system."

---

## Phase 0 — Secure Foundation & Implementation Package (1 week)

**Goal:** Convert the TRD into implementation-ready artifacts.

### Required Artifacts
1. **Threat model** — assets, trust boundaries, actors, abuse cases, mitigations
2. **Source hierarchy + privacy spec** — path/content exclusions and PII classes
3. **Stack decision record** — FastAPI/SQLAlchemy/HTMX + SQLite-vs-PostgreSQL runtime split
4. **Data model v0 ERD** — cases, packets, approvals, review flags, form library, inbox items, activity log, AI runs
5. **Approval policy table** — silent allowed / visible confirmation / explicit approval / forbidden
6. **First workflow specs:**
   - Existing-client pull-up + inconsistency audit
   - ABC packet assembly
   - Inbox-to-case draft response
7. **Security tooling baseline** — ruff, pytest, bandit, pip-audit, gitleaks/detect-secrets, pinned dependencies
8. **Clean-room GHL research note** — module map only, no proprietary UI/code copying

### Security Gate
- [ ] Threat model reviewed
- [ ] No secrets in repo/vault
- [ ] Pre-commit hooks installed and passing
- [ ] Privacy exclusion tests designed
- [ ] Runtime DB decision recorded
- [ ] Approval policy accepted by stakeholders before real data/actions

---

## Phase 1 — Local Core MVP (3-4 weeks)

**Goal:** Hardened local-first core with real domain objects.

### Deliverables
- FastAPI scaffold, config, middleware, logging, error handling
- SQLAlchemy models and Alembic migrations
- DB runtime chosen per Phase 0 decision
- **the permit consultant Today / Pearl Home** — the first screen that answers "what needs my attention?"
- Client/case directory with unified search
- Case pipeline with drag-and-drop stage transitions
- Per-case tasks with activity log
- Packet workspace with checklist
- Intake pipeline (New → Qualifying → Proposal → Deposit → Engaged/Not a Fit)
- City directory + forms library
- Privacy enforcement tests
- Local auth if real data is present

### Security Gate
- [ ] Pydantic validation on all routes
- [ ] File serving/upload path traversal tests pass
- [ ] Private files absent from all API/search/AI responses
- [ ] Activity log captures task moves, case changes, packet actions, file access, draft creation
- [ ] Dependency scan has no high/critical CVEs

---

## Phase 2 — Data Integrity, Backups & Hardening (2 weeks)

**Goal:** Ensure data correctness, recoverability, and security hardening.

### Deliverables
- Backup/restore plan and monthly restore test
- At-rest encryption decision implemented
- Migration/export tooling
- Audit log UI
- Settings UI for stages/license types/tags
- Security headers (CSP, HSTS, etc.)
- Internal scan with bandit, semgrep, pip-audit

### Security Gate
- [ ] Backup encrypted and restore-tested
- [ ] No high/critical static-analysis findings
- [ ] Security headers present
- [ ] Form currency gate works (blocks packets with stale forms)

---

## Phase 3 — Authentication, Authorization & Multi-User (2-3 weeks)

**Goal:** Role-based access control for the system.

### Deliverables
- User management
- Session auth / Argon2 / optional passkeys
- RBAC roles: Owner, Admin, Operator, Viewer, Pearl Service
- Tenant/user scoping
- Login rate limits
- Pearl scoped service account (revocable)

### Security Gate
- [ ] Auth required on all non-public routes
- [ ] Route and service auth tests pass
- [ ] Rate limits verified
- [ ] Service account permissions scoped and revocable

---

## Phase 4 — Automation, AI & Integrations (3-4 weeks)

**Goal:** Connect Pearl's AI capabilities to the hardened core.

### Deliverables
- Code-defined workflow engine (no user-editable expression engine yet)
- Gmail draft tracking after OAuth scope review
- Calendar sync after scope review
- Pearl assistant source-scoped retrieval
- Prompt sandbox and output sanitizer
- Approval broker (connects AI output to approval gates)

### Code-Defined Automations (Phase 4 Start Set)
- Stage change → suggested next task
- Overdue task → digest item
- New intake → draft checklist
- Stale form → review flag
- Inbox item → case-link review
- Missing evidence → packet review flag

### Security Gate
- [ ] Pearl cannot execute external/destructive actions without approval
- [ ] Every AI run logged with hash, scope, actor, tool calls
- [ ] OAuth scopes minimal and reviewed
- [ ] Webhooks signed (HMAC + timestamp + replay defense)
- [ ] Automation rules code-reviewed and versioned

---

## Phase 5 — Production Readiness (2-3 weeks)

**Goal:** Remote/multi-user access if required. Only proceeds if remote access is explicitly needed.

### Deliverables
- PostgreSQL migration if not already used
- Reverse proxy / TLS / VPN / authenticated access
- Monitoring and alerting
- CI/CD pipeline
- Disaster recovery runbook
- Penetration/security review
- Stakeholder signoff

### Security Gate
- [ ] TLS/VPN/authenticated access enforced
- [ ] OWASP + domain-specific checklist addressed
- [ ] DR restore tested
- [ ] Pen-test findings remediated or risk-accepted
- [ ] Stakeholder signoff before go-live

---

## Multi-Agent Rollout (Parallel Track)

Agent specialist deployment runs in parallel with the core platform phases:

| Agent | Role | Target Phase | Status |
|---|---|---|---|
| **Pearl** | Chief of Staff / Orchestrator | Active | ✅ Operational |
| **Ruth** | Drafting & Forms | Phase 2A | Designed |
| **Marcus** | Communications | Phase 2A | Designed |
| **Henry** | CRM & Broker Relations | Phase 2A | Designed |
| **Iris** | Field Agent | Phase 2B | Concept |
| **Vivian** | Customer Q&A | Phase 2B | Concept |
| **Diana** | Marketing & Reactivation | Phase 2B | Concept |
| **Walter** | Submissions | Phase 3 | Concept |
| **Frank** | Engineering | Phase 3 | Concept |

See [Multi-Agent Architecture](../architecture/agent-roster.md) for the full Paperclip roster.

## Vertical Expansion

PermitOps OS is the first vertical in a broader pattern:

```
Pearl / a permit consulting business  →  PermitOps OS  (permit consulting) ✅ In production
Marvin / a field-services business  →  FieldOps OS  (field services) 🔜 Next
Future LMS clients  →  ServiceOps OS  (general services) 📋 Planned
```

Each vertical gets the same architecture: Hermes Agent brain + domain-specific skills + approval gates + Stripe integration + multi-agent team.

## Related

- [Architecture Overview](../architecture/overview.md)
- [Data Model](../architecture/data-model.md)
- [Approval Gates](../architecture/approval-gates.md)
- [Multi-Agent Architecture](../architecture/agent-roster.md)
