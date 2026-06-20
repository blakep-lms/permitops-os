# Architecture

## System Overview

PermitOps OS runs **Pearl**, a Hermes Agent profile, as the AI operator for a permit consulting business.

## Agent Layer (Hermes Agent)

Pearl is a Hermes Agent profile with:

- **Persistent memory** — every client, case, jurisdiction, document, and interaction is remembered across sessions
- **Skills** — modular capabilities for document intake, packet assembly, jurisdictional research, filing preparation, and client communication
- **Approval gates** — no send, file, spend, or private-data move happens without human sign-off
- **Multi-agent coordination** — Pearl delegates to specialist sub-agents for:
  - Document QA / packet readiness checking
  - Jurisdictional research (city codes, filing requirements)
  - Filing preparation and review queues

## Inference Layer (NVIDIA)

- NVIDIA-accelerated inference for document classification, case processing, and agent reasoning
- Designed for NemoClaw/OpenShell runtime compatibility
- Production target: Nemotron 3 Ultra for high-throughput permit processing

## Payment Layer (Stripe)

### Earn (Revenue)
- Pearl drafts service invoices through **Stripe Skills**
- Invoices are sent to clients for payment
- Payment confirmation triggers case advancement
- Full revenue tracking: invoice → payment → case stage

### Spend (Expenses)
- When a case requires a filing fee, OCR processing, or background check API:
  1. Pearl queues the expense through Stripe with a reason and a limit
  2. A human reviews and approves
  3. Only after approval does the expense execute
- Every spend is logged with: amount, reason, approver, case link

## Safety Layer

The approval gate is the core safety mechanism:

| Action | Pearl Can Draft | Pearl Can Execute (with approval) | Pearl Cannot Do |
|--------|----------------|-----------------------------------|-----------------|
| Documents | ✅ | ✅ (after review) | Send without sign-off |
| Filings | ✅ | ✅ (after approval) | File without sign-off |
| Payments | ✅ (queue) | ✅ (after approval) | Spend without sign-off |
| File moves | ✅ (propose) | ✅ (after approval) | Move private data without sign-off |
| Client comms | ✅ (draft) | ✅ (after approval) | Send without sign-off |

## Data Flow

```
┌──────────────┐
│  New Client  │
│   Request    │
└──────┬───────┘
       │
       ▼
┌──────────────────────────────────┐
│         Pearl (Hermes)           │
│  ┌─────────┐  ┌──────────────┐   │
│  │Classify │→ │  Link person │   │
│  │ case    │  │  entity/city │   │
│  └─────────┘  └──────────────┘   │
│       │                          │
│       ▼                          │
│  ┌──────────────────┐            │
│  │ Packet Readiness │            │
│  │  ready/missing/  │            │
│  │    review        │            │
│  └──────────────────┘            │
│       │                          │
│       ▼                          │
│  ┌──────┐    ┌──────────┐        │
│  │ EARN │    │  SPEND   │        │
│  │Stripe│    │  Stripe  │        │
│  │Invoice│   │  +Gate   │        │
│  └──────┘    └──────────┘        │
│       │            │             │
│       ▼            ▼             │
│  ┌─────────────────────┐         │
│  │  Human Approval     │         │
│  │  ┌───┐  ┌────────┐ │         │
│  │  │ ✓ │  │  ✗    │ │         │
│  │  └───┘  └────────┘ │         │
│  └─────────────────────┘         │
│       │                          │
│       ▼                          │
│  ┌──────────────────┐            │
│  │ File / Send /    │            │
│  │ Execute          │            │
│  └──────────────────┘            │
└──────────────────────────────────┘
```

## Multi-Jurisdiction Support

Pearl maintains a knowledge layer of:
- City/county permit requirements
- Document checklists per license type
- Filing deadlines and renewal cycles
- Source/evidence hierarchy for each jurisdiction

This is not hardcoded — it's learned and maintained through the agent's memory and skills.
