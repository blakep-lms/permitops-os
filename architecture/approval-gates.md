# Approval Gates — Human-in-the-Loop Operations

No agent in PermitOps OS can send emails, file documents, move files, or spend money without human sign-off. This is not a limitation — it is the core safety architecture that makes autonomous business operation trustworthy.

## Design Philosophy

The approval gate system follows one principle: **Pearl prepares, the human decides.**

Pearl (the AI operator) can draft, organize, analyze, audit, suggest, and queue. But every action with real-world consequences — sending, filing, spending, deleting, moving, sharing — requires explicit human approval through a defined, auditable gate.

This means Pearl can work autonomously for hours: classifying documents, checking packet readiness, drafting email replies, preparing forms, generating invoices — and then present a queue of decisions for the human operator.

## Action Classification

### Tier 1: Silent Allowed (No Approval Needed)

Pearl can do these autonomously:

| Action | Example |
|---|---|
| Read and classify documents | "This PDF is a business license renewal form" |
| Search and link records | "This document belongs to the Anaheim case" |
| Draft content | Email replies, LOIs, form fills, briefings |
| Generate metadata | Tags, summaries, relationship suggestions |
| Queue items for review | "This case has a missing document" |
| Update internal state | Task status, pipeline stage suggestions |
| Run analysis | Inconsistency audits, deadline checks |

### Tier 2: Visible Confirmation (One-Click Approve)

Pearl prepares and the human confirms with a single action:

| Action | What Pearl Does | What Human Does |
|---|---|---|
| Send drafted email | Drafts complete reply with attachments | Clicks "Approve & Send" |
| Promote document to Final Deliverables | Validates, checks version currency | Clicks "Promote" |
| Create Stripe invoice | Drafts invoice with correct items | Clicks "Send Invoice" |
| Update case status | Suggests next pipeline stage | Clicks "Confirm" |

### Tier 3: Explicit Approval (Detailed Review Required)

These require the human to review full context before approving:

| Action | What Pearl Does | What Human Reviews |
|---|---|---|
| File a permit application | Prepares complete packet + submission checklist | Full packet review + submission method |
| Move source files | Proposes canonical folder structure | File-by-file review or batch approval |
| Spend money (filing fees, background checks) | Queues expense with reason + limit | Amount, vendor, justification |
| Delete or archive records | Flags for cleanup | What gets deleted, why, and irreversibility warning |
| OCR / broad document ingestion | Proposes scope of files to process | Which files, what extraction method |

### Tier 4: Launch-Forbidden (Cannot Do At All)

Pearl will never do these, period:

| Action | Why |
|---|---|
| Submit filings autonomously | Filing is an actor-of-record action; Walter (Phase 3) may handle this only after audited success history |
| Sign legal documents | Requires human signature |
| Access personal/private files | Quarantined by privacy classification |
| Modify source archive without approval | Source files are evidence — must not be mutated |
| Exceed budget limits | Agents that exceed limits are paused |
| Execute generated code | Code must be reviewed and tested first |

## Gate Implementation

### Approval Object Schema

```json
{
  "id": "uuid",
  "entity_type": "case|packet|email|invoice|file_move",
  "entity_id": "reference",
  "action_type": "send|file|spend|move|delete|promote|ingest",
  "requested_by": "pearl|marcus|ruth|...",
  "reason": "Client requested ABC transfer application filing deadline 2026-07-15",
  "context": {
    "case_id": "...",
    "amount": 450.00,
    "vendor": "CA Department of Alcoholic Beverage Control",
    "documents": ["abc-221.pdf", "articles-of-inc.pdf", "..."]
  },
  "status": "pending|approved|rejected|expired",
  "approver_id": "user_id",
  "approved_at": "timestamp",
  "audit_notes": "Any notes the approver adds"
}
```

### Approval Queue (the permit consultant Today View)

The approval queue is the most important screen in the system. It shows:

```
┌──────────────────────────────────────────────────────┐
│  🔔 Needs Your Approval (3)                           │
├──────────────────────────────────────────────────────┤
│                                                       │
│  📧 Email Reply — Mohammed Khoury (Anaheim)          │
│  RE: ABC Transfer Application Status                 │
│  Drafted by Pearl · 2 min ago                        │
│  [Review] [Approve & Send] [Reject]                  │
│                                                       │
│  💳 Filing Fee — $450.00                             │
│  ABC Transfer Application — Costa Mesa case          │
│  Requested by Pearl · 15 min ago                     │
│  [Review Details] [Approve] [Reject]                 │
│                                                       │
│  📁 Packet Promotion — 7-Eleven #5678               │
│  4 documents ready for Final Deliverables            │
│  Prepared by Pearl · 1 hour ago                      │
│  [Review All 4] [Approve Batch] [Reject]             │
│                                                       │
└──────────────────────────────────────────────────────┘
```

## Audit Trail

Every approval (and rejection) is logged in `activity_log` with:

- **Who** requested it (agent identity)
- **Why** it's needed (reason)
- **What** the action entails (full context)
- **Who** approved/rejected it
- **When** it was decided
- **Outcome** of the action

This trail is immutable and exportable for compliance review.

## Budget Enforcement

Each agent has daily limits:

| Agent | Token Budget | API Call Limit | Spend Authority |
|---|---|---|---|
| Pearl | Configurable | Configurable | Can queue, cannot execute |
| Ruth | Lower (drafting focus) | Limited | None |
| Marcus | Lower (comms focus) | Email API capped | None |
| Walter (Phase 3) | Bounded | Filing API scoped | Per-case limit with approval |

**Rule:** agents that exceed limits are **paused**, not silently continued. A paused agent triggers an alert and requires human intervention to resume.

## Connection to Evidence Hierarchy

Approval gates and evidence hierarchy work together:

1. Pearl proposes an action with evidence at a certain source level
2. If the evidence is below Level 3 (human confirmation), the action is automatically Tier 3
3. If the evidence is Level 7 (model inference), the action cannot be queued at all — it must first be promoted through review
4. High-severity review flags can **block** packet promotion until resolved

## Stripe Integration

See [Stripe Integration](stripe-integration.md) for how the approval gate connects to payment operations.

## Related

- [Evidence Hierarchy](evidence-hierarchy.md)
- [Stripe Integration](stripe-integration.md)
- [Multi-Agent Architecture](agent-roster.md) — agent budget and authority limits
