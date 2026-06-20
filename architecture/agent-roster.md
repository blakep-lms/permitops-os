# Multi-Agent Architecture — The Paperclip Control Plane

PermitOps OS deploys a team of specialized AI agents coordinated through the [Paperclip](https://paperclip.ing) control plane. All agents operate under defined authority boundaries with budget limits, audit trails, and heartbeat monitoring.

The system has two orchestration layers: **Silas** (the LMS hub consultant that manages the deployment) and **Pearl** (the business orchestrator that runs daily operations).

## Architecture Overview

```
┌──────────────────────────────────────────────────────────────┐
│  TIER 0: SILAS — LMS Hub Consultant                           │
│                                                                │
│  Deploys and manages multiple business agents:                 │
│  Pearl (PermitOps) · Marvin (FieldOps) · Future agents         │
│                                                                │
│  Capabilities:                                                 │
│  • Package delivery (skills, research, software, security)     │
│  • Weekly business reviews                                     │
│  • Cross-business intelligence aggregation                     │
│  • Air-gap management and security monitoring                  │
│  • Escalation routing from business agents                     │
└───────────────────────────┬──────────────────────────────────┘
                             │
              deploys / updates / reviews / air-gaps
                             │
┌───────────────────────────▼──────────────────────────────────┐
│  TIER 1: PEARL — Business Chief of Staff (Orchestrator)       │
│                                                                │
│  ✅ Active on dedicated hardware                               │
│  Model: GPT-5.5 (primary) + Nemotron 3 Ultra (document)       │
│  Runtime: Hermes Agent on Mac mini                             │
│                                                                │
│  Responsibilities:                                             │
│  • Orchestrate all permit operations                           │
│  • Client intake and classification                            │
│  • Document intelligence and canonical filing                  │
│  • Packet readiness assessment                                 │
│  • Multi-jurisdiction tracking (100+ CA cities)                │
│  • Approval gate enforcement                                   │
│  • Daily morning briefs                                        │
│  • Weekly business self-audits (reported to Silas)             │
│  • Voice/phone triage and call routing                         │
│  Skills: 27 PNM-specific Hermes skills                         │
│  Authority: Can delegate to all sub-agents; cannot             │
│  send/file/spend without human approval                        │
└──┬──────┬──────┬──────┬──────┬──────┬──────┬──────┬───────────┘
   │      │      │      │      │      │      │      │
   ▼      ▼      ▼      ▼      ▼      ▼      ▼      ▼
 Tier 2A              Tier 2B              Tier 3
```

## Tier 0: Silas — LMS Hub Consultant

**Silas** is not a permit specialist — Silas is the AI consultant from Linear Marketing Solutions that deploys and manages AI business operators across multiple client businesses.

| Attribute | Value |
|---|---|
| **Role** | Central hub — deploys, updates, reviews, and secures business agents |
| **Manages** | Pearl (PermitOps), Marvin (FieldOps), future business agents |
| **Weekly cycle** | Reads each agent's business review → delivers improvements |
| **Security** | Can air-gap any business, monitor logs, deliver patches |
| **Intelligence** | Aggregates learnings across all deployments |

See [Silas Hub Consultant](silas-hub-consultant.md) for full details.

## Tier 1: Pearl — Chief of Staff / Orchestrator

| Attribute | Value |
|---|---|
| **Status** | ✅ Active on dedicated hardware |
| **Model** | GPT-5.5 (primary) + Nemotron 3 Ultra (document reasoning) |
| **Runtime** | Hermes Agent on Mac mini |
| **Skills** | 27 PNM-specific Hermes Agent skills |
| **Authority** | Can delegate to all sub-agents; cannot send/file/spend without human approval |

### Pearl's Voice Role
Pearl also serves as the **voice gatekeeper** for inbound calls:
- Answers business line with ElevenLabs voice
- Triages caller intent using case context
- Routes routine calls to specialist agents
- Passes urgent calls to the permit consultant with context summary
- Takes messages and creates tasks for important but non-urgent matters

See [Voice Escalation](voice-escalation.md) for the full call routing design.

## Tier 2: Specialist Agents

### Tier 2A — Core Specialists

#### Ruth — Drafting & Forms
| Attribute | Value |
|---|---|
| **Status** | 🔜 Phase 2A |
| **Model** | Nemotron 3 Ultra (long-context document reasoning) |
| **Responsibilities** | Draft findings reports, fill PDF permit forms, assemble exhibit packets, cross-reference city requirements |
| **Voice role** | Can handle delegated calls requesting document copies or form explanations |

#### Marcus — Communications
| Attribute | Value |
|---|---|
| **Status** | 🔜 Phase 2A |
| **Model** | GPT-5.5 |
| **Responsibilities** | Email auto-reply and triage, referral SMS campaigns, deposit follow-up sequences, client status updates |
| **Voice role** | Can handle delegated calls for appointment scheduling or routine status questions |

#### Henry — CRM & Broker Relations
| Attribute | Value |
|---|---|
| **Status** | 🔜 Phase 2A |
| **Model** | Nemotron 3 Ultra |
| **Responsibilities** | Broker/field-rep relationship management, hearing date reminders, GM# (store ID) geocoding, account expansion identification |

### Tier 2B — Extended Team

#### Iris — Field Agent
| Attribute | Value |
|---|---|
| **Model** | GPT-5.5 (laptop-deployed) |
| **Role** | On-site document capture, premises verification, field photography |

#### Vivian — Customer Q&A
| Attribute | Value |
|---|---|
| **Model** | Nemotron 3 Ultra |
| **Role** | Client-facing status updates, FAQ, portal navigation help |
| **Voice role** | Primary agent for delegated routine calls ("What's my case status?") |

#### Diana — Marketing & Reactivation
| Attribute | Value |
|---|---|
| **Model** | GPT-5.5 |
| **Role** | Reactivation campaigns, city-specific outreach, social media |

## Tier 3: Automation Layer

#### Walter — Submissions
| Attribute | Value |
|---|---|
| **Model** | Nemotron 3 Ultra |
| **Role** | Automated city-portal submissions (where APIs exist), filing status tracking |
| **Phase** | Phase 3 only — requires audited success history and Packet QA pass |

#### Frank — Engineering
| Attribute | Value |
|---|---|
| **Model** | GPT-5.5 |
| **Role** | Custom tool builds, API integrations, infrastructure automation |

## Paperclip Control Plane

[Paperclip](https://paperclip.ing) provides the multi-agent coordination layer:

```
         Silas (Hub)
              │
         Pearl (CEO)
        /    |    |    |    |    |    |    \
      Ruth Marcus Henry Iris Vivian Diana Walter Frank
```

Each agent has:
- **Budget limit** — max tokens/API calls per day
- **Audit trail** — every action logged with timestamp, reason, and outcome
- **Heartbeat** — health check every interval; silent agents trigger alerts
- **Escalation queue** — uncertain decisions route to Pearl → Silas → human
- **Authority scope** — explicitly defined what each agent can and cannot do

See [Paperclip Orchestration](paperclip-orchestration.md) for the full control plane design.

## Voice/Phone Routing Through the Roster

When a client calls, Pearl triages and may delegate to a specialist:

```
Client calls → Pearl answers
  ↓
Pearl triages:
  "Status update?"     → Vivian handles
  "Schedule a time?"   → Marcus handles
  "Need a document?"   → Ruth handles
  "Urgent issue!"      → Pearl passes to the permit consultant
```

See [Voice Escalation](voice-escalation.md) for the full triage system.

## Safety Boundaries

1. **No external sends** without human approval (emails, forms, payments, signatures)
2. **No source file mutation** without explicit approval (no move/delete/rename/ingest)
3. **Uncertainty preservation** — unknowns are tagged, not guessed:
   - `candidate`
   - `pending-review`
   - `needs-confirmation`
   - `needs-source-review`
4. **Privacy protection** — `_personal/` and sensitive docs marked `private-do-not-ingest`
5. **Budget enforcement** — agents that exceed limits are paused, not silently continued
6. **Escalation chain** — uncertain decisions route up: specialist → Pearl → Silas → the LMS owner/human
7. **Air-gap capability** — Silas can disconnect any business from the internet at any time

## Related

- [Paperclip Orchestration](paperclip-orchestration.md)
- [Silas Hub Consultant](silas-hub-consultant.md)
- [Voice Escalation](voice-escalation.md)
- [Weekly Feedback Loop](weekly-feedback-loop.md)
- [Air-Gapped Security](air-gapped-security.md)
