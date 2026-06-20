# Permit Workflows — PermitOps OS

PermitOps OS manages **7 documented permit workflows**, each with specific requirements, document checklists, city-specific variations, and approval gates. These are real workflows used in production for a California permit consulting business.

All workflows are sanitized — no client names, addresses, case numbers, or personal data.

---

## 1. ABC License Transfer Workflow

**Purpose:** Transfer an existing California alcohol license from one location/entity to another.

### Key Steps
1. Intake — identify transfer type (person-to-person, location-to-location, both)
2. Entity verification — confirm legal entity, ownership, good standing
3. Premises verification — address validation, zoning check, lease/ownership proof
4. Proximity exhibit — generate ABC radius map (see [Google Cloud Integration](../architecture/google-cloud-integration.md))
5. Application preparation — assemble packet: ABC 221 form, entity docs, premise docs, Fingerprint Card Services
6. Background investigation — schedule and coordinate with ABC investigator
7. Public notice — posting period (30 days), handle protests
8. Hearing — prepare for ABC hearing if required
9. Approval — license issued, condition compliance
10. Final deliverables — complete packet with issued license

### Typical Documents
- ABC Form 221 (Application for Transfer)
- Articles of Incorporation / LLC Operating Agreement
- Seller's Permit
- Lease Agreement or Property Deed
- Fingerprint Cards (for each principal)
- Personal History Records (ABC 407)
- Proximity/Radius Map exhibit
- Financial Responsibility documents

### Approval Gates
- Application submission → Tier 3 (explicit approval)
- Filing fee payment → Tier 3
- Public notice posting → Tier 2

---

## 2. Conditional Use Permit (CUP) Workflow

**Purpose:** Obtain a Conditional Use Permit from a city for a use that requires discretionary approval (e.g., alcohol sales in a commercial zone).

### Key Steps
1. Intake — identify the city, zone, and requested use
2. Zoning verification — confirm current zoning, check if CUP is required
3. Pre-application meeting — coordinate with city planning staff
4. Application preparation — assemble CUP packet per city requirements
5. Radius map + resident notification list
6. Environmental review — CEQA exemption or initial study
7. Planning Commission hearing — prepare testimony, exhibits
8. Conditions of approval — negotiate and finalize conditions
9. Final CUP issuance

### City-Specific Variations
Each of the 56+ California cities in Pearl's knowledge base has different:
- CUP application forms
- Radius requirements for resident notification
- Hearing schedules and procedures
- Required findings for approval
- Environmental review thresholds

### Typical Documents
- CUP Application (city-specific form)
- Site plan / floor plan
- Operational statement
- Radius map with notification list
- Environmental determination
- Findings statement

---

## 3. Business License Workflow

**Purpose:** Obtain or renew a city business license for operating a business.

### Key Steps
1. Intake — identify city, business type, entity structure
2. Zoning clearance — verify business use is permitted at the location
3. Application preparation — city-specific business license form
4. Supporting documents — entity docs, fictitious business name statement, seller's permit
5. Fee payment (approval-gated)
6. Renewal tracking — annual/biennial renewal calendar

### Pearl's Role
- Tracks renewal dates across all cities
- Flags upcoming renewals 60/30/15 days before expiration
- Drafts renewal applications with updated information
- Queues renewal fees for approval

---

## 4. Letter of Intent (LOI) Drafting Workflow

**Purpose:** Draft city-specific Letters of Intent for permit applications, planning submissions, or business license requests.

### Key Steps
1. Case context loaded — entity, premises, requested use, city
2. Jurisdiction playbook retrieved — city-specific format, required content, addressee
3. LOI drafted from boilerplate + case data (Nemotron 3 Ultra for regulatory reasoning)
4. Approval gate — the permit consultant reviews and edits before sending
5. Send tracking — logged in case communications

### Why This Is Automated
LOIs are repetitive but high-stakes — a wrong statement about the business or premises can delay a filing by weeks. Pearl's `pnm-loi-draft` skill generates city-specific LOIs from the case data, ensuring consistency and accuracy while preserving the human approval gate.

---

## 5. Planning Referral Workflow

**Purpose:** Manage the planning department referral process for projects that require inter-departmental review.

### Key Steps
1. Submit application to planning department
2. Planning refers to: building, fire, police, public works, engineering
3. Track each department's response and conditions
4. Consolidate conditions into project requirements
5. Address conditions before final approval

### Pearl's Role
- Tracks which departments have responded
- Flags departments that haven't responded within expected timeframes
- Drafts follow-up emails (approval-gated)
- Consolidates conditions into a single checklist

---

## 6. ABC License Outreach Lead Funnel Workflow

**Purpose:** Manage the business development pipeline for new ABC license clients.

### Key Stages
1. **Lead** — identified opportunity (expiring license, new business, referral)
2. **Contacted** — initial outreach made
3. **Interested** — prospect engaged, needs assessment
4. **Proposal** — service agreement drafted and sent
5. **Deposit** — awaiting signed agreement + deposit
6. **Engaged** — agreement signed, deposit received → convert to case
7. **Not a Fit** — disqualified (budget, timeline, scope mismatch)

### Pearl's Role
- `pnm-deposit-tracker` monitors unsigned agreements and outstanding deposits
- Drafts follow-up sequences (approval-gated)
- Identifies reactivation opportunities from backburner clients
- Morning briefing surfaces leads needing attention

---

## 7. ABC-TTB Proximity Exhibit Workflow

**Purpose:** Generate court-ready "Points of Consideration" maps for ABC and TTB applications.

### Key Steps
1. Address input from case premises
2. Google Geocoding API → precise coordinates
3. Google Address Validation API → verify real address
4. OSM/Overpass query → find sensitive uses within 500-600ft
5. U.S. Census Geocoder → residential corroboration
6. Google Static Maps API → branded map with markers + radius
7. HTML → PDF → PNG conversion
8. Approval gate → the permit consultant reviews before exhibit enters packet

### See Also
- [Google Cloud Integration](../architecture/google-cloud-integration.md) — full technical pipeline
- This is the most technically innovative workflow — fully automated from address input to filing-ready exhibit

---

## Workflow Design Principles

1. **Source-grounded** — every step cites evidence from the 7-level hierarchy
2. **City-aware** — each workflow adapts to jurisdiction-specific requirements
3. **Approval-gated** — filing, sending, spending all require human sign-off
4. **Audit-trailed** — every action in every workflow is logged
5. **the permit consultant-safe** — workflows surface what needs attention, not raw process logs

## Pipeline Stage Mapping

Workflows map to the case pipeline stages:

```
Intake → Entity/Premises Verification → Zoning/CUP Checks →
Business License → ABC Application → Public Notice →
Agency Follow-up → Approval/License Issued → Final Deliverable → Closed
```

Multiple workflows may be active on a single case (e.g., an ABC Transfer may also require a CUP and a Business License).

## Related

- [Skills](../skills/README.md) — the Hermes Agent skills that execute these workflows
- [Data Model](../architecture/data-model.md) — how cases, packets, and pipeline stages are stored
- [Evidence Hierarchy](../architecture/evidence-hierarchy.md) — how each workflow step cites its sources
- [Approval Gates](../architecture/approval-gates.md) — how risky actions within workflows are gated
