# Multi-Model Inference Strategy

PermitOps OS uses a **multi-model orchestration** architecture — many different AI models, each routed to tasks they're best at, with a hybrid local/cloud design that keeps sensitive data on-premise while leveraging cloud power when connected.

The main orchestrator model is **GLM-5.2**, currently the highest-ranked model for agentic tool-calling. Silas updates model assignments as benchmarks and technology advance — the specific models listed here are the current best-in-class but will rotate as better options emerge.

## Architecture: Three-Tier Inference

```
┌──────────────────────────────────────────────────────────────────┐
│                    HERMES AGENT HARNESS                           │
│                                                                   │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │  MAIN ORCHESTRATOR: GLM-5.2                                  │ │
│  │  Attached to the Hermes Agent harness as the default model.  │ │
│  │  Routes tasks to specialist models based on task type and    │ │
│  │  security classification (local vs cloud).                   │ │
│  │  Currently #1 in tool-calling (Livebench agentic coding).   │ │
│  └──────────────────────────┬──────────────────────────────────┘ │
│                             │                                      │
│              ┌──────────────┼──────────────┐                      │
│              ▼              ▼              ▼                      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐            │
│  │  TIER 1      │  │  TIER 2      │  │  TIER 3      │            │
│  │  LOCAL       │  │  CLOUD       │  │  SPECIALIZED │            │
│  │  (always on) │  │  (when       │  │  (task-      │            │
│  │              │  │   connected) │  │   specific)  │            │
│  └──────────────┘  └──────────────┘  └──────────────┘            │
└──────────────────────────────────────────────────────────────────┘
```

## The Privacy Router

Every task is classified before model routing:

```
Task arrives at GLM-5.2 orchestrator
  ↓
Privacy classification check:
  • Filing-PII (SSN, bank, tax ID)?     → ALWAYS local, regardless of mode
  • Personal-private?                   → ALWAYS local
  • Unknown-sensitive?                  → Default local until reviewed
  • Business-operational (no PII)?      → May use cloud if connected
  ↓
Route to appropriate model in the correct tier
```

**This means SSNs, bank accounts, and personal history records are processed by local models on the workstation — never sent to any cloud API, period.** Even when Pearl is connected to the internet for form submissions, the sensitive data in those forms was processed locally.

---

## TIER 1: Local Models (Always Available — Air-Gap Capable)

These models run entirely on the local workstation. They are always available, even in full air-gap mode with zero internet connectivity.

### Target Hardware

| Workstation | GPU | Unified Memory | Max Model Size | Price | OS |
|---|---|---|---|---|---|
| **NVIDIA DGX Station** | GB300 Grace Blackwell Ultra | 748 GB (252GB HBM3e + 496GB LPDDR5X) | 1 trillion parameters | ~$15-25K | Ubuntu |
| **AMD Ryzen AI Halo** | Ryzen AI Max+ 395 | 128 GB LPDDR5x | 200B parameters | $3,999 | Windows/Linux |
| **AMD Ryzen AI Max+ PRO 495** (upcoming) | Next-gen | 192 GB | ~300B parameters | TBD | Windows/Linux |

The **NVIDIA DGX Station** is the primary target — its 748 GB of coherent memory can run models up to 1 trillion parameters entirely locally. Pearl currently runs on a Mac mini; the plan is to upgrade her to a DGX Station (or Halo as a budget alternative) once available.

Both workstations are designed for exactly this use case: always-on agentic AI running locally without cloud dependencies.

### Local Model Lineup

| Model | Parameters | Primary Use Case | Why Local |
|---|---|---|---|
| **GLM-5.2 (local weights)** | ~350B (quantized) | Orchestrator fallback, document reasoning when offline | #1 tool-calling benchmark; maintains operations in air-gap |
| **GPT-OSS-120B** | 120B | General reasoning, code generation, form logic | OpenAI open weights; near-parity with o4-mini; single-GPU |
| **GPT-OSS-20B** | 20B | Fast lightweight tasks, document classification | Lightweight; runs alongside larger models |
| **Qwen 3.5-122B** | 122B (MoE, 10B active) | Document analysis, multilingual, coding | Apache 2.0; top open-source performer |
| **Qwen 3.6-35B** | 35B (MoE, 3B active) | Fast inference, document triage, OCR post-processing | Small enough to run concurrent with larger models |
| **DeepSeek V4-Pro** | ~671B (MoE, 37B active) | Deep reasoning, regulatory analysis, complex compliance | Top open-source reasoning model |
| **Kimi K2.6** | ~260B (MoE) | Long-context document review, agent workflows | Strong agentic coding; 256K context |
| **GLM 4.7 Flash-30B** | 30B (MoE, 3B active) | High-speed document classification, triage | Extremely fast; AMD benchmarks show leadership |

### How Local Models Work Together

The workstation doesn't run one model — it runs a **stable of models**, each loaded for its specialty:

```
┌─────────────────────────────────────────────────────┐
│  WORKSTATION (DGX Station / Halo)                    │
│                                                      │
│  Always loaded (resident in memory):                 │
│  • GLM-5.2 local (orchestrator + reasoning)          │
│  • GPT-OSS-20B (fast triage + classification)        │
│                                                      │
│  Loaded on demand (swapped in for specific tasks):   │
│  • GPT-OSS-120B (heavy reasoning, code generation)   │
│  • Qwen 3.5-122B (document analysis, multilingual)   │
│  • DeepSeek V4-Pro (deep regulatory compliance)      │
│  • Kimi K2.6 (long-context case review)              │
│                                                      │
│  The orchestrator (GLM-5.2) decides which model      │
│  to invoke for each task, loads it if needed,        │
│  and returns the result.                             │
└─────────────────────────────────────────────────────┘
```

On the DGX Station with 748 GB, multiple large models can stay resident simultaneously. On the Halo with 128 GB, models are swapped more aggressively — still fast thanks to unified memory architecture.

---

## TIER 2: Cloud Models (Available When Connected)

When Pearl is connected to the internet (for form submissions, research, marketing, or any task that doesn't involve raw PII), cloud models provide additional power and specialization that local models can't match.

### Cloud Model Lineup

| Model | Provider | Primary Use Case | When Used |
|---|---|---|---|
| **GLM-5.2 (max)** | ZAI | Primary orchestrator (cloud), general reasoning, tool-calling | Day-to-day operations when connected; #1 agentic benchmark |
| **NVIDIA Nemotron 3 Ultra** | NVIDIA via Nous Portal | Regulatory document reasoning, compliance logic, long-context analysis | 1M token context for entire regulatory codes; 91% PinchBench |
| **Claude Haiku** | Anthropic | Fast drafting, email composition, document summarization | Excellent writing quality at low cost/latency |
| **GPT-5.5** | OpenAI | Code generation, engineering, dashboard development | Best-in-class code generation for tool/dashboard building |
| **GPT-mini** | OpenAI | High-volume, low-latency tasks | Cost-effective for bulk classification, quick Q&A |
| **GPT Real-Time** | OpenAI | Live conversation, voice transcription, real-time reasoning | Voice triage calls with sub-second latency |
| **Grok** | xAI | Real-time information, social media monitoring, news analysis | Marketing intelligence, competitive monitoring |
| **MiniMax** | MiniMax | Multimodal tasks, creative content, marketing copy | Marketing campaign generation, client-facing content |
| **Kimi K2.7 (cloud)** | Moonshot | Long-context coding, agent workflows | Complex engineering tasks when local Kimi K2.6 isn't enough |

### Cloud Connection Modes

```
HYBRID MODE (default):
  Local models: handle ALL PII/sensitive data
  Cloud models: handle non-sensitive ops, research, form submission
  → Best of both worlds

LOCAL-ONLY MODE:
  All models local; cloud APIs disabled
  → Used when the permit consultant wants zero data leaving the building

FULL AIR-GAP:
  Network cable unplugged; Tailscale disabled
  Only local models; no cloud access at all
  → Maximum security; Silas delivers updates via physical transfer
```

### Cloud Task Routing

When connected, the orchestrator routes tasks to cloud models:

| Task | Cloud Model | Why |
|---|---|---|
| Submit ABC form to state portal | GLM-5.2 + portal CLI | Orchestrator handles workflow; form data was prepared locally |
| Research new city regulations | Nemotron 3 Ultra | 1M context ingests entire city code at once |
| Draft marketing email | Claude Haiku | Best writing quality for client-facing communication |
| Build new dashboard feature | GPT-5.5 | Best-in-class code generation |
| Real-time phone call triage | GPT Real-Time + ElevenLabs | Sub-second voice reasoning |
| Monitor competitor activity | Grok | Real-time social/news access |
| Generate marketing campaign | MiniMax | Creative multimodal content |
| Bulk classify 500 inbox items | GPT-mini | Cost-effective high-volume |

---

## TIER 3: Specialized Models

These models handle specific non-text tasks:

| Model | Type | Use Case |
|---|---|---|
| **ElevenLabs** | Voice synthesis | Pearl's voice for phone triage, TTS narration, demo content |
| **Midjourney** | Image generation | Marketing materials, social media graphics, brand visuals for Diana (Marketing agent) |
| **Flux 2 / SDXL** | Image generation (local) | On-premise marketing visuals when cloud image APIs aren't available |
| **Tesseract / Apple Vision** | OCR (local) | Document text extraction — always local, never cloud for PII |
| **Google Cloud Geocoding** | Maps | Proximity exhibit generation (addresses only, no PII) |

---

## Legacy Website Bridge: Printing Press

Many government and municipal portals that the permit consultant files through are **legacy websites with no modern API**. PermitOps OS bridges this gap using [Printing Press](https://github.com/mvanhorn/cli-printing-press) — Matt Van Horn's Go-based CLI generator that turns any website (even those without an API) into a command-line tool that AI agents can interact with.

### How It Works

```
┌──────────────────────────────────────────────────────┐
│  LEGACY PORTAL ACCESS PIPELINE                        │
│                                                       │
│  1. Pearl needs to file a form on a city portal      │
│     (no API, 1990s-era web form)                     │
│                                                       │
│  2. Printing Press generates a Go CLI:                │
│     • Navigates the portal page structure             │
│     • Maps form fields to CLI arguments               │
│     • Handles session/auth cookies                    │
│     • SQLite sync for offline search of portal state  │
│                                                       │
│  3. Pearl calls the CLI:                              │
│     $ pnm-file-anaheim --case 5678 --form abc-221    │
│                                                       │
│  4. CLI fills and submits the form (approval-gated)   │
│     • Full audit trail of what was submitted          │
│     • Screenshot/snapshot of confirmation page        │
│     • Receipt logged to source_evidence               │
│                                                       │
│  5. Walter (Phase 3) can automate this at scale       │
│     once audited success history exists               │
└──────────────────────────────────────────────────────┘
```

### Why Go

Go (Golang) is ideal for CLI tools because:
- Single binary deployment (no runtime dependencies)
- Fast compilation and execution
- Excellent HTTP client and HTML parsing libraries
- Cross-platform (works on macOS, Linux, Windows)
- Designed for concurrent operations (file multiple forms simultaneously)

### Scope

Printing Press CLIs are generated for:
- City business license portals (50+ California cities)
- ABC portal (California Department of Alcoholic Beverage Control)
- TTB portal (federal alcohol/tobacco permits)
- Secretary of State business entity search/filing
- County clerk/recorder systems
- Board of Equalization tax registration

Each CLI is a **bridge to legacy infrastructure**, not a replacement for it. When these portals eventually modernize and add proper APIs, the CLIs are replaced with native API integrations.

---

## Model Rotation Policy

Silas (the LMS hub consultant) is responsible for keeping PermitOps OS on the best available models. The AI landscape changes rapidly — models that are top-tier today may be surpassed in months.

### Rotation Principles

1. **Silas evaluates benchmarks monthly** — tracks Livebench, BenchLM, PinchBench, SWE-bench for model quality
2. **Silas tests new models against real tasks** — not just benchmarks, but actual permit workflows
3. **Rotation is non-disruptive** — new models are tested in a sandbox before going live
4. **Local and cloud rotate independently** — a new local model doesn't require a cloud change and vice versa
5. **Backward compatibility** — old model weights are archived so the system can roll back if a new model underperforms
6. **Weekly review includes model performance** — Silas checks if any model is underperforming during the weekly Pearl review

### Current Model Assignments (June 2026)

| Role | Model | Benchmark Rationale |
|---|---|---|
| **Main orchestrator** | GLM-5.2 | #1 tool-calling (Livebench); #3 overall (BenchLM 94/100) |
| **Document reasoning** | Nemotron 3 Ultra | 1M context; 91% PinchBench; 82% IFBench |
| **Code generation** | GPT-5.5 | Best-in-class SWE-bench |
| **Voice** | ElevenLabs | Natural voice synthesis |
| **Local fallback** | GLM-5.2 local + GPT-OSS + Qwen + DeepSeek | Top open-weight models for offline |

### Why GLM-5.2 Is the Orchestrator

GLM-5.2 (from ZAI) is currently the best model for agentic tool-calling workflows:
- **#1 in function calling** (LLM Stats: #1 of 10 in "Calling" category)
- **#3 overall** on BenchLM (94/100, behind only frontier models)
- **Top agentic coding model** (Livebench, tied with Kimi K2.7 Code)
- **Available locally and via cloud** — same model family runs on both tiers
- **Hermes Agent native** — GLM-5.2 is the current default model in the Hermes Agent harness

This means the orchestrator model is consistent whether Pearl is connected to the internet (cloud GLM-5.2) or fully air-gapped (local GLM-5.2 weights). The same reasoning quality, the same tool-calling ability, just running on different hardware.

### When Models Rotate

```
Silas identifies new model (e.g., "GLM-5.3 just released, benchmarks higher")
  ↓
Sandbox test: run 20 real permit tasks on new model
  ↓
Compare results against current model
  ↓
If clearly better:
  → Update model config in Pearl's harness
  → Deliver via weekly package update
  → Monitor for 1 week before archiving old model
  ↓
If unclear or worse:
  → Note in research log; revisit next month
```

---

## Inference Cost Management

### Local (Air-Gap) Cost
- **Zero per-token cost** — models run on owned hardware
- Hardware cost amortized over 3-5 year lifespan
- Electricity (~$30-50/month for DGX Station)

### Cloud (Connected) Cost

| Model | Cost per 1M tokens (approx) | Monthly estimate |
|---|---|---|
| GLM-5.2 | $1-3 | $150-450 |
| Nemotron 3 Ultra | $1-2 (via Nous Portal) | $100-300 |
| GPT-5.5 | $2-5 | $200-450 |
| Claude Haiku | $0.25-1 | $50-150 |
| GPT-mini | $0.15-0.50 | $30-100 |
| ElevenLabs | Per-character | $30-100 |
| **Total cloud (hybrid mode)** | | **$560-1,550/mo** |

In air-gap mode: **$0 cloud cost**. The business saves $560-1,550/month when running fully offline, at the cost of reduced inference quality for non-sensitive tasks.

---

## Configuration

### Hermes Agent Model Config (excerpt)

```yaml
# Main orchestrator — attached to Hermes Agent harness
model:
  default: glm-5.2              # ZAI GLM-5.2 — #1 tool-calling
  provider: zai

# Per-task routing
model_routing:
  document_reasoning: nvidia/nemotron-3-ultra    # 1M context
  code_generation: gpt-5.5                        # SWE-bench leader
  voice_synthesis: elevenlabs                      # Natural voice
  fast_triage: gpt-mini                            # Bulk classification
  real_time: gpt-real-time                         # Voice calls
  research_monitoring: grok                        # Real-time info
  marketing_creative: minimax                      # Multimodal content
  marketing_visuals: midjourney                    # Image generation

# Local model fallback (air-gap mode)
local_models:
  orchestrator: glm-5.2-local          # Local weights
  document_reasoning: deepseek-v4-pro  # Deep reasoning
  general: gpt-oss-120b                # OpenAI open weights
  fast: gpt-oss-20b                    # Lightweight
  multilingual: qwen-3.5-122b          # Apache 2.0
  classification: glm-4.7-flash-30b    # Fast MoE

# Privacy routing
privacy_routing:
  filing_pii: local_only              # SSN, bank, tax ID → always local
  personal_private: local_only         # Birth certs, marriage → always local
  unknown_sensitive: local_default     # Unknown → local until reviewed
  business_operational: cloud_if_connected  # Case data (no PII) → cloud OK
```

### Legacy Portal CLI Example (Printing Press)

```bash
# Generated CLI for a city portal (Go binary)
$ pnm-file-anaheim \
    --case-id "case-5678" \
    --form-type "abc-transfer" \
    --dry-run  # preview what will be submitted

# Actual filing (approval-gated)
$ pnm-file-anaheim \
    --case-id "case-5678" \
    --form-type "abc-transfer" \
    --approved-by "human-operator" \
    --approval-token "uuid"
```

## Related

- [Air-Gapped Security](air-gapped-security.md) — how local models enable offline operation
- [Silas Hub Consultant](silas-hub-consultant.md) — Silas manages model rotation
- [Weekly Feedback Loop](weekly-feedback-loop.md) — model performance reviewed weekly
- [Architecture Overview](overview.md) — system data flow
- [Agent Roster](agent-roster.md) — which models serve which agents
