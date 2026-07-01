# Secure Document Intake — Replacing the Internet-Facing Portal

The original client portal concept (a web app where clients log in, upload documents, and chat with agents) is a **fundamental contradiction** of the local-only isolation security model. If Pearl's workstation is local-only isolationped, it cannot serve a public-facing portal. If it serves a portal, it is not local-only isolationped.

This document resolves that contradiction with an architecture inspired by SecureDrop (used by newsrooms and intelligence agencies), data diodes (used in classified environments), and quarantine intake devices.

The core principle: **Pearl never touches the internet. A separate, disposable intake relay handles the internet-facing side. Documents cross the local-only isolation boundary through controlled, encrypted, audited transfers.**

---

## The Contradiction

```
❌ ORIGINAL PORTAL CONCEPT:

  Client Browser ──internet──▶ Pearl's Workstation (local-only isolationped)
                                    │
                                    └── contradiction: if it's local-only isolationped,
                                        no internet connection exists
```

An local-only isolationped machine has no network cable, no WiFi, no Bluetooth, no connection to the outside world. A web portal requires a network connection. These two things cannot coexist on the same machine.

---

## The Solution: Secure Intake Relay

Inspired by SecureDrop (Freedom of the Press Foundation), where a public-facing server accepts encrypted submissions, but decryption and review happen on a completely separate local-only isolationped computer.

```
┌──────────────────────────────────────────────────────────────┐
│  SECURE INTAKE ARCHITECTURE                                    │
│                                                                │
│  ┌─────────────┐                                               │
│  │  Client      │  uploads docs via:                           │
│  │              │  • Web form (TLS, no login required)          │
│  │              │  • Encrypted email (PGP/GPG)                  │
│  │              │  • Secure link (time-limited, per-case)       │
│  └──────┬───────┘                                               │
│         │                                                       │
│         │  internet (TLS encrypted)                              │
│         ▼                                                       │
│  ┌──────────────────────────────────────┐                      │
│  │  INTAKE RELAY (separate device)       │                      │
│  │                                       │                      │
│  │  • Cheap, disposable Mac mini / Pi    │                      │
│  │  • Internet-connected                  │                      │
│  │  • Receives files only                 │                      │
│  │  • NO AI processing here               │                      │
│  │  • NO Pearl software                   │                      │
│  │  • NO client database                  │                      │
│  │  • NO vault access                     │                      │
│  │  • Files encrypted at rest             │                      │
│  │  • Auto-deletes after transfer         │                      │
│  │                                       │                      │
│  │  Think: digital mailbox, not computer  │                      │
│  └──────────┬───────────────────────────┘                      │
│             │                                                   │
│             │  controlled transfer (see methods below)          │
│             ▼                                                   │
│  ┌──────────────────────────────────────┐                      │
│  │  QUARANTINE ZONE (on Pearl's machine) │                      │
│  │                                       │                      │
│  │  • Encrypted partition                │                      │
│  │  • Virus/malware scan                  │                      │
│  │  • Document type validation            │                      │
│  │  • Privacy classification scan         │                      │
│  │  • NO processing until cleared         │                      │
│  │                                       │                      │
│  │  Think: customs checkpoint             │                      │
│  └──────────┬───────────────────────────┘                      │
│             │                                                   │
│             │  after quarantine clears                          │
│             ▼                                                   │
│  ┌──────────────────────────────────────┐                      │
│  │  PEARL'S WORKSTATION (local-only isolationped)     │                      │
│  │                                       │                      │
│  │  • DGX Station or Halo                │                      │
│  │  • NO internet connection             │                      │
│  │  • Full vault, skills, agents         │                      │
│  │  • Local model inference              │                      │
│  │  • Processes documents locally        │                      │
│  │  • All PII stays here                 │                      │
│  └──────────────────────────────────────┘                      │
└──────────────────────────────────────────────────────────────┘
```

---

## How Clients Submit Documents

### Method 1: Secure Intake Form (Web)

The intake relay hosts a simple, no-login web form:

1. the permit consultant sends the client a unique case link (e.g., `https://submit.permitsnmore.org/case/abc123`)
2. Client visits the link — no account, no password, just a case-specific upload page
3. Client uploads documents (SSN docs, bank statements, entity paperwork)
4. Files are encrypted with Pearl's public key **before storage on the relay** (client-side encryption in the browser via WebCrypto API)
5. Client sees confirmation: "Documents received. the permit consultant will process them securely."
6. Files sit encrypted on the intake relay until the permit consultant/Pearl pulls them

**Key security property:** Even if the intake relay is compromised, the attacker gets encrypted blobs they cannot read. The decryption key lives only on Pearl's local-only isolationped machine.

### Method 2: Encrypted Email

1. the permit consultant gives each client a PGP/GPG public key
2. Client encrypts documents and emails them to a dedicated intake address
3. Intake relay receives the encrypted email, extracts the attachment (still encrypted)
4. the permit consultant pulls the encrypted attachment to Pearl's machine
5. Pearl decrypts locally with her private key

**Key security property:** The email passes through standard email infrastructure (Gmail, etc.) but is unreadable without Pearl's private key. The relay never has the decryption key.

### Method 3: Physical Delivery

Already documented in the local-only isolation architecture:

1. Client brings documents on a USB drive to the permit consultant's office
2. USB is scanned on the quarantine device
3. Files transferred to Pearl's machine
4. USB wiped/destroyed

**Key security property:** Zero network exposure. Documents never touch any network.

### Method 4: Silas Package Delivery (Reverse Direction)

For updates going TO Pearl (software, skills, research), Silas uses the same controlled transfer in reverse:

1. Silas builds a package on his infrastructure
2. Package is checksummed and signed
3. Brief authenticated Tailscale connection transfers the package to a staging area
4. Pearl's machine validates the checksum and signature before installing
5. Connection is severed after transfer

---

## The Intake Relay: Design Details

The intake relay is intentionally minimal — it is a **digital mailbox**, not a computer:

### Hardware
- A separate, cheap device (Raspberry Pi, Mac mini, or cloud VPS)
- Lives outside Pearl's network
- No shared filesystem, no shared credentials, no shared network
- If compromised: wipe and rebuild from image in 10 minutes

### Software
- Static web form with client-side encryption (WebCrypto API)
- No database (files only)
- No user accounts
- No AI/LLM processing
- No vault access
- Files stored encrypted on disk with AES-256
- Auto-cleanup: files deleted after 7 days or after confirmed transfer

### What the Relay Does NOT Do
- ❌ No document classification (Pearl does that locally)
- ❌ No OCR (Pearl does that locally)
- ❌ No client data storage (no names, no SSNs in any database)
- ❌ No communication with Pearl's machine (transfer is manual or one-directional)
- ❌ No agent software (no Hermes, no skills, no models)

### What the Relay IS
- A TLS-encrypted web endpoint for file upload
- A temporary encrypted storage location
- A dead drop — files wait there until physically or directionally pulled

---

## Transfer Methods (Relay → Pearl)

### Option A: USB Sneakernet (Maximum Security)

```
1. the permit consultant (or automated script) downloads encrypted files from relay
2. Files written to USB drive (FAT32, no executable permissions)
3. USB drive carried to Pearl's workstation
4. USB scanned in quarantine zone (virus scan, type validation)
5. Files decrypted on Pearl's machine
6. USB drive wiped (dd if=/dev/zero)
```

This is the SecureDrop model. The local-only isolation is physical. No network path exists between the relay and Pearl. An attacker who compromises the relay gets encrypted files they can't read.

### Option B: Data Diode (Hardware One-Way Transfer)

A **data diode** is a physical hardware device that only allows data to flow in ONE direction. It is physically impossible for data to flow backwards.

```
Intake Relay ────data diode (one-way)───▶ Pearl's Workstation
                                           (can receive, cannot send back)
```

- Hardware-enforced: no software exploit can reverse the flow
- Pearl's machine can receive files but cannot send anything back to the relay
- Used in classified military/industrial control systems
- Products: Owl Cyber Defense, Everfox, Advenica

**Key security property:** Even if the intake relay is compromised, the attacker cannot reach Pearl's machine through it. The diode only allows inbound transfer.

### Option C: Brief Authenticated Pull (Practical Compromise)

```
1. Pearl's machine briefly connects to relay via Tailscale (5 seconds)
2. Downloads all pending encrypted files
3. Verifies checksums
4. Disconnects immediately
5. Processes files in quarantine zone
```

This is less secure than USB/diode but more practical for daily operation. The connection window is tiny, authenticated, and encrypted. Pearl's machine is only "connected" for seconds, not continuously.

---

## Quarantine Zone

Before any external file enters Pearl's vault or is processed by any agent:

```
┌──────────────────────────────────────────────────────┐
│  QUARANTINE CHECKPOINT                                │
│                                                        │
│  1. MALWARE SCAN                                       │
│     • ClamAV or similar local antivirus                │
│     • File magic byte validation (not extension)       │
│     • Executable files rejected                        │
│     • Macros in Office docs stripped                   │
│                                                        │
│  2. TYPE VALIDATION                                    │
│     • Whitelist: PDF, PNG, JPG, DOCX, XLSX             │
│     • Reject: .exe, .sh, .js, .app, .bat, .py          │
│     • Max file size: 50MB                              │
│                                                        │
│  3. PRIVACY CLASSIFICATION SCAN                        │
│     • Pattern detection: SSN format, bank routing      │
│       numbers, credit card numbers                     │
│     • If PII detected: file tagged, processing         │
│       restricted to local-only models                  │
│     • Personal/private docs flagged for review         │
│                                                        │
│  4. SOURCE TAGGING                                     │
│     • File tagged with: source = client_upload         │
│     • Confidence = candidate (not yet confirmed)       │
│     • Captured_at = transfer timestamp                 │
│     • Privacy = unknown-sensitive (until classified)   │
│                                                        │
│  5. APPROVAL GATE                                      │
│     • the permit consultant reviews quarantined files                │
│     • Approves or rejects each file                    │
│     • Only approved files enter the vault/case folder  │
│                                                        │
│  CLEARED → enters Pearl's processing pipeline          │
│  REJECTED → deleted from quarantine                    │
│  FLAGGED → held for the permit consultant review                     │
└──────────────────────────────────────────────────────┘
```

---

## What the Client Experience Looks Like

The client doesn't know about local-only isolations, relays, or quarantine. They experience:

1. **the permit consultant says:** "I'll send you a secure upload link for your documents."
2. **Client gets:** A link like `https://submit.permitsnmore.org/case/ABC123`
3. **Client sees:** A clean page: "a permit consulting business — Secure Document Upload. Drag your files here."
4. **Client uploads:** SSN docs, bank statements, business formation papers
5. **Client sees:** "Documents received securely. the permit consultant will contact you within 24 hours."
6. **Behind the scenes:** Files encrypted in browser → stored on relay → the permit consultant pulls to Pearl → quarantine → approval → processing

The client portal experience (upload, see progress, communicate) can still exist as a lightweight web app on the relay — but it shows **status information only**, never actual case documents or PII:

```
Client Portal (on intake relay):
  ✅ "Your documents were received on June 17"
  ✅ "Your ABC Transfer application is being prepared"
  ✅ "Background investigation scheduled for July 2"
  📎 "Additional document needed: Current lease agreement"
  💬 Chat with Vivian (agent) — but no PII exchanged through chat
```

The portal shows **status flags and messages generated by Pearl**, not raw case data. Pearl pushes status updates to the relay (one-way); the relay displays them to the client. No sensitive data flows back through the portal.

---

## Why This Is Better Than a Traditional Portal

| Traditional Client Portal | Secure Intake Architecture |
|---|---|
| Pearl's machine is internet-facing | Pearl stays fully local-only isolationped |
| Client data passes through web server in plaintext (unless TLS+server-decrypt) | Client-side encryption before files ever leave the browser |
| Database of client credentials on the server | No client accounts, no credential database |
| Breach of portal = full data exposure | Breach of relay = encrypted blobs, no keys |
| AI processes data on internet-facing server | AI processes data only on local-only isolationped workstation |
| Server stores full case history | Server stores nothing (temporary relay only) |
| Continuous internet exposure | Seconds-long connection windows (or zero with USB/diode) |

---

## Security Properties

1. **Pearl's private key never leaves the local-only isolationped machine** — decryption is impossible without physical access to Pearl's workstation
2. **The relay has no intelligence** — it's a mailbox, not a computer. No AI, no database, no vault
3. **Client-side encryption** — files are encrypted in the browser before upload, so even the relay operator can't read them
4. **Quarantine is non-negotiable** — no external file enters Pearl's vault without passing malware scan, type validation, privacy classification, and human approval
5. **The relay is disposable** — if compromised, wipe and rebuild. No data loss (files are encrypted, originals are on client machines)
6. **Direction matters** — data flows IN to Pearl's machine (one-way). Pearl pushes status updates OUT through a controlled, audited path. No inbound connections to Pearl ever.

---

## Deployment Options

### Minimal (Phase 1)
- Intake relay: cloud VPS ($5/month) running a static form
- Transfer: the permit consultant manually downloads files to USB, carries to Pearl
- Quarantine: manual scan + visual review

### Standard (Phase 3)
- Intake relay: dedicated Mac mini at the permit consultant's office
- Transfer: brief Tailscale pull (seconds-long window)
- Quarantine: automated ClamAV + type validation + privacy scan + the permit consultant approval

### Maximum Security (Government/High-Net-Worth Clients)
- Intake relay: dedicated device with no persistent storage (RAM-only)
- Transfer: data diode (hardware one-way)
- Quarantine: full malware analysis sandbox + multi-layer validation

---

## Replaces: Client Portal Section in Security Catalog

This document supersedes the "Client Portal Security" section in [security-threat-catalog.md](security-threat-catalog.md). The original portal concept assumed Pearl's machine would serve a public-facing web app — that design is incompatible with the local-only model and has been replaced by the Secure Intake Relay architecture described here.

The client portal experience (status visibility, document upload, agent chat) still exists — but it lives on the **intake relay**, not on Pearl's machine, and it displays only **status flags and non-sensitive messages**, never raw case data or PII.

---

## References

- [SecureDrop](https://securedrop.org/) — Freedom of the Press Foundation's whistleblower submission system. Same model: internet-facing server accepts submissions, local-only isolationped machine decrypts and reviews.
- [Owl Cyber Defense — Data Diodes](https://owlcyberdefense.com/learn-about-data-diodes/) — Hardware-enforced one-way data transfer for classified environments.
- [Nextcloud Isolated Collaboration](https://nextcloud.com/blog/top-5-use-cases-for-local-only isolationped-collaboration-in-2026/) — Self-hosted collaboration with local-only isolation support.

## Related

- [Evidence Hierarchy](evidence-hierarchy.md) — quarantine tagging connects to source_of-truth levels
- [Approval Gates](approval-gates.md) — the quarantine approval checkpoint
- [Security Threat Catalog](security-threat-catalog.md) — supersedes the original portal section
- [Silas Hub Consultant](silas-hub-consultant.md) — Silas manages the intake relay as part of deployment
