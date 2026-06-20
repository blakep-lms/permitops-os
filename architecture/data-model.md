# Data Model — PNM Business Operating System

This is a sanitized version of the data model from the Silas-approved TRD. All client-specific data, real names, addresses, and PII have been removed.

## Design Principles

1. **Local-first** — canonical data and files stay on-prem unless explicitly approved
2. **Source hierarchy beats model confidence** — every fact carries provenance metadata
3. **Approval gates everywhere** — risky actions queue for human sign-off
4. **Audit everything** — every state change, file access, and AI run has a receipt
5. **Privacy classifications are first-class** — personal/filing-PII/business/unknown-sensitive

## Entity-Relationship Overview

```
┌─────────────────────────────────────────────────────────┐
│                    TENANT / WORKSPACE                     │
│  tenants ──< memberships >── users                        │
└─────────────────────────────────────────────────────────┘
        │
        ▼
┌─────────────────────────────────────────────────────────┐
│                  RELATIONSHIP GRAPH                       │
│                                                           │
│  contacts ──< relationships >── entities                  │
│      │                           │                        │
│      │                     business_locations             │
│      │                      (DBAs / stores)               │
│      │                           │                        │
│      └─────────────── premises ───┘                       │
│                          │                                │
│                       cities                               │
│                                                           │
│  (disambiguation across people, entities, DBAs, stores,   │
│   premises, cities — multiple similar names, one truth)   │
└─────────────────────────────────────────────────────────┘
        │
        ▼
┌─────────────────────────────────────────────────────────┐
│               CASES & PIPELINES                           │
│                                                           │
│  cases ──< tasks                                          │
│    │                                                      │
│    ├── pipeline_stage (with stage transition log)         │
│    ├── packets ──< packet_items                           │
│    ├── approvals                                          │
│    ├── review_flags                                       │
│    └── intakes (intake pipeline → case conversion)        │
└─────────────────────────────────────────────────────────┘
        │
        ▼
┌─────────────────────────────────────────────────────────┐
│              FORMS LIBRARY & EVIDENCE                     │
│                                                           │
│  forms ──< form_versions ──< form_refresh_log             │
│  source_evidence (provenance for every fact)              │
│  privacy_classifications (exclusion rules)                │
└─────────────────────────────────────────────────────────┘
        │
        ▼
┌─────────────────────────────────────────────────────────┐
│            INTAKE / INBOX / AI AUDIT                      │
│                                                           │
│  inbox_items ──< email_drafts                             │
│  activity_log (every action, every actor)                 │
│  ai_runs (every model call, hashed, scoped)               │
└─────────────────────────────────────────────────────────┘
```

## Core Identity & Workspace

```sql
users           → id, email, name, role, password_hash, mfa_enabled,
                    created_at, updated_at, last_login

tenants         → id, name, slug, settings

memberships     → user_id, tenant_id, role
```

Single-tenant now; `tenant_id` on every table from day one for future multi-agency expansion.

## Relationship Graph (PNM Differentiator)

```sql
contacts            → id, tenant_id, name, email, phone, type, notes,
                       source_status, privacy, created_at, updated_at

entities            → id, tenant_id, legal_name, entity_type, tax_id_ref,
                       notes, source_status, privacy

premises            → id, tenant_id, address, city_id, county, apn,
                       notes, source_status, privacy

business_locations  → id, tenant_id, dba, store_number, brand,
                       premises_id, entity_id, notes

cities              → id, name, county, state, portal_url, status,
                       notes, last_verified_at

relationships       → id, source_type, source_id, target_type, target_id,
                       relation_type, confidence, source_evidence_id
```

**Why this matters:** A permit consultant may have 4 clients named "Mohammed" in different cities, some related by family or business ownership. The relationship graph disambiguates them and links person ↔ entity ↔ DBA/store ↔ premises ↔ case ↔ city without guessing.

## Cases, Pipelines, Packets & Tasks

```sql
cases               → id, tenant_id, canonical_stem, display_label,
                       contact_id, entity_id, premises_id, city_id,
                       workflow_id, status, priority, source_status,
                       pipeline_stage, license_type, privacy,
                       created_at, updated_at

pipelines           → id, tenant_id, name, kind, stage_config

case_pipeline_events → id, case_id, from_stage, to_stage, updated_by,
                        updated_at, reason

tasks               → id, tenant_id, case_id, title, description, status,
                       due_date, assignee_id, tags, linked_step,
                       order_index, created_by, created_at, updated_at

packets             → id, tenant_id, case_id, packet_type, status,
                       target_agency, due_date, created_at, updated_at

packet_items        → id, packet_id, form_id, source_evidence_id,
                       status, required, notes, order_index

approvals           → id, tenant_id, entity_type, entity_id, action_type,
                       requested_by, approver_id, status, approved_at,
                       notes

review_flags        → id, tenant_id, entity_type, entity_id, flag_type,
                       severity, description, source_evidence_id,
                       status, created_at, resolved_at
```

### Default Case Pipeline Stages

1. Intake
2. Entity / ownership / premises verification
3. Zoning / CUP / local agency checks
4. City business license
5. ABC application
6. Public notice / posting
7. Agency follow-up
8. Approval / license issued
9. Final deliverable
10. Closed / Not a fit

### Intake Pipeline Stages

1. New
2. Qualifying
3. Proposal Sent
4. Awaiting Signature/Deposit
5. Engaged
6. Not a Fit

## Forms Library & Evidence

```sql
forms                   → id, tenant_id, city_id, name, category,
                           file_path, url, status, notes,
                           last_verified_at

form_versions           → id, form_id, version, file_path, url, checksum,
                           effective_date, created_at

form_refresh_log        → id, form_id, checked_at, checked_by, result,
                           old_checksum, new_checksum, notes

source_evidence         → id, tenant_id, entity_type, entity_id,
                           source_type, source_path_or_url, captured_at,
                           captured_by, confidence, privacy_classification,
                           checksum, excerpt_ref

privacy_classifications → id, path_pattern, content_pattern, class,
                           action, notes
```

**Forms currency gate:** packet generation warns or blocks if a required form is stale beyond policy. A Forms Librarian role checks sources every six months with immediate refresh for agency notices.

## Intake, Inbox, Email Drafts & AI Audit

```sql
intakes         → id, tenant_id, contact_id, entity_name, city_id,
                    license_type, source, stage, estimated_value,
                    notes, created_at, updated_at

inbox_items     → id, tenant_id, source, source_ref, linked_case_id,
                    summary, status, priority, requires_approval,
                    created_at, updated_at

email_drafts    → id, tenant_id, case_id, inbox_item_id, subject,
                    body_path_or_body_ref, status, created_by,
                    reviewed_by, created_at, updated_at

activity_log    → id, tenant_id, entity_type, entity_id, action,
                    actor_id, ip_address, user_agent, timestamp,
                    metadata

ai_runs         → id, tenant_id, prompt_id, model, input_hash,
                    output_hash, source_scope, tool_calls,
                    actor_id, timestamp, review_status
```

**Rule:** Sensitive columns (SSN, TIN, personal docs, filing credentials) are never stored directly. The system stores redacted references and source paths only.

## Privacy Classification System

| Class | Examples | Default Action |
|---|---|---|
| `business-operational` | Case files, entity docs, business licenses | Processable with standard gates |
| `filing-required-PII` | SSN, tax ID, financial statements | Processable but never exposed in UI/logs/prompts |
| `personal-safe-context` | Family relationships, cultural notes | Optional context marker; never stored in structured tables |
| `personal-private` | Birth certificates, marriage licenses | Quarantined; never ingested |
| `unknown-sensitive` | Unidentified documents | Held for review before any processing |

## Runtime Decision: SQLite vs PostgreSQL

- **SQLite:** acceptable for dev, prototypes, UI state, test fixtures
- **PostgreSQL:** preferred for production runtime holding real operational records
- Phase 0 requires a stack decision record documenting the chosen path, file permissions, backup/restore process, and migration path

## Sanitization Note

This data model is derived from a real California permit consulting business. All client names, addresses, tax IDs, case numbers, and personal information have been removed. The schema represents the actual production design, not a theoretical exercise.
