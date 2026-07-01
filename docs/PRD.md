# Product Requirements Document (PRD)
## PermitOps OS — Permit Consulting AI Operating System

**Version:** 1.0 (sanitized for public repo)
**Status:** Active development
**Source:** Derived from the PNM product requirements. All client-specific data removed.

---

## 1. Product Vision

PermitOps OS is an AI operator — not a chatbot, not a CRM — that runs the full back-office workflow of a permit and liquor-license consulting business. It manages client intake, document intelligence, case pipelines, packet readiness, jurisdiction-specific requirements, and revenue operations — all behind human approval gates.

The system is deployed and managed by Silas, an AI consultant from Linear Marketing Solutions, which runs multiple business instances using the same architecture pattern.

## 2. Target Users

### Primary: The Permit Consultant (the permit consultant)
- 30-year veteran managing hundreds of active cases
- Handles ABC transfers, CUPs, business licenses, TTB permits across 50+ California cities
- Needs to know what needs attention TODAY without digging through files
- Cannot afford a filing mistake — her reputation depends on accuracy
- Wants AI to handle the repetitive work so she can focus on client relationships and closing deals

### Secondary: The AI Consultant / Deployer (Silas / LMS)
- Deploys and manages PermitOps OS instances for client businesses
- Conducts weekly reviews, delivers updates, monitors security
- Generalizes learnings across multiple business deployments

### Tertiary: Clients of the Permit Consultant
- 7-Eleven franchisees, restaurant owners, gas station operators
- Want timely status updates and smooth filing experiences
- Need confidence that their sensitive data (SSN, bank info) is protected

## 3. Core Problems Solved

### Problem 1: "Which Mohammed?"
A permit consultant with hundreds of clients has multiple people with the same name in the same industry. The system must disambiguate across people, entities, DBAs, stores, premises, cities, and family relationships — never confusing one client for another.

### Problem 2: Packet Readiness
Every permit filing requires a specific set of documents assembled in a specific way. Missing one document delays the filing by weeks. The system must check readiness against city-specific requirements and flag what's missing before the consultant even looks.

### Problem 3: Document Chaos
Documents arrive via email, download, scan, secure intake relay — and end up in random folders. The system must watch for new documents, classify them, match them to cases, and file them in canonical folders automatically.

### Problem 4: Proximity Exhibits
ABC applications require radius maps showing sensitive uses within 500-600ft. Manual map creation takes 30-60 minutes per location. The system must automate this end-to-end using Google Cloud APIs.

### Problem 5: Sensitive Data Protection
Permit filings require SSNs, bank information, and personal history records. This data requires local handling, privacy classification, and approval gates before any external action.

### Problem 6: Revenue Leakage
Dropped deals, forgotten follow-ups, uncollected deposits, and underpriced services cost the business $200-400K/year. The system must track revenue, flag missing deposits, and surface pricing intelligence.

### Problem 7: Phone Interruption Overload
Every client call interrupts the consultant regardless of urgency. The system must triage calls, route routine questions to specialist agents, and pass only urgent matters to the human.

## 4. Feature Requirements

### 4.1 Client & Case Management
- Unified search across all entities (people, businesses, premises, cases)
- Disambiguation for similar names/addresses
- Rich client profiles with source-grounded facts
- Relationship graph (family, business, geographic links)
- Case pipeline with configurable stages
- Per-case task management with activity log

### 4.2 Document Intelligence
- Automated document watcher (metadata-only by default)
- OCR + fuzzy matching for document classification
- Canonical folder structure enforcement
- Privacy classification (filing-PII, personal-private, business-operational)
- Packet readiness checklist with missing-document flags

### 4.3 Forms Library
- Official blank forms organized by jurisdiction/agency/category
- Form version tracking with checksum verification
- Six-month refresh cycle (Forms Librarian role)
- Packet QA form-currency gate (blocks stale forms)

### 4.4 Proximity Exhibits
- Automated ABC/TTB radius map generation
- Google Geocoding + Address Validation + Static Maps
- OSM/Overpass for sensitive-use detection
- Branded, court-ready PDF output
- Approval-gated before entering packet

### 4.5 Secure Document Intake
- Separate, disposable intake relay accepts encrypted client submissions (SecureDrop model)
- Client-side encryption: files encrypted in browser before upload
- Pearl's workstation remains isolated from public intake surfaces — relay has no AI, no vault, no database
- Quarantine zone: malware scan, type validation, privacy classification, human approval
- Client experience: upload docs, view case status, chat with agent — all through relay
- Three transfer methods: USB sneakernet, data diode, brief authenticated pull
- See [architecture/secure-document-intake.md](../architecture/secure-document-intake.md)

### 4.6 Inbox & Communications
- Email classification and case-linking
- Draft-only email replies (approval-gated)
- Voicemail/after-hours handling
- Voice triage and specialist delegation

### 4.7 Revenue Operations
- Stripe invoice creation (approval-gated)
- Deposit tracking with follow-up sequences
- Payment confirmation via webhooks
- Spend gates for filing fees, API costs (approval-gated)
- Full financial audit trail

### 4.8 Approval System
- 4-tier gate system (silent, confirm, explicit, forbidden)
- Approval queue as primary dashboard screen
- Every send/file/spend/move logged with agent, reason, approver

### 4.9 Multi-Agent Team
- Pearl (Chief of Staff) as permanent orchestrator
- Specialist agents for drafting, comms, CRM, field, Q&A, marketing
- Paperclip control plane for delegation, budget, heartbeat, audit
- Silas hub for deployment, updates, weekly reviews, cross-business intelligence

### 4.10 Voice Escalation
- Inbound call triage by Pearl
- Routine → delegate to specialist
- Important → message + task
- Urgent → immediate pass to human with context summary
- Outbound calls for reminders (approval-gated)

### 4.11 Security & Privacy Controls
- Privacy classification for documents, cases, and outbound actions
- Privacy classification enforcement (testable)
- PII never sent to cloud APIs
- Silas-managed security monitoring and incident response
- Local model routing for sensitive-data handling

### 4.12 Management & Feedback
- Weekly Silas ↔ Pearl business review
- Package delivery for skill/knowledge/software updates
- Escalation queue routing to Silas
- Cross-business intelligence aggregation
- Marketing/sales/research intelligence routing

## 5. Success Metrics

| Metric | Target |
|---|---|
| Document classification accuracy | >90% |
| Packet readiness flags catch rate | >95% of missing documents caught |
| Proximity exhibit generation time | <5 min per location (vs 30-60 min manual) |
| Phone calls triaged without human | >60% of inbound calls |
| Revenue leakage reduction | $200-400K/yr recovered |
| Sensitive data exposure incidents | Zero |
| Filing rejection rate (data errors) | <2% |
| the permit consultant time saved | 250-500 hrs/yr |

## 6. Non-Goals (Phase 1)

- Not a generic CRM
- Not user-editable automation rules (code-defined only)
- Not autonomous filing/submission (Walter is Phase 3 with audit history)
- Not a cloud-only product (local-first is core, not optional)
- Not replacing the human — Pearl prepares, the human decides

## 7. Constraints

- Must run on a Mac mini (low CPU footprint)
- Must keep sensitive-data workflows available on controlled client hardware
- Must handle sensitive data without cloud exposure
- Must comply with CCPA, GLBA, and ABC regulatory requirements
- Must maintain audit trail for all actions

## 8. Dependencies

- Hermes Agent runtime (Nous Research)
- NVIDIA Nemotron 3 Ultra (via Nous Portal or local)
- GPT-5.5 (via OpenAI) for engineering tasks
- ElevenLabs for voice synthesis
- Google Cloud Platform for proximity exhibits
- Stripe for payment operations
- Paperclip for multi-agent orchestration
- Silas (LMS Hub) for deployment and management
