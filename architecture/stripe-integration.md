# Stripe Integration — Earn & Spend Gates

PermitOps OS uses Stripe for both **revenue operations** (earning) and **expense management** (spending), with every financial action gated behind human approval.

## Design Philosophy

The hackathon theme is "agents that can earn, spend, and run real operations at scale." PermitOps OS implements this literally:

- **EARN:** Pearl drafts and sends Stripe invoices for permit consulting services
- **SPEND:** Pearl queues expenses (filing fees, API costs, background checks) through Stripe with approval gates
- **AUDIT:** Every financial action is logged with agent, reason, amount, and approver

## EARN — Revenue Operations

### Invoice Creation Flow

```
┌──────────────────────────────────────────────────────┐
│  1. Case reaches "Engaged" stage                     │
│     └─ Pearl identifies billing trigger              │
│                                                       │
│  2. Pearl drafts invoice                             │
│     ├─ Service type: ABC Transfer Application        │
│     ├─ Amount: $3,500 (from city/license rate card)  │
│     ├─ Line items: application prep, filing, hearing │
│     ├─ Client: linked contact + email                │
│     └─ Due date: net-15 from engagement              │
│                                                       │
│  3. Approval gate                                    │
│     └─ the permit consultant reviews → Approve & Send              │
│                                                       │
│  4. Stripe invoice sent                              │
│     ├─ Email delivered to client                     │
│     ├─ Payment link (Stripe Checkout)                │
│     └─ Status tracked: sent → viewed → paid          │
│                                                       │
│  5. Payment confirmation                             │
│     ├─ Stripe webhook → Pearl notified               │
│     ├─ Case advances to next stage                   │
│     └─ Activity log: "Deposit received: $3,500"     │
└──────────────────────────────────────────────────────┘
```

### Deposit Tracking

Pearl's `pnm-deposit-tracker` skill monitors unsigned service agreements and outstanding deposits:

- Flags cases where agreement was sent but not signed
- Tracks deposits by follow-up stage (initial, 1st reminder, 2nd reminder, overdue)
- Drafts follow-up emails (approval-gated before sending)
- Surfaces stale deposits in morning briefings

### Service Rate Intelligence

Permit consulting pricing varies by:
- License type (ABC transfer, CUP, business license, TTB)
- City/jurisdiction (complexity varies)
- Entity complexity (multi-site, corporate chain, individual)
- Historical pricing for this client

Pearl surfaces pricing intelligence on every quote using the evidence hierarchy:
```
"Last 3 ABC transfers in Anaheim averaged $3,200-$3,800.
Your last quote for this client was $3,500 with 10% discount built in.
Recommend: $3,800 → effective $3,420 after standard 10% discount."
```

## SPEND — Expense Management

### Approval-Gated Spending

```
┌──────────────────────────────────────────────────────┐
│  1. Case requires an expense                         │
│     Example: ABC filing fee $450                     │
│                                                       │
│  2. Pearl queues the expense                         │
│     ├─ Amount: $450.00                               │
│     ├─ Vendor: CA Department of ABC                  │
│     ├─ Reason: "Filing fee for ABC Transfer, case..."│
│     ├─ Budget check: within agent limit              │
│     └─ Linked to: case_id, packet_id                 │
│                                                       │
│  3. Human approval required (Tier 3)                 │
│     └─ the permit consultant reviews full context before approving │
│                                                       │
│  4. Approved → Stripe payment executes               │
│     ├─ Payment confirmation logged                   │
│     ├─ Receipt attached to case packet               │
│     └─ source_evidence: "filing_fee_receipt_..."     │
│                                                       │
│  5. Full audit trail                                 │
│     └─ activity_log: agent, amount, reason, approver │
└──────────────────────────────────────────────────────┘
```

### Expense Categories

| Category | Examples | Typical Approval |
|---|---|---|
| Filing fees | ABC application, city business license, CUP | Tier 3 (explicit) |
| API costs | OCR processing, background checks, geocoding | Tier 2 (batch approval) |
| Courier/mail | Certified mail, overnight delivery | Tier 2 |
| Subcontractor | Process server, notary | Tier 3 |
| Software | SaaS subscriptions, API quotas | Tier 3 |

## Stripe Skills (Hermes Agent)

Pearl uses Hermes Agent's Stripe integration through dedicated skills:

### Earn Skills
- **Invoice creation** — draft invoice from case context
- **Invoice sending** — send via Stripe after approval
- **Payment tracking** — monitor payment status via webhooks
- **Deposit follow-up** — automated reminder sequences (drafts only, approval-gated)

### Spend Skills
- **Expense queuing** — create pending payment with reason + limit
- **Budget checking** — verify agent hasn't exceeded daily limits
- **Payment execution** — process approved Stripe payment
- **Receipt logging** — attach payment confirmation to case

## Audit Trail

Every financial action creates an immutable record:

```json
{
  "action": "stripe_invoice_sent",
  "actor": "pearl",
  "case_id": "...",
  "amount": 3500.00,
  "reason": "ABC Transfer Application — Anaheim case",
  "approver": "human-operator",
  "approved_at": "2026-06-17T14:30:00Z",
  "stripe_invoice_id": "in_...",
  "activity_log_id": "..."
}
```

## Why This Matters for the Hackathon

The hackathon asks for agents that "earn, spend, and run real operations." Most submissions will show a toy. PermitOps OS shows:

1. **Real earning** — actual invoice generation and payment tracking for a real business
2. **Real spending** — approval-gated expenses for real filing fees and API costs
3. **Real operations** — not a demo, but a system running on dedicated hardware for a $1-2M/year consulting business
4. **Real safety** — no money moves without human approval, every dollar is auditable

## Stripe Configuration

```yaml
# Stripe integration (keys redacted)
stripe:
  mode: test  # test mode for development; production requires separate keys
  webhook_secret: ${STRIPE_WEBHOOK_SECRET}
  
  earn:
    invoice_defaults:
      currency: usd
      payment_terms: net_15
    approval_required: true
    
  spend:
    per_agent_daily_limit: 500  # USD
    approval_tier: 3  # explicit approval required
    categories:
      - filing_fees
      - api_costs
      - courier
      - subcontractor
    
  audit:
    log_all_actions: true
    export_format: json
```

## Related

- [Approval Gates](approval-gates.md) — how financial actions connect to the gate system
- [Multi-Model Inference](multi-model-inference.md) — ElevenLabs for payment confirmation calls
- [Architecture Overview](overview.md) — system data flow including Stripe
