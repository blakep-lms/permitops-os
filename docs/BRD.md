# Business Requirements Document (BRD)
## PermitOps OS — Permit Consulting AI Operating System

**Version:** 1.0 (sanitized for public repo)
**Status:** Active development
**Source:** Derived from the PNM business requirements. All client financial data removed.

---

## 1. Business Context

Permit consulting is a multi-billion dollar industry run on spreadsheets, email chains, and institutional knowledge passed down through decades of experience. Consultants manage hundreds of active cases across dozens of California cities, each with different forms, deadlines, and requirements. Documents get lost. Deadlines get missed. Nobody knows the status of anything without a phone call.

The tools that exist — generic CRMs, marketing platforms, spreadsheets — don't understand the specific realities of permit consulting: the relationship between people, entities, stores, premises, and cities; document intake and packet readiness; official source hierarchies; privacy classifications; and the need for human approval before any consequential action.

PermitOps OS is purpose-built for this industry. It is the first product in a broader vision from Linear Marketing Solutions (LMS): deploying AI business operators across multiple service verticals using a hub-and-spoke model where a central AI consultant (Silas) manages and improves each business agent.

## 2. Business Opportunity

### Market Size
- California has thousands of businesses requiring alcohol, zoning, and business license permits annually
- ABC alone processes 40,000+ license applications per year
- Permit consultants charge $2,000-$10,000+ per case
- Top consultants manage $1-2M+ in annual revenue

### Pain Points Addressed
| Pain Point | Annual Impact | PermitOps OS Solution |
|---|---|---|
| Dropped/forgotten deals | $40K/yr per consultant | Automated pipeline tracking + reactivation campaigns |
| Underpriced services | $30-100K/yr | Pricing intelligence on every quote using evidence hierarchy |
| Manual document handling | 250-500 hrs/yr | Automated document classification + canonical filing |
| Manual proximity exhibits | 25-50 hrs/yr | Automated Google Cloud proximity exhibit pipeline |
| Phone interruptions | 2-3 hrs/day | Voice AI triage with specialist delegation |
| Filing errors/rejections | 2-4 weeks delay per error | Packet readiness checks + inconsistency auditing |
| Sensitive data risk | Reputation + liability | Local processing, privacy classifications, approval gates |

**Total addressable impact: $200-400K/yr per consultant.**

## 3. Stakeholders

| Stakeholder | Role | Interest |
|---|---|---|
| **Permit Consultant** | Permit consultant / business owner | Wants AI to handle operations so she can focus on relationships and deals |
| **LMS owner** | LMS owner / Silas deployer | Wants a repeatable product pattern for deploying AI operators across businesses |
| **Silas** | AI consultant (LMS Hub) | Deploys, manages, and improves Pearl; weekly reviews; security monitoring |
| **Pearl** | AI Chief of Staff (PermitOps agent) | Runs daily operations, delegates to specialists, reports to Silas |
| **Clients** | 7-Eleven franchisees, restaurant owners | Want timely updates, accurate filings, data protection |
| **NVIDIA** | Hardware/inference sponsor | Nemotron 3 Ultra for document reasoning and compliance |
| **Stripe** | Payment platform sponsor | Invoice creation, spend gates, audit trail |
| **Nous Research** | Agent runtime sponsor | Hermes Agent platform for skills, memory, multi-agent coordination |

## 4. Business Objectives

### Primary Objectives
1. **Recover lost revenue** — automated pipeline tracking, deposit follow-up, and reactivation campaigns recover $200-400K/yr
2. **Reduce operational time** — document automation, proximity exhibits, and voice triage save 250-500+ hrs/yr
3. **Eliminate filing errors** — packet readiness checks and evidence-grounded consistency auditing reduce rejection rate to <2%
4. **Protect sensitive data** — privacy classification and local processing keep SSNs, bank info, and PII out of cloud workflows
5. **Scale the business** — multi-agent team handles growing caseload without proportional headcount increase

### Secondary Objectives
6. **Create a repeatable product pattern** — PermitOps OS as first vertical; FieldOps OS (a field-services business) and ServiceOps OS (future clients) follow the same architecture
7. **Establish LMS as AI business operator company** — Silas hub model positions LMS as the managed service for deploying and improving AI business agents
8. **Build cross-business intelligence** — learnings from each deployment compound across all business agents

## 5. Revenue Model (PermitOps OS as LMS Product)

### Tier 1: Deployment Fee
- One-time setup: deploy PermitOps OS on client hardware
- Includes: Hermes Agent configuration, skill package, vault setup, proximity exhibit pipeline, dashboard
- Delivered by Silas (automated) with the LMS owner oversight

### Tier 2: Monthly Management Retainer
- Weekly Silas ↔ business agent review cycle
- Package updates (skills, research, security patches)
- Cross-business intelligence routing
- Security monitoring and privacy governance
- Strategic consulting based on business performance data

### Tier 3: Usage-Based (Inference/API Costs)
- Pass-through: Nemotron 3 Ultra inference, Google Cloud APIs, Stripe transaction fees
- Transparent markup or at-cost depending on tier
- Sensitive-data mode uses local inference and strict approval gates

## 6. Competitive Landscape

| Competitor Type | Weakness | PermitOps OS Advantage |
|---|---|---|
| Generic CRM (Salesforce, HubSpot) | Doesn't understand permit workflows, entities, packets | Purpose-built for permit consulting |
| GHL / Marketing platforms | Marketing-focused, not operations | Operations-first, approval-gated |
| Spreadsheets | Manual, error-prone, no intelligence | Fully automated with AI reasoning |
| Other "AI for business" tools | Cloud-first, weak approval gates | Local-first sensitive-data handling, human-in-the-loop |
| Manual / no system | Everything depends on the consultant's memory | Never forgets a detail, cites every source |

## 7. Key Differentiators

1. **Evidence hierarchy** — 7-level source-of-truth system means every fact is verifiable. No hallucinations become filings.
2. **Approval gates** — Pearl prepares, the human decides. No autonomous sends, filings, or spending.
3. **Privacy-routed** — sensitive client data stays on controlled local systems and only approved non-sensitive work leaves the environment.
4. **Multi-agent team** — specialist agents handle different task types, coordinated through Paperclip.
5. **Silas hub management** — weekly reviews, continuous improvement, cross-business intelligence. The system gets smarter every week.
6. **Voice escalation** — Pearl triages phone calls, delegates routine questions, protects the consultant's time.
7. **Proximity exhibits** — automated court-ready radius maps using Google Cloud APIs. Saves 25-50 hours/year.
8. **Real deployment** — not a demo. Running on dedicated hardware for a real $1-2M/year consulting business with 396 client profiles across 56 cities.

## 8. Risk Analysis

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| AI filing error causes regulatory issue | Low | High | Evidence hierarchy + approval gates + human actor-of-record |
| Sensitive data exposure | Low | Critical | Privacy classifications + local-only processing for PII + approval gates |
| Client resistance to AI | Medium | Medium | Pearl named persona, human always available, gradual trust building |
| Model provider outage | Medium | Low | Multi-model routing + local model fallback |
| Regulatory changes | Medium | Medium | Forms librarian + weekly Silas research updates |
| Hardware failure | Low | High | Encrypted backups + Tailscale remote access + recovery procedures |

## 9. Success Criteria

| Criterion | Measurement | Target |
|---|---|---|
| Filing accuracy | Rejection rate due to data errors | <2% |
| Revenue recovery | Dropped deals saved + deposits collected | $200K+/yr |
| Time savings | Hours saved per year | 250-500 hrs/yr |
| Data security | PII exposure incidents | Zero |
| Client satisfaction | Retention rate | >95% |
| Consultant adoption | Daily active usage of Pearl | 5+ days/week |
| Silas review cycle | Weekly reviews completed | 100% |
| Cross-business value | Learnings applied to new verticals | FieldOps OS deployment within 6 months |

## 10. Timeline

| Milestone | Target |
|---|---|
| Hackathon submission | ✅ Complete (2026-06-17) |
| Phase 0: Secure foundation | 2026-07 |
| Phase 1: Local core MVP | 2026-08 |
| Phase 2: Data integrity | 2026-09 |
| Phase 3: Multi-user auth | 2026-10 |
| Phase 4: Automation & AI integration | 2026-11 |
| Phase 5: Production readiness | 2026-12 |
| Multi-agent specialist rollout | 2026-10 to 2027-01 |
| FieldOps OS (Marvin) deployment | 2027 Q1 |
| Voice escalation live | 2027 Q1 |

## 11. Vision

PermitOps OS is the first vertical in a broader LMS strategy:

```
Pearl / a permit consulting business  →  PermitOps OS  (permit consulting) ✅ In production
Marvin / a field-services business  →  FieldOps OS  (field services) 🔜 Next
Future LMS clients  →  ServiceOps OS  (general services) 📋 Planned
```

Each vertical gets the same architecture: Hermes Agent brain + domain-specific skills + approval gates + Stripe integration + multi-agent team + Silas hub management.

The end goal: LMS deploys and manages AI business operators for service businesses across every vertical — each one local-first, privacy-routed, and continuously improved through the weekly Silas review cycle. One hub, many businesses, compound intelligence.
