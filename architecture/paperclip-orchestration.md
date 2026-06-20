# Paperclip — Multi-Agent Orchestration Control Plane

[Paperclip](https://paperclip.ing) is the multi-agent coordination layer that enables PermitOps OS to deploy a team of specialized AI agents under Pearl's authority, with budget limits, heartbeat monitoring, and escalation queues.

## What Paperclip Does

Paperclip sits above individual Hermes Agent instances and provides:

1. **Agent lifecycle management** — deploy, pause, resume, and decommission specialist agents
2. **Delegation routing** — Pearl assigns tasks to the right specialist; results flow back
3. **Budget enforcement** — per-agent token/API/spend limits with auto-pause on exceed
4. **Heartbeat monitoring** — health checks; silent or failed agents trigger alerts
5. **Escalation queues** — uncertain decisions route up to the orchestrator → human
6. **Audit trail** — every inter-agent communication is logged

## Architecture

```
┌──────────────────────────────────────────────────────────┐
│                    PAPERCLIP CONTROL PLANE                 │
│                                                            │
│  ┌─────────────────────────────────────────────────────┐  │
│  │  Silas (LMS Hub)                                    │  │
│  │  The AI consultant that deploys and manages Pearl    │  │
│  │  across multiple business instances.                 │  │
│  │  See: silas-hub-consultant.md                       │  │
│  └──────────────────────┬──────────────────────────────┘  │
│                          │                                  │
│           deploys / updates / reviews / air-gaps            │
│                          │                                  │
│  ┌───────────────────────▼──────────────────────────────┐  │
│  │  Pearl (Business Orchestrator)                       │  │
│  │  Chief of Staff for a single business instance.      │  │
│  │  Delegates to specialist agents.                     │  │
│  └──┬──────┬──────┬──────┬──────┬──────┬──────┬───────┘  │
│     │      │      │      │      │      │      │           │
│     ▼      ▼      ▼      ▼      ▼      ▼      ▼           │
│   Ruth  Marcus  Henry  Iris  Vivian Diana Walter  Frank   │
│                                                            │
│   Each agent:                                              │
│   • Has scoped authority (can/cannot do specific things)   │
│   • Has budget limits (tokens, API calls, spend)           │
│   • Has heartbeat monitoring                               │
│   • Routes uncertain decisions up the chain                │
│   • Logs every action to audit trail                       │
└──────────────────────────────────────────────────────────┘
```

## Delegation Flow

### Normal Delegation

```
1. Event arrives at Pearl (new document, email, intake, deadline)
2. Pearl classifies the task type
3. Pearl delegates to the appropriate specialist:
   - Document drafting → Ruth
   - Email reply → Marcus
   - CRM/broker update → Henry
   - Field capture → Iris
   - Customer question → Vivian
   - Marketing campaign → Diana
   - Portal submission → Walter (Phase 3, with audit history)
   - Tool/code build → Frank
4. Specialist completes task (or queues for approval)
5. Result flows back to Pearl
6. Pearl logs outcome and updates case state
```

### Escalation Chain

```
Specialist can't decide (uncertainty, missing data, policy violation risk)
  ↓
Routes to Pearl with reason + context
  ↓
Pearl decides:
  (a) Handle it herself
  (b) Re-delegate to different specialist
  (c) Escalate to the permit consultant (human approval needed)
  (d) Park in review queue (not urgent enough to interrupt)
```

### Voice/Phone Escalation (see voice-escalation.md)

```
Client calls business line
  ↓
Pearl answers (voice AI)
  ↓
Pearl triages:
  (a) Routine → delegate to specialist (e.g., Vivian for status)
  (b) Important but not urgent → take message, create task
  (c) Urgent → pass call to the permit consultant with context summary
```

## Inter-Agent Communication

Paperclip agents communicate through structured messages:

```json
{
  "from": "pearl",
  "to": "ruth",
  "task_type": "draft_finding",
  "case_id": "...",
  "context": {
    "premises_address": "...",
    "license_type": "ABC Transfer",
    "city": "Anaheim"
  },
  "authority_scope": "draft_only",
  "budget_limit": {"tokens": 5000},
  "requires_approval_before_action": true,
  "deadline": "2026-06-20T17:00:00Z"
}
```

Responses carry:
```json
{
  "from": "ruth",
  "to": "pearl",
  "status": "completed|blocked|escalated",
  "result": {...},
  "evidence_refs": ["source_evidence:uuid", ...],
  "confidence": "candidate",
  "approval_needed": "human_review"
}
```

## Budget Enforcement

| Agent | Daily Token Budget | API Call Limit | Spend Authority | Notes |
|---|---|---|---|---|
| Pearl | High (orchestrator) | High | Can queue, cannot execute | Manages everyone else's limits |
| Ruth | Medium (drafting) | Limited | None | Nemotron 3 Ultra for long docs |
| Marcus | Medium (comms) | Email API capped | None | Cannot send without approval |
| Henry | Low (CRM) | Geocoding API | None | Read/write CRM only |
| Vivian | Low (Q&A) | None | None | Read-only responses |
| Walter | Bounded | Filing API scoped | Per-case with approval | Phase 3 only |
| Frank | Variable | Dev tools | None | Code review required |

**Rule:** When an agent exceeds its budget, it is **paused** and Pearl is alerted. The agent does not silently continue burning resources. Human intervention is required to increase the budget or resume the agent.

## Heartbeat Monitoring

Every agent sends a heartbeat at a configured interval. If an agent misses heartbeats:
1. Pearl checks if the agent process is alive
2. If dead, Pearl attempts restart
3. If restart fails, Pearl alerts Silas/the permit consultant
4. Tasks assigned to the dead agent are re-queued or re-delegated

## Multi-Business Deployment (Silas Hub)

Paperclip's most powerful feature is **multi-business orchestration from a central hub**:

```
┌──────────────────────────────────────────────────────────┐
│                     SILAS (LMS HUB)                       │
│                                                            │
│  AI Consultant for Linear Marketing Solutions              │
│  Deploys and manages multiple business agents:             │
│                                                            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐    │
│  │  Pearl / PNM │  │ Marvin / IPS │  │  Future / X  │    │
│  │  PermitOps   │  │  FieldOps    │  │  ServiceOps  │    │
│  │              │  │              │  │              │    │
│  │  9 agents    │  │  TBD agents  │  │  TBD agents  │    │
│  └──────────────┘  └──────────────┘  └──────────────┘    │
│                                                            │
│  Silas can:                                                │
│  • Deploy new agent packages to any business               │
│  • Push software/skill updates                             │
│  • Review weekly logs and session data                     │
│  • Air-gap any business from the internet                  │
│  • Conduct weekly business reviews with each agent         │
│  • Route research/marketing/sales intelligence updates     │
└──────────────────────────────────────────────────────────┘
```

See [Silas Hub Consultant](silas-hub-consultant.md) for the full hub architecture.

## Phase Rollout

| Phase | Agents Active | Paperclip Features |
|---|---|---|
| **Phase 1 (Current)** | Pearl only | Single-agent mode, skills, memory, approval gates |
| **Phase 2A** | + Ruth, Marcus, Henry | Multi-agent delegation, budget enforcement, heartbeats |
| **Phase 2B** | + Iris, Vivian, Diana | Extended team, voice escalation integration |
| **Phase 3** | + Walter, Frank | Full automation with submission agent, engineering agent |

Paperclip is designed so that Phase 1 works perfectly as a single-agent system. The control plane activates as specialist agents come online. Pearl never loses authority — specialists always report up.

## Related

- [Silas Hub Consultant](silas-hub-consultant.md) — the central hub that manages multiple Paperclip deployments
- [Agent Roster](agent-roster.md) — the 9-agent team
- [Voice Escalation](voice-escalation.md) — phone triage through the Paperclip chain
- [Weekly Feedback Loop](weekly-feedback-loop.md) — Silas ↔ Pearl review cycle
- [Air-Gapped Security](air-gapped-security.md) — isolation modes for sensitive deployments
