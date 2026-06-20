# Evidence Hierarchy — 7-Level Source-of-Truth

Every important fact in PermitOps OS carries verifiable provenance. This is not a feature — it is the core differentiator that separates a real business operating system from a chatbot that hallucinates.

## The Problem

Permit consultants deal with conflicting information constantly:
- A client's email says one entity name; the Secretary of State says another
- A city planner verbally confirmed something over the phone; nobody wrote it down
- An old document references a previous DBA; the new DBA doesn't match
- An AI model infers a relationship that might be true but can't be verified

Without a source hierarchy, the system would silently promote the most recent or most confident claim. That is how filings get rejected, permits get delayed, and trust gets broken.

## The 7-Level Hierarchy

```
┌─────────────────────────────────────────────────────┐
│  LEVEL 1: Official Portal / Agency Requirement       │
│  The law. The current form. The published rule.      │
│  → Always wins.                                       │
├─────────────────────────────────────────────────────┤
│  LEVEL 2: Executed / Source Documents                │
│  Scanned originals, signed contracts, recorded deeds │
│  → Beats everything except Level 1.                  │
├─────────────────────────────────────────────────────┤
│  LEVEL 3: Human Confirmation                         │
│  "the permit consultant confirmed this is correct"                 │
│  → Trusted but documented with who/when/why.         │
├─────────────────────────────────────────────────────┤
│  LEVEL 4: Structured DB Row with Source Path         │
│  Data in the system, but traceable to evidence       │
│  → Only as good as its source_evidence link.         │
├─────────────────────────────────────────────────────┤
│  LEVEL 5: Vault Note / Profile Soft Context          │
│  Narrative knowledge, relationship notes             │
│  → Useful but not authoritative.                     │
├─────────────────────────────────────────────────────┤
│  LEVEL 6: Historical Pattern                         │
│  "In the last 5 similar cases, the city required…"  │
│  → Directional signal, not rule.                     │
├─────────────────────────────────────────────────────┤
│  LEVEL 7: Model Inference                            │
│  AI-generated reasoning, extrapolation               │
│  → NEVER sufficient alone. Always labeled.           │
└─────────────────────────────────────────────────────┘
```

## Evidence Metadata

Every important fact in the system carries:

| Field | Type | Description |
|---|---|---|
| `source_type` | enum | `portal`, `source_file`, `human_confirmation`, `db_row`, `vault_note`, `historical_note`, `model_inference` |
| `source_path_or_url` | string | Where the evidence lives |
| `captured_at` | timestamp | When this evidence was recorded |
| `captured_by` | string | Who or what captured it |
| `confidence` | enum | `approved`, `candidate`, `needs-review`, `rejected` |
| `privacy_classification` | enum | `business-operational`, `filing-required-PII`, `personal-safe`, `personal-private`, `unknown-sensitive` |

## How It Works in Practice

### Scenario: Entity Name Conflict

```
1. Pearl checks a case for packet readiness
2. Finds: email says "Smith Enterprises LLC"
   but Secretary of State says "Smith Enterprise LLC" (no 's')
3. Source hierarchy resolves it:
   - Level 2 (source doc: SoS filing) beats Level 5 (email/vault note)
4. Pearl flags: "Entity name mismatch — using SoS filing as authority"
5. Review flag created: severity=info, confidence=needs-review
6. the permit consultant sees the flag in her dashboard, confirms, or overrides
```

### Scenario: Missing Document

```
1. Packet checklist requires a current ABC license
2. Pearl searches source_evidence for this case
3. Finds: nothing at Level 1-3
4. Finds: a reference at Level 5 (vault note mentions "old ABC from 2023")
5. Pearl flags: "ABC license referenced in notes but no source document found"
6. Review flag: severity=high, action=block-until-resolved
7. Task created: "Obtain current ABC license from client"
```

### Scenario: AI Inference Boundary

```
1. Pearl infers that two clients are related based on shared address
2. Source type = model_inference (Level 7)
3. Confidence = candidate
4. The inference is VISIBLE but NEVER acted upon without:
   → Level 3 human confirmation
   OR
   → Level 2 source document proving the relationship
5. If the permit consultant never confirms, the inference stays labeled as candidate forever
```

## Confidence Lifecycle

```
candidate → needs-review → approved
    │                        ↑
    └────────────────────────┘
     (human review promotes)

candidate → rejected
     (human review or contradicting evidence)
```

**Rule:** Unknowns are tagged, not guessed. `needs-review` items never silently resolve — a human must move them to `approved` or `rejected`.

## Why This Matters for the Hackathon

Most "AI for business" demos show the AI doing things. PermitOps OS shows the AI *not* doing things without evidence — and that discipline is what makes it safe enough to operate a real $1-2M/year consulting business.

The evidence hierarchy is the reason Pearl can:
- Prepare filings that don't get rejected for wrong entity names
- Flag inconsistencies before they cost weeks of delay
- Never send an email or file a form based on a hallucinated fact
- Tell the permit consultant exactly *why* it believes something, with a clickable source

## Related

- [Approval Gates](approval-gates.md) — how the evidence hierarchy connects to human-in-the-loop
- [Data Model](data-model.md) — how `source_evidence` and `privacy_classifications` are stored
