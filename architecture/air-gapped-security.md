# Air-Gapped Security — Local-First & Offline-Capable

Permit consulting involves some of the most sensitive personal and financial data in any industry. PermitOps OS is designed to operate in environments where **zero client data may touch the internet** — including social security numbers, bank account details, tax IDs, personal history records, and fingerprint data.

This document describes the security architecture that makes local-first, air-gapped operation possible.

## The Threat Model

### What's at Stake

| Data Type | Examples | Regulatory Driver |
|---|---|---|
| **SSN / Tax ID** | Personal identification for ABC applications | CA ABC Act, IRS |
| **Bank/Financial** | Proof of funds, account numbers, financial statements | GLBA, CA CCPA |
| **Personal History** | ABC Form 407 — residence history, employment, references | CA ABC Act |
| **Fingerprint Data** | Biometric cards for background checks | CA Penal Code |
| **Business Ownership** | Articles of incorporation, operating agreements, ownership stakes | CA Corporations Code |
| **Premises/Lease** | Lease agreements, property deeds, site plans | Contractual |

A single data breach involving this information could:
- Destroy client trust permanently
- Expose the consultant to regulatory liability
- Result in identity theft for clients
- Invalidate pending applications

### Why Cloud-Only AI Is Unacceptable

Most "AI for business" products send all data to cloud APIs for processing. For a permit consultant handling SSNs and bank information, this is a **non-starter**:

1. Data sent to cloud APIs may be stored, logged, or used for training
2. Cloud providers may be subpoenaed, breached, or change terms of service
3. Compliance frameworks (CCPA, GLBA) require data minimization
4. Clients may explicitly require that their data never leaves the local network
5. the permit consultant herself wants maximum protection for her clients' most sensitive information

## Security Tiers

PermitOps OS supports three security tiers, configurable per deployment:

### Tier 1: Hybrid (Default)

```
┌──────────────────────────────────────────────────┐
│  HYBRID MODE                                      │
│                                                    │
│  ┌──────────────────────────────────────────┐    │
│  │  Dedicated Hardware (Mac mini)            │    │
│  │                                           │    │
│  │  Sensitive data:                          │    │
│  │  • SSNs, bank info, PII                   │    │
│  │  • Personal history records               │    │
│  │  • Fingerprint data                       │    │
│  │  → Processed LOCALLY only                 │    │
│  │  → Never sent to cloud APIs               │    │
│  │  → Encrypted at rest (FileVault + 0600)   │    │
│  │                                           │    │
│  │  Non-sensitive operations:                │    │
│  │  • Case management, workflow logic        │    │
│  │  • Proximity exhibits (addresses only)    │    │
│  │  • Email drafting (without PII)           │    │
│  │  → May use cloud inference (routed)       │    │
│  └──────────────────────────────────────────┘    │
│                                                    │
│  Tailscale VPN for Silas package delivery only    │
│  No public internet exposure for client data      │
└──────────────────────────────────────────────────┘
```

**How it works:** The privacy classification system (see [Data Model](data-model.md)) automatically routes data:
- Documents tagged `filing-required-PII` or `personal-private` → **local processing only**
- Documents tagged `business-operational` → may use cloud inference (no PII sent)
- Documents tagged `unknown-sensitive` → held for review, processed locally by default

### Tier 2: Local-Only Inference

```
┌──────────────────────────────────────────────────┐
│  LOCAL-ONLY MODE                                  │
│                                                    │
│  ALL inference runs on local hardware.             │
│  No cloud API calls for any data.                  │
│                                                    │
│  ┌──────────────────────────────────────────┐    │
│  │  Local Model Stack                        │    │
│  │                                           │    │
│  │  • Local LLM (Nemotron local / open model)│    │
│  │  • Local OCR (Tesseract / Apple Vision)   │    │
│  │  • Local embeddings for search            │    │
│  │  • Local geocoding (offline maps)         │    │
│  └──────────────────────────────────────────┘    │
│                                                    │
│  Tailscale VPN: ONLY for Silas updates             │
│  No other outbound connections                     │
└──────────────────────────────────────────────────┘
```

**When to use:** When the business owner wants zero data leaving the machine for any reason, but still needs periodic Silas updates.

### Tier 3: Full Air-Gap

```
┌──────────────────────────────────────────────────┐
│  FULL AIR-GAP MODE                                │
│                                                    │
│  ┌──────────────────────────────────────────┐    │
│  │  Dedicated Hardware (Mac mini)            │    │
│  │                                           │    │
│  │  • NO network connection                  │    │
│  │  • NO Tailscale                           │    │
│  │  • NO internet access                     │    │
│  │  • ALL inference local                    │    │
│  │  • ALL storage local                      │    │
│  └──────────────────────────────────────────┘    │
│                                                    │
Silas updates via:
  • Physical USB package delivery (checksummed)
  • Or brief authenticated connection for verified
    package transfer only, then disconnect

**Client document intake** (when air-gapped):
  See [secure-document-intake.md](secure-document-intake.md) for the full architecture.
  Clients upload to a separate, disposable intake relay (internet-facing).
  Files are encrypted client-side before upload, transferred to Pearl's
  machine through USB/diode/brief-pull, and processed in quarantine.
  Pearl never touches the internet.
```
```

**When to use:** Maximum security. Government clients, high-net-worth clients, or clients who explicitly require air-gapped operations. Also the fallback if any security breach is suspected.

## Local Inference Stack

When operating in local-only or air-gap mode, PermitOps OS uses:

| Component | Cloud (Hybrid) | Local (Air-Gap) |
|---|---|---|
| **Document reasoning** | Nemotron 3 Ultra (Nous Portal) | Local model (Nemotron local if hardware available, otherwise reduced-capability open model) |
| **Code/engineering** | GPT-5.5 (OpenAI) | Local model (reduced capability) |
| **OCR** | Cloud OCR API | Tesseract / Apple Vision Framework |
| **Geocoding** | Google Maps API | Local geocoder / offline OSM data |
| **Embeddings/search** | Cloud embedding API | Local embedding model |
| **Voice synthesis** | ElevenLabs API | Local TTS (reduced quality) |

**Graceful degradation:** In air-gap mode, the system loses some inference quality but retains ALL safety mechanisms:
- ✅ All approval gates still work
- ✅ All vault operations work
- ✅ All privacy classifications work
- ✅ All audit trails work
- ✅ All document watchers work
- ✅ All cron jobs work
- ⚠️ Document reasoning quality may be lower
- ⚠️ Code generation capability reduced
- ⚠️ Voice quality reduced

The core insight: **safety mechanisms are local. Only inference quality depends on cloud.**

## Privacy Classification Enforcement

The privacy classification system is the technical mechanism that prevents sensitive data from reaching cloud APIs:

```python
# Simplified routing logic
def route_for_processing(document):
    classification = classify_document(document)
    
    if classification in ["filing-required-PII", "personal-private"]:
        # ALWAYS local, regardless of mode
        return process_locally(document)
    
    if classification == "unknown-sensitive":
        # Default to local until reviewed
        return process_locally(document)
    
    if classification == "business-operational":
        if current_mode == "hybrid":
            return route_to_cloud_if_no_pii(document)
        else:
            return process_locally(document)
    
    if classification == "personal-safe-context":
        # Context only, never structured data
        return process_locally(document)
```

### Testable Privacy Rules

Phase 0 requires testable privacy exclusions:
- Path patterns: `_personal/`, `_secure/`, `_quarantine/`
- Content patterns: SSN format, bank routing numbers, date-of-birth patterns
- Fixture tests proving private files NEVER appear in:
  - API responses
  - Search results
  - AI prompts
  - Packet assembly
  - Activity logs
  - Data exports

## Physical Security

| Control | Implementation |
|---|---|
| **Hardware** | Dedicated Mac mini, no shared access |
| **Disk encryption** | macOS FileVault (AES-256) |
| **File permissions** | Database files `0600`, sensitive dirs restricted |
| **Access control** | Local session auth required for real data |
| **Screen lock** | Auto-lock + screensaver password |
| **Backup** | Encrypted local backup + encrypted offsite (if approved) |
| **Physical access** | Hardware in secure location (the permit consultant's office) |
| **Network** | Tailscale VPN (no public ports); air-gap mode disables all network |

## Silas Managed Security

Silas (the LMS hub consultant) is responsible for:

1. **Monitoring** — periodic security scans, log review, privacy gate tests
2. **Patching** — delivering security updates through the package pipeline
3. **Air-gap execution** — Silas can trigger air-gap mode remotely if a breach is suspected
4. **Audit review** — weekly review of access logs, failed logins, unusual activity
5. **Compliance** — ensuring the deployment meets CCPA/GLBA/ABC requirements

See [Silas Hub Consultant](silas-hub-consultant.md) for the full management model.

## Why This Matters

Most AI business tools cannot operate without cloud connectivity. They fundamentally cannot serve industries where:
- Client data legally cannot leave the local network
- The business owner requires maximum data sovereignty
- Regulatory compliance demands air-gapped processing
- Clients explicitly require that their SSN/bank data never touches a cloud API

PermitOps OS is designed from the ground up to serve these industries. The local-first architecture is not a limitation — it is the reason this system can be trusted with the most sensitive data in a permit consultant's office.

## Related

- [Silas Hub Consultant](silas-hub-consultant.md) — Silas manages air-gap mode and package delivery
- [Evidence Hierarchy](evidence-hierarchy.md) — privacy classifications are evidence-tagged
- [Data Model](data-model.md) — `privacy_classifications` table and exclusion rules
- [Approval Gates](approval-gates.md) — all risky data operations require approval
- [Weekly Feedback Loop](weekly-feedback-loop.md) — security posture reviewed weekly
