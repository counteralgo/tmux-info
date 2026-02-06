# System Context: G-France-VM-Project & Tailscale Mesh

**Project ID:** g-france-vm-project
**Network Admin:** njwilkes@gmail.com
**Tailnet:** tail5ceaf1.ts.net
**Primary Domain:** Litigation Support / AI Infrastructure
**Last Verified:** 2026-02-06

---

## 1. Network Topology (Tailscale Mesh v1.94.1)

Overview of secure overlay network nodes. Devices utilize MagicDNS.

| Hostname | Tailscale IP | OS | Role | Current State |
|---|---|---|---|---|
| vm4-seed | 100.78.159.59 | Linux (Debian 12) | Active AI Interface (Kylo-Khan). Primary dev/chat node. | Online |
| vm1-orch | 100.108.11.17 | Linux (Debian 12) | Orchestrator (Telo-Khan). Heavy compute/Legal Brain. | Offline (Intentional)* |
| win-jmjt4ep4eb5 | 100.95.228.49 | Windows | Admin Console. Primary workstation. | Online |
| iphone-14 | 100.99.79.97 | iOS | Mobile Command. User interface. | Online |
| samsung-sm-a156u | 100.94.79.39 | Android | Field Unit 1. Secondary interface. | Online |
| samsung-sm-a156u-1 | 100.114.193.120 | Android | Field Unit 2. Tertiary interface. | Online |

> **Note on VM1-ORCH:** This node is powered down when not in active use to save costs. Its "Offline" status in Tailscale is a known and valid state, not a network error.

---

## 2. Compute Infrastructure & AI Personas

### VM4-SEED (The Tactical Unit)
- **Infrastructure:** GCP e2-standard-4 (4 vCPU, 16GB RAM)
- **Agent Name:** Kylo-Khan
- **AI Model:** Gemini 2.5 Flash (Vertex AI)
- **Function:** Rapid response, legal research, coding assistance, document upload via Telegram
- **Services Running:**
  - Kylo-Khan CLI (`~/kylo-agent/kylo_legal.py`)
  - Telegram Bot (`kylo_telegram_bot.py`) — @g_patton_cli_bot
  - Telegram Worker (`kylo_telegram_worker.py`) — Pub/Sub consumer
  - Apache Guacamole (Docker, port 8080) — browser-based remote access
  - RustDesk Server (Docker, ports 21115-21119) — self-hosted remote desktop relay
- **Access:**
  - `ssh mobile_admin@100.78.159.59` (via Tailscale)
  - `gcloud compute ssh vm4-seed --project=g-france-vm-project --zone=us-central1-a`

### VM1-ORCH (The Strategist)
- **Infrastructure:** GCP e2-standard-4 (4 vCPU, 16GB RAM)
- **Agent Name:** Telo-Khan
- **AI Model:** Gemini (version TBD — verify when online)
- **Function:** Deep legal reasoning, document drafting, heavy batch processing
- **Deployment:** Intermittent. Powered on only for heavy batch processing or deep legal strategy sessions.
- **Access:**
  - `ssh mobile_admin@100.108.11.17` (via Tailscale, when online)
  - `gcloud compute ssh vm1-orch --project=g-france-vm-project --zone=us-central1-a`

### VM2-WORKER (The Processor)
- **Infrastructure:** GCP e2-medium (2 vCPU, 4GB RAM)
- **Function:** PDF OCR, Bates numbering, document ingestion
- **Network Status:** Non-Mesh. Exists in GCP but does not have a Tailscale node installed.
- **Action Item:** Requires public IP or internal GCP network hop to access until Tailscale is installed.

---

## 3. Remote Access Services (vm4-seed)

### Apache Guacamole (Docker)
| Container | Image | Port | Role |
|---|---|---|---|
| guacamole | guacamole/guacamole | 8080 (web UI) | Web frontend |
| guacd | guacamole/guacd | 4822 | Connection proxy |
| guac-mysql | mysql:8.0 | 3306 | Auth/config database |

Access at: `http://100.78.159.59:8080/guacamole`

### RustDesk Server (Docker)
| Container | Image | Ports | Role |
|---|---|---|---|
| hbbs | rustdesk/rustdesk-server | 21115-21116, 21118 | Rendezvous/ID server |
| hbbr | rustdesk/rustdesk-server | 21117, 21119 | Relay server |

---

## 4. Data Layer

### GCS Buckets
| Bucket | Purpose |
|---|---|
| `gs://g-france-vm-project-input/` | Document ingestion (30-day lifecycle, versioned) |
| `gs://g-france-vm-project-output/` | OCR results (7-year lifecycle, versioned) |
| `gs://g-france-vm-project-terraform-files/` | Terraform state |
| `gs://g-france-project-docs-20260205/` | Project docs |
| `gs://njw-agent-outbox/` | Cloud backup — sessions, audit reports, uploads |
| `gs://njw-agent-inbox/` | Empty (unused) |

### Cloud Backup Pattern
- Session files: `gs://njw-agent-outbox/sessions/YYYY-MM-DD_filename`
- Mobile uploads: `gs://njw-agent-outbox/uploads/YYYY-MM-DD_filename`
- Audit reports: `gs://njw-agent-outbox/` (root)

### Firestore
- **Database:** `database-1`
- **Primary Collection:** `alj_documents_v2` (982 documents with AI summaries)
- **Other Collections:** `documents`, `tasks`, `auditlog`, `systemstate`, `oauthtokens`

### Pub/Sub
| Topic | Subscription | Purpose |
|---|---|---|
| doc-ingestion | vm2-doc-processor | PDF document processing |
| photo-ingestion | vm3-photo-processor | Photo processing (inactive — no VM3) |
| ocr-requests | ocr-worker | OCR processing |
| dead-letter | dead-letter-monitor | Failed message handling |
| kylo-telegram-requests | kylo-telegram-worker | Telegram bot message queue |

---

## 5. Operational Protocols for AI Assistants

### Connectivity Rules
- **Prioritize Tailscale:** Always use `100.x.x.x` IPs or MagicDNS hostnames for management to ensure encryption.
- **VM1 State Check:** Before attempting to SSH or deploy code to vm1-orch, the Assistant must ask the user: "Is VM1 currently powered on?"
- **VM2 Access:** Access to vm2-worker must currently be routed through GCP Console or Public IP, as it is outside the Tailscale mesh.

### Software Stack
- **OS:** Debian 12 (Bookworm) on all Linux VMs
- **Python:** 3.11.2 with venv
- **AI SDK:** google-genai (Vertex AI)
- **Telegram:** python-telegram-bot v22.6 (async)
- **Messaging:** Pub/Sub message queue pattern
- **Git:** GitHub account `counteralgo`, no global git config — set per-repo

### Known Issues
- **vm3-worker** (photo processor) is offline — photo processing disabled
- **Orphaned Pub/Sub subscription:** `patton-uplink` (topic deleted)
- **vm1-orch** requires OAuth setup before Drive monitoring works
- **vm2-worker** missing from Tailscale mesh
- **Bot latency:** If @g_patton_cli_bot is unresponsive, check that the bot process is running on vm4-seed
- **Telegram file uploads:** Limited to 20MB per file (Telegram API limit)

---

## 6. GitHub Repositories
| Repo | Purpose |
|---|---|
| [counteralgo/tmux-info](https://github.com/counteralgo/tmux-info) | Mobile tmux config, protocols, audit reports, infrastructure docs |

---

## 7. Case Reference
**STAA Whistleblower Case 2025-STA-00035** — Nicholas Wilkes v. Sysco Denver, Inc.
- 982 documents in Firestore with AI summaries
- Bates numbered: wilkes-00001 through wilkes-02XXX
- Document types: motions, orders, notices, complaints, exhibits, grievances, appeals
