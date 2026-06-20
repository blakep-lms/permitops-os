# Voice Escalation — Phone Triage & Agent Routing

PermitOps OS envisions a voice/phone layer where client calls to the business line (or the permit consultant's personal number) are answered, triaged, and routed by Pearl — protecting the permit consultant's time while ensuring urgent matters reach her immediately.

## The Problem

the permit consultant manages hundreds of active clients. Her phone rings constantly:
- "What's the status of my ABC transfer?" (routine — can be answered without the permit consultant)
- "The city planner needs a call back today." (important — the permit consultant needs to act)
- "My ABC license hearing is tomorrow and I don't have my documents." (urgent — the permit consultant NOW)
- "I'm buying a new store and need a permit consultant." (new business — intake opportunity)

Currently, every call interrupts the permit consultant regardless of urgency. She becomes a switchboard operator instead of a permit consultant. The most expensive person in the business handles the most routine questions.

## The Vision: Pearl as Voice Gatekeeper

```
┌──────────────────────────────────────────────────────────────┐
│                   INBOUND CALL FLOW                           │
│                                                                │
│  Client calls business line (or the permit consultant's number)             │
│       │                                                        │
│       ▼                                                        │
│  ┌──────────────────────────────────────┐                     │
│  │  PEARL (Voice AI — ElevenLabs)       │                     │
│  │                                      │                     │
│  │  "Hi, this is Pearl, the permit consultant's       │                     │
│  │   chief of staff. How can I help?"   │                     │
│  └──────────────┬───────────────────────┘                     │
│                 │                                               │
│                 ▼                                               │
│  ┌──────────────────────────────────────┐                     │
│  │  TRIAGE DECISION                     │                     │
│  │                                      │                     │
│  │  Pearl identifies caller, looks up   │                     │
│  │  case context, assesses urgency      │                     │
│  └──────────────┬───────────────────────┘                     │
│                 │                                               │
│     ┌───────────┼───────────┐                                   │
│     ▼           ▼           ▼                                   │
│  ROUTINE    IMPORTANT    URGENT                                  │
│     │           │           │                                   │
│     ▼           ▼           ▼                                   │
│  Delegate    Take msg     Pass to                               │
│  to agent    + create     the permit consultant                               │
│              task          immediately                          │
└──────────────────────────────────────────────────────────────┘
```

## Triage Tiers

### Tier 1: ROUTINE — Delegate to Specialist Agent

Pearl handles the call entirely without the permit consultant:

| Call Type | Delegated To | Example |
|---|---|---|
| Status update | Vivian (Q&A) | "Where is my ABC transfer in the process?" |
| Document request | Ruth (Drafting) | "Can you send me a copy of my filed application?" |
| Appointment scheduling | Marcus (Comms) | "I need to schedule a time to come in" |
| General question | Vivian (Q&A) | "What do I need for a beer & wine license?" |

Pearl's response: "Let me check on that for you. [pause] Your ABC transfer is currently in the public notice period — that takes 30 days. You're 12 days in, so approximately 18 days remaining. Is there anything else I can help with?"

Call summary logged to case. the permit consultant sees it in her morning briefing but was never interrupted.

### Tier 2: IMPORTANT — Take Message + Create Task

The call needs the permit consultant's attention but not immediately:

| Call Type | Pearl's Action | Example |
|---|---|---|
| City planner callback needed | Message + task with deadline | "Planning department called about your CUP conditions" |
| Document correction needed | Message + task linked to packet | "The county needs an updated site plan" |
| Deposit/payment question | Message + task linked to finance | "Client asking about invoice #1234" |
| Client concern/complaint | Message + task flagged for review | "Client unhappy with timeline" |

Pearl's response: "I understand — the city planner needs a call back about the conditions of approval. I'll make sure the permit consultant gets this message and calls them back. What's the best number for the planner, and is there a specific deadline?"

Task created in the system. the permit consultant sees it in her next approval queue or morning briefing.

### Tier 3: URGENT — Pass to the permit consultant Immediately

The call requires the permit consultant's direct attention now:

| Call Type | Pearl's Action | Example |
|---|---|---|
| Hearing tomorrow with missing docs | Immediate pass + context summary | "Client's ABC hearing is at 9am and packet is incomplete" |
| Regulatory emergency | Immediate pass + relevant regulations | "ABC just issued a citation" |
| Time-sensitive filing deadline | Immediate pass + deadline info | "Filing deadline is today at 5pm" |
| Major client escalation | Immediate pass + relationship context | "Top client threatening to leave" |

Pearl's response: "This sounds urgent. I'm connecting you to the permit consultant right now. One moment — I'm also pulling up your case file so she has full context when she picks up."

**Pearl passes the call with a whisper:** "the permit consultant, this is Mohammed Khoury from Anaheim. His ABC hearing is tomorrow at 9am and he's missing his premises verification. I've pulled up case #5678. Connecting now."

## Caller Identification

Pearl uses multiple signals to identify callers:

1. **Caller ID lookup** — match phone number to client/contact records
2. **Voice identification** — recognize returning callers (future, with consent)
3. **Self-identification** — "Hi, this is Mohammed Khoury, I'm calling about my ABC transfer"
4. **Case number** — if the client knows it
5. **Disambiguation** — "Which Mohammed? Anaheim, Tustin, Garden Grove, or Costa Mesa?"

The disambiguation system from the [Data Model](data-model.md) is critical here — multiple clients may share names, and Pearl must route to the correct case context.

## Outbound Calls & Notifications

Pearl can also make outbound calls for proactive client communication:

| Scenario | Trigger | Pearl's Action |
|---|---|---|
| Hearing reminder | 48 hours before hearing | Call client with checklist reminder |
| Document pickup needed | Missing document flagged | Call client to request document |
| Filing status change | ABC application approved | Call client with the good news |
| Renewal approaching | 30 days before expiration | Call client about renewal |

All outbound calls are **approval-gated**:
- Pearl drafts the call script and context
- the permit consultant approves (or the call is pre-authorized for routine reminders)
- Pearl makes the call via voice API
- Call summary logged to case

## Voicemail & After-Hours

When the permit consultant is unavailable (after hours, in a meeting, on another call):

```
Call comes in → Pearl answers
  ↓
Pearl triages as above
  ↓
If Tier 3 (urgent) and the permit consultant unavailable:
  → Pearl attempts to reach the permit consultant via secondary channel (text, Slack)
  → If still unreachable, takes detailed message with urgency flag
  → Creates high-priority task
  → Notifies the permit consultant through morning briefing + immediate alert
```

## Technical Implementation Path

| Component | Technology | Status |
|---|---|---|
| **Voice synthesis** | ElevenLabs (Pearl's voice) | Active (used for demo narration) |
| **Speech-to-text** | Whisper / cloud STT | Planned |
| **Telephony** | VoIP integration (Twilio/Vonage) | Planned |
| **Caller ID matching** | Pearl's client database (396 profiles) | Database ready |
| **Triage logic** | Hermes Agent + evidence hierarchy | Skills ready |
| **Call handoff** | Transfer to the permit consultant's phone | Planned |

### Voice Persona

Pearl's voice is designed for the permit consultant's client demographic (older, established business owners):
- Warm, professional, unhurried
- Uses the client's name correctly (critical — see disambiguation)
- Never robotic or rushed
- Knows when to say "let me connect you to the permit consultant" instead of guessing

## Privacy & Consent

- Call recording requires client consent (California is a two-party consent state)
- Call summaries are stored in the case record (text, not audio, unless consent given)
- Personal/private matters are never processed by the voice system
- Caller can always request to speak directly to the permit consultant — Pearl never blocks access

## Related

- [Paperclip Orchestration](paperclip-orchestration.md) — how voice routing connects to the agent delegation chain
- [Agent Roster](agent-roster.md) — Vivian (Q&A), Marcus (Comms), Ruth (Drafting) handle delegated calls
- [Evidence Hierarchy](evidence-hierarchy.md) — call summaries are evidence-tagged
- [Approval Gates](approval-gates.md) — outbound calls are approval-gated
- [Multi-Model Inference](multi-model-inference.md) — ElevenLabs for voice synthesis
