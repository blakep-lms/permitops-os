# Memory & Wiki System — The LLM Second Brain

PermitOps OS implements an enhanced version of **Andrej Karpathy's LLM Wiki pattern** — a system where the AI agent incrementally builds and maintains a persistent, interlinked knowledge base instead of rediscovering knowledge from scratch on every query.

This is the architecture that makes Pearl actually smart. Not chatbot-smart (regenerating answers from raw context each time) but **institutional-smart** — accumulating knowledge over months and years, cross-referencing it, flagging contradictions, and getting better the longer she operates.

## The Core Problem Karpathy Identified

> "Most people's experience with LLMs and documents looks like RAG: you upload a collection of files, the LLM retrieves relevant chunks at query time, and generates an answer. This works, but the LLM is rediscovering knowledge from scratch on every question. There's no accumulation."

> "Historically, personal wikis failed because human maintenance burden grew faster than the value. By outsourcing the 'grunt work' (bookkeeping, filing, summarizing) to the LLM, the cost of maintenance approaches zero."

Permit consulting is the perfect test case. the permit consultant has 30 years of institutional knowledge in her head. Every client has a story — family relationships, business partnerships, past filing complications, personality quirks. That knowledge is currently stored in folders, email chains, and the permit consultant's memory. When the permit consultant retires or scales, that knowledge dies or scatters.

Pearl's wiki system captures it, structures it, and compounds it.

## Karpathy's 3-Layer Architecture (Original)

| Layer | Role | Karpathy's Description |
|---|---|---|
| **Raw Sources** | Immutable source documents | Articles, papers, filings, emails |
| **The Wiki** | LLM-generated, LLM-owned markdown | Summaries, entity pages, cross-references |
| **The Schema** | Configuration that disciplines the LLM | Structure, conventions, workflows (CLAUDE.md / AGENTS.md) |

Core operations: **Ingest** (read source → extract → update wiki), **Query** (answer from wiki), **Lint** (health check for contradictions/stale/orphan pages).

## Our 7 Layers (What We Added)

We extended Karpathy's pattern with layers that make it work for a real regulated business — not just personal notes, but an operating system where wrong information costs real money.

```
┌──────────────────────────────────────────────────────────┐
│                    PEARL'S WIKI SYSTEM                     │
│                                                            │
│  Layer 1: RAW SOURCES (immutable evidence)                 │
│  Source documents, filings, emails, PDFs, court records    │
│  → Never modified; referenced by path/hash                 │
│                                                            │
│  Layer 2: SOURCE-OF-TRUTH METADATA (our extension)         │
│  Every fact tagged with provenance: source type,           │
│  confidence level, captured by/when, privacy class         │
│  → See evidence-hierarchy.md for the 7 levels              │
│                                                            │
│  Layer 3: YAML FRONTMATTER (our extension)                 │
│  Every page has structured metadata: slug, tags,           │
│  status, priority, privacy, source_status, created,        │
│  updated, relationships, confidence                        │
│  → Machine-parseable; enables Dataview queries             │
│                                                            │
│  Layer 4: THE WIKI (Karpathy's core layer)                 │
│  LLM-generated markdown pages: client profiles, city       │
│  jurisdictions, case notes, workflow docs, relationship    │
│  maps, review queues                                       │
│  → Interlinked with [[wikilinks]]; LLM-owned               │
│                                                            │
│  Layer 5: CANDIDATE vs APPROVED TRUTH (our extension)      │
│  Every page and fact is labeled:                           │
│  • approved — the permit consultant/the LMS owner confirmed                      │
│  • candidate — Pearl's inference, not yet confirmed        │
│  • needs-review — uncertain, awaiting human check          │
│  • rejected — disproven or outdated                        │
│  → Prevents hallucinations from becoming filing facts      │
│                                                            │
│  Layer 6: CHRONOLOGICAL JOURNAL (our extension)            │
│  Append-only daily log of what happened, decisions made,   │
│  artifacts created — the timeline of the business          │
│  → Pearl writes to journal first, then updates wiki        │
│                                                            │
│  Layer 7: SCHEMA / AGENTS.md (Karpathy's schema layer)     │
│  Rules for how Pearl maintains the wiki: naming            │
│  conventions, frontmatter standards, tag taxonomy,         │
│  ingestion rules, when to create vs update pages           │
│  → Lives at vault root as AGENTS.md                        │
└──────────────────────────────────────────────────────────┘
```

## How Pearl Maintains the Wiki

### Daily Operations

Pearl runs a continuous loop of wiki maintenance:

**Morning Briefing** (`pnm-morning-briefing` skill):
1. Scan the vault — what's changed since yesterday
2. Check review queues — what needs the permit consultant's attention
3. Check document watcher — what new documents arrived
4. Check inbox — what emails came in
5. Generate morning brief from vault state
6. Dispatch autonomous work (classifications, drafts, audits)

**Throughout the Day** (`pnm-remember` skill):
1. When Pearl learns something new, she writes it to the wiki
2. She detects memorable facts in conversation: "Mohammed's daughter just graduated UCLA"
3. She writes to the correct vault note with source tags and timestamp
4. She creates wikilinks to related entities
5. She tags the fact with confidence level

**End of Day** (`pnm-end-of-day` skill):
1. Capture what happened today
2. Log decisions made
3. List artifacts created
4. Note open questions
5. Write to journal

**Session Rituals** (weekly cycle):
- `pnm-stand-up` — morning orientation from vault
- `pnm-journal` — append chronological truth
- `pnm-wrap-up` — close session, surface blockers
- `pnm-session-handoff` — exact next moves for next session
- `pnm-weekly-review` — summarize week, escalate stale items

### Frontmatter Standard

Every page Pearl creates follows a consistent frontmatter pattern:

```yaml
---
slug: tustin-7eleven-corp-14090red
display_name: "7-Eleven Tustin"
short_ref: "7-Eleven (Tustin)"
type: client-profile
tags: [client, 7-eleven, tustin, abc-transfer, active]
status: active
priority: top-20
privacy: business-operational
source_status: candidate
confidence: needs-review
created: 2026-06-04
updated: 2026-06-17
city: "[[Tustin]]"
license_type: "ABC On-Sale General"
relationships:
  - "[[anaheim-7eleven-corp-5678]]"  # sister store
  - "[[ontario-7eleven-inc]]"          # same corporation
---

# 7-Eleven #14090 — Tustin, CA

## Quick Recognition
- Store #14090, 14090 Red Hill Ave, Tustin CA 92780
- ABC License #XXXXXX (on-sale general)
- Owner: [redacted for public repo]

## Business Context
...

## Filing History
...

## Source Evidence
- [[source-evidence/abc-license-tustin-14090]]
- [[source-evidence/cup-findings-tustin-2024]]
```

### Wikilink Topology

Pearl's vault is a graph, not a folder hierarchy. Everything connects:

```
Permit Consultant (center node)
  ├── [[client-profiles]]
  │     ├── [[tustin-7eleven-corp-14090]]
  │     │     ├── [[Tustin]] (city page)
  │     │     ├── [[ABC Transfer Workflow]]
  │     │     └── [[anaheim-7eleven-corp-5678]] (sister store)
  │     └── [[anaheim-7eleven-corp-5678]]
  │           ├── [[Anaheim]] (city page)
  │           └── [[ABC Transfer Workflow]]
  ├── [[cities]]
  │     ├── [[Tustin]] ← city page with portal links, forms, contacts
  │     ├── [[Anaheim]]
  │     └── [[56 other cities]]
  ├── [[workflows]]
  │     ├── [[ABC Transfer Workflow]]
  │     ├── [[CUP Workflow]]
  │     └── [[7 workflows]]
  └── [[review-queues]]
        ├── [[File Organization v2]]
        ├── [[the permit consultant Interview Decision Log]]
        └── [[New Document Intake Queue]]
```

When Pearl adds a new client, she doesn't just create one page — she creates the entity page, links it to the city page, links it to related entities, updates the city page to show the new client, and updates the index. Karpathy called this "touching 15 files in one pass."

### Tag Taxonomy

Pearl uses a structured tag system:

| Category | Tags | Purpose |
|---|---|---|
| Entity type | `client`, `city`, `case`, `entity`, `premises`, `vendor`, `attorney` | Classify page type |
| Status | `active`, `backburner`, `closed`, `prospect`, `not-a-fit` | Business lifecycle |
| Workflow | `abc-transfer`, `cup`, `business-license`, `ttb`, `loi` | Active permit type |
| Confidence | `approved`, `candidate`, `needs-review`, `rejected` | Truth state |
| Privacy | `business-operational`, `filing-pii`, `personal-safe`, `personal-private` | Data classification |
| Priority | `top-10`, `top-20`, `top-35`, `standard`, `backburner` | Relationship depth |

### The Lint Cycle

Karpathy described periodic health checks. Pearl implements this through the weekly review and Silas audit:

1. **Contradiction detection** — Pearl scans for conflicting facts across pages (e.g., two pages showing different addresses for the same entity)
2. **Stale flag detection** — pages not updated in 90+ days flagged for review
3. **Orphan detection** — pages with no incoming wikilinks (lost knowledge)
4. **Missing cross-reference detection** — pages that should link but don't (same address, same owner, same city)
5. **Confidence degradation** — candidate facts that have sat unconfirmed for 30+ days are escalated to review queue

## Current Vault Scale (June 2026)

| Metric | Count |
|---|---|
| Client profiles | 396 |
| City jurisdiction pages | 56 |
| Documented workflows | 7 |
| Relationship edges | Hundreds (family, business, geographic) |
| Canonical case folders | 289 |
| Review queues | 5+ active |
| Skills maintaining vault | 6 (journal, remember, wiki-ops, morning-briefing, weekly-review, end-of-day) |
| Journal entries | Daily (since 2026-06-04) |
| Weekly reviews | Weekly (since 2026-06-14) |

## Why This Is Different From RAG

| Traditional RAG | Pearl's Wiki System |
|---|---|
| Rediscover knowledge from scratch each query | Accumulate knowledge over time |
| No memory between sessions | Persistent, compounding memory |
| Finds chunks, doesn't synthesize | Synthesizes across 396 profiles and 56 cities |
| No contradiction detection | Flags conflicts between sources |
| Human maintains the knowledge base | LLM maintains it; human curates |
| Starts fresh every session | Gets smarter every day |
| No provenance | Every fact has source, confidence, and privacy tags |

## Connection to Other Systems

- **Evidence Hierarchy** ([evidence-hierarchy.md](evidence-hierarchy.md)) — the 7-level source-of-truth feeds into Layer 2 metadata
- **Approval Gates** ([approval-gates.md](approval-gates.md)) — candidate vs approved truth connects to the approval queue
- **Silas Hub** ([silas-hub-consultant.md](silas-hub-consultant.md)) — Silas reviews wiki quality during weekly reviews
- **Weekly Feedback Loop** ([weekly-feedback-loop.md](weekly-feedback-loop.md)) — Pearl's weekly review is a wiki lint cycle
- **Data Model** ([data-model.md](data-model.md)) — vault is the "soft knowledge" layer; database is the "hard data" layer

## Karpathy's Words

> "Obsidian is the IDE; the LLM is the programmer; the wiki is the codebase."

Pearl is the programmer. The vault is her codebase. And unlike a chatbot that resets every conversation, Pearl's codebase has been compounding for 6 weeks and will compound for years — eventually holding the complete institutional knowledge of a 30-year permit consulting business.

## References

- [Andrej Karpathy's LLM Wiki gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) — the original pattern spec
- [Obsidian](https://obsidian.md/) — the markdown knowledge base Pearl uses
- Vannevar Bush, "As We May Think" (1945) — the Memex vision this realizes

## Related

- [Architecture Overview](overview.md)
- [Evidence Hierarchy](evidence-hierarchy.md)
- [Skills](../skills/README.md) — the 6 skills that maintain the wiki
- [Data Model](data-model.md) — structured data vs soft knowledge split
