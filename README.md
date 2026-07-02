# NetTrap — AI-Powered Honeypot Platform (NIS2)

> **Threat detection that protects your business and supports NIS2 compliance.**
> 7 multi-protocol honeypots, behavioural analysis with AI, and NIS2 Article 23
> incident reporting. Built at **Expertmode**.

![status](https://img.shields.io/badge/status-in%20production-success)
![python](https://img.shields.io/badge/Python-3.12-blue)
![NIS2](https://img.shields.io/badge/NIS2-Article%2023-orange)
![license](https://img.shields.io/badge/source-private-lightgrey)

> ℹ️ This repository is a **technical showcase**. The source code is property
> of Expertmode and remains private. Here I describe the architecture, the
> engineering decisions and the features — without exposing implementation,
> detection logic or configuration.

---

## The problem

The **NIS2** directive requires thousands of European SMEs to detect and
report cybersecurity incidents within tight deadlines (early warning within
24h, notification within 72h — Article 23). Most have no SOC and no budget
for enterprise solutions (€7,500+/year per device).

**NetTrap** solves this: an affordable appliance that lures attackers into
decoys (honeypots), analyses their behaviour with AI, blocks them
automatically, and generates the NIS2 compliance documentation.

---

## Demo

<!-- Replace these with your own screenshots (demo data only:
     no real IPs, no credentials, no customer data).
     Put the PNGs in docs/img/ with these names. -->

| Real-time dashboard | NIS2 compliance report |
|:---:|:---:|
| ![Dashboard](docs/img/dashboard.png) | ![NIS2 report](docs/img/nis2-report.png) |

| Alert management | Profile & MFA |
|:---:|:---:|
| ![Alerts](docs/img/alerts.png) | ![MFA](docs/img/mfa.png) |

---

## Architecture (overview)

```
                       ┌─────────────────────────┐
   Attacker  ──────►   │   7 multi-protocol       │
                       │   honeypots (decoys)     │
                       │ SSH HTTP SMB FTP Telnet  │
                       │ MySQL RDP + canary       │
                       └────────────┬─────────────┘
                                    │ events
                       ┌────────────▼─────────────┐
                       │   Event pipeline          │
                       │  (memory + persistence)   │
                       └────────────┬─────────────┘
              ┌─────────────────────┼─────────────────────┐
   ┌──────────▼─────────┐ ┌─────────▼────────┐ ┌──────────▼─────────┐
   │   5 AI agents      │ │   Alerting        │ │  NIS2 compliance   │
   │ Scanner Profiler   │ │  Email + MQTT     │ │  CERT notification │
   │ Deceiver Resolver  │ │                   │ │  Audit trail       │
   │ Maestro            │ │                   │ │  Retention         │
   └────────────────────┘ └───────────────────┘ └────────────────────┘
                                    │
                       ┌────────────▼─────────────┐
                       │  HTTPS web dashboard      │
                       │  JWT · RBAC · MFA TOTP    │
                       └──────────────────────────┘
```

### The 5 AI agents
| Agent | Role |
|-------|------|
| **Scanner** | Classifies the threat (attack type, severity) from behavioural features |
| **Profiler** | Maps MITRE ATT&CK techniques and detects anomalies against a baseline |
| **Deceiver** | Generates credible responses to keep the attacker engaged |
| **Maestro** | Orchestrates the pipeline and decides the action (engage / block) |
| **Resolver** | Applies automatic blocking (firewall + shared blocklist) |

---

## Key features

- **7 multi-protocol honeypots** + canary tokens
- **Behavioural analysis with AI** (5 coordinated agents)
- **Automatic blocking** of attackers
- **Real-time web dashboard** (WebSocket) over **HTTPS**
- **Security**: JWT auth, RBAC (admin/analyst/viewer), **MFA TOTP**,
  CSP/HSTS headers, encrypted secrets (Fernet)
- **Reliable persistence** (SQLite + in-memory cache), **automatic
  encrypted backups**
- **NIS2 support**: Article 23 incident reports generated for manual
  submission to the CSIRT, **verified** tamper-evident audit hash-chain,
  retention policy
- **Reports** in STIX 2.1 format for submission to the national CSIRT

---

## Tech stack

**Backend:** Python 3.12 · FastAPI · asyncio · SQLite (aiosqlite) · ZeroMQ
**Frontend:** HTMX · Alpine.js · Tailwind CSS · WebSocket
**AI/ML:** ONNX Runtime · scikit-learn · NumPy · MITRE ATT&CK mapping
**Security:** JWT (python-jose) · bcrypt · pyotp (TOTP) · cryptography (Fernet)
**Infra:** Alpine Linux · OpenRC · Docker · nftables · fail2ban
**Quality:** pytest (security regression suite) · CI (GitHub Actions)

---

## Engineering & technical decisions

Some of the challenges I solved in this project:

- **Resilient event pipeline** — fan-out of every event to 5 consumers
  (memory, database, alerting, AI, compliance) without losing forensic
  events, with timeouts and failure logging.
- **Real tamper-evidence** — implemented periodic verification of the audit
  log's Merkle hash chain; detects database tampering (a NIS2 requirement
  that many solutions only simulate).
- **Security hardening** — an internal audit that found and fixed stored
  XSS, open CORS, missing security headers and default secrets; all covered
  by regression tests.
- **Signed over-the-air updates** — RSA-signed release manifests verified on
  the appliance, with SHA-256 payload pinning and automatic rollback on a
  failed health check.
- **One-command installer** for Alpine (from manual SSH to a single
  reproducible command) and packaging for a physical appliance or VM.

---

## Deployment models

| Tier | Format | Use case |
|------|--------|----------|
| **Pro** | Pre-configured physical appliance (mini-PC) | SME wanting turn-key |
| **Standard** | ISO/OVA for the customer's VM | Those who already virtualise |
| **Cloud** | Managed instance | Internet-facing honeypot |

---

## My role

Platform development at Expertmode: backend architecture, event pipeline,
AI agent integration, dashboard, NIS2 compliance, security hardening,
automated tests, and packaging/deployment.

> 📬 Additional technical detail or a demo available on request.
> Source code is private (property of Expertmode).
