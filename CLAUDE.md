# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Kylo-Khan AI Legal Assistant** - An AI-powered legal research assistant for STAA whistleblower case 2025-STA-00035 (Nicholas Wilkes v. Sysco Denver, Inc.). Uses Google Gemini 2.5 Flash for document analysis and natural language Q&A over 982 legal documents stored in Firestore.

## Repository Structure

```
/home/mobile_admin/aia-base/
├── PROGRESS.md                      # Detailed project progress and feature documentation
├── gcloud-infra-overview/           # Infrastructure-as-code
│   ├── gcloud-infra-overview.md     # Infrastructure summary
│   └── terraform/                   # 107 Terraform resource files
│       └── resources/               # Organized by resource type (Compute, Storage, IAM, etc.)
```

## Architecture

### Compute Infrastructure (GCP: g-france-vm-project, us-central1-a)
| VM | Type | Role |
|---|---|---|
| vm1-orch | e2-standard-4 | Orchestrator + web dashboard |
| vm2-worker | e2-medium | PDF processing + OCR |
| vm4-seed | e2-standard-4 | Development + Kylo-Khan agent host |

### Remote Access (vm4-seed)
| Service | Type | Ports | Notes |
|---|---|---|---|
| Guacamole | Docker (guacamole + guacd + guac-mysql) | 8080 (web UI) | Browser-based remote desktop/SSH |
| RustDesk | Docker (hbbs + hbbr) | 21115-21119 | Self-hosted remote desktop relay |
| Tailscale | v1.94.1 mesh VPN | — | 6 devices: vm4-seed, vm1-orch, iphone-14, 2x samsung, win-jmjt4ep4eb5 |

### Data Layer
- **Firestore:** `database-1` → `alj_documents_v2` collection (982 documents with AI summaries)
- **Firestore Collections:** `documents`, `tasks`, `auditlog`, `systemstate`, `oauthtokens`
- **GCS Buckets:**
  - `g-france-vm-project-input` — Document ingestion
  - `g-france-vm-project-output` — OCR results
  - `g-france-vm-project-terraform-files` — Terraform state
  - `g-france-project-docs-20260205` — Project docs
  - `njw-agent-outbox` — Cloud backup (sessions, audit reports)
  - `njw-agent-inbox` — Empty (unused)
- **Pub/Sub Topics:** doc-ingestion, photo-ingestion, ocr-requests, dead-letter, kylo-telegram-requests

### Application Stack
- **AI Model:** Google Gemini 2.5 Flash (Vertex AI)
- **CLI:** `kylo_legal.py` on vm4-seed
- **Telegram Bot:** @g_patton_cli_bot via Pub/Sub message queue

## Common Commands

### Running the Legal Assistant (on vm4-seed)
```bash
cd ~/kylo-agent && source venv/bin/activate
python3 kylo_legal.py
```

### Telegram Services
```bash
# Start
cd ~/kylo-agent && source venv/bin/activate
nohup python3 kylo_telegram_bot.py > /tmp/kylo_bot.log 2>&1 &
nohup python3 kylo_telegram_worker.py > /tmp/kylo_worker.log 2>&1 &

# Stop
pkill -f kylo_telegram

# Logs
tail -f /tmp/kylo_bot.log /tmp/kylo_worker.log
```

### SSH to VMs
```bash
gcloud compute ssh vm1-orch --project=g-france-vm-project --zone=us-central1-a
gcloud compute ssh vm2-worker --project=g-france-vm-project --zone=us-central1-a
gcloud compute ssh vm4-seed --project=g-france-vm-project --zone=us-central1-a
```

### Terraform
```bash
cd /home/mobile_admin/aia-base/gcloud-infra-overview/terraform
terraform init
terraform plan
```

## Key Application Files (on vm4-seed: ~/kylo-agent/)

| File | Purpose |
|---|---|
| `kylo_legal.py` | Main legal assistant - document search, AI Q&A, Bates lookup |
| `kylo_telegram_bot.py` | Telegram message receiver, publishes to Pub/Sub |
| `kylo_telegram_worker.py` | Pub/Sub subscriber, processes queries, sends responses |
| `IDENTITY.md` | Agent persona definition |

## CLI Commands (kylo_legal.py)

| Command | Description |
|---|---|
| `bates <num>` | Lookup by Bates number (e.g., `bates wilkes00055`) |
| `bates <start>-<end>` | Bates range (e.g., `bates wilkes00001-00050`) |
| `date <YYYY-MM>` | Documents from month |
| `read <bates>` | Full document text from GCS |
| `party <name>` | Filter by party (wilkes, sysco, osha, dol, alj) |
| `timeline` | Chronological case events |
| `summary` | AI-generated case summary |
| `export <filename>` | Save results to ~/kylo-agent/exports/ |
| `clear` | Clear conversation memory |
| `stats` | Database statistics |
| (any text) | Natural language Q&A |

## GCP Configuration

| Setting | Value |
|---|---|
| Project | g-france-vm-project |
| Region | us-central1 |
| Firestore DB | database-1 |
| Collection | alj_documents_v2 |
| AI Model | gemini-2.5-flash |
| Pub/Sub Topic | kylo-telegram-requests |

## Cloud Backup

Session files and configs are backed up to GCS:
```bash
# Upload to outbox
gcloud storage cp <file> gs://njw-agent-outbox/sessions/$(date +%F)_<filename>

# List backups
gcloud storage ls -r gs://njw-agent-outbox/
```

Configs are also version-controlled in GitHub: [counteralgo/tmux-info](https://github.com/counteralgo/tmux-info)

## Tmux (vm4-seed)

- TPM at `~/.tmux/plugins/tpm`, config at `~/.tmux.conf`
- Plugins: resurrect, continuum (15 min auto-save), sensible
- First launch: `Ctrl+b Shift+I` to install plugins
- Restore session: `Ctrl+b Ctrl+r`

## Git Identity

No global git config on vm4-seed. Set per-repo:
```bash
git config user.name "counteralgo"
git config user.email "counteralgo@users.noreply.github.com"
```

## Known Issues

- vm3-worker (photo processor) is offline - photo processing disabled
- Orphaned Pub/Sub subscription: `patton-uplink` (topic deleted)
- Orchestrator (vm1-orch) requires OAuth setup before Drive monitoring works
- vm1-orch frequently offline on Tailscale
