# Kylo-Khan Mobile Operations Kit

## Overview
Mobile tmux configuration for persistent AI legal research sessions on GCP VM infrastructure. Supports session recovery across disconnects and device switches.

## Infrastructure
| Resource | Value |
|---|---|
| **Host VM** | vm4-seed (e2-standard-4, us-central1-a) |
| **Project** | g-france-vm-project |
| **Cloud Backup** | gs://njw-agent-outbox/sessions/ |
| **AI Model** | Google Gemini 2.5 Flash |
| **Document DB** | Firestore `database-1` → `alj_documents_v2` (982 docs) |

## Remote Access

### Apache Guacamole (Docker)
Browser-based remote desktop/SSH gateway running on vm4-seed.

| Container | Image | Port | Role |
|---|---|---|---|
| `guacamole` | guacamole/guacamole | 8080 (web UI) | Web frontend |
| `guacd` | guacamole/guacd | 4822 | Connection proxy |
| `guac-mysql` | mysql:8.0 | 3306 | Auth/config database |

Access at: `http://<vm4-seed-ip>:8080/guacamole`

### RustDesk Server (Docker)
Self-hosted remote desktop relay running on vm4-seed.

| Container | Image | Ports | Role |
|---|---|---|---|
| `hbbs` | rustdesk/rustdesk-server | 21115-21116, 21118 | Rendezvous/ID server |
| `hbbr` | rustdesk/rustdesk-server | 21117, 21119 | Relay server |

### Tailscale Mesh Network (v1.94.1)
| Device | Tailscale IP | OS | Status |
|---|---|---|---|
| vm4-seed | 100.78.159.59 | linux | online |
| vm1-orch | 100.108.11.17 | linux | offline |
| iphone-14 | 100.99.79.97 | iOS | online |
| samsung-sm-a156u-1 | 100.114.193.120 | android | online |
| samsung-sm-a156u | 100.94.79.39 | android | online |
| win-jmjt4ep4eb5 | 100.95.228.49 | windows | online |

**Note:** vm1-orch (orchestrator) is currently offline.

## Tmux Session Recovery

### First Time Setup
1. SSH into vm4-seed
2. Open tmux: `tmux new -s kylo`
3. Install plugins: `Ctrl+b` then `Shift+I`
4. Wait for confirmation message

### Restoring a Session
1. SSH into vm4-seed
2. Attach to session: `tmux attach -t kylo`
3. If session was lost, restore layout: `Ctrl+b` then `Ctrl+r`

### Auto-Save
Sessions auto-save every 15 minutes via tmux-continuum. No manual action needed.

## Starting the AI Assistant

```bash
cd ~/kylo-agent && source venv/bin/activate
python3 kylo_legal.py
```

## Starting Telegram Services

```bash
cd ~/kylo-agent && source venv/bin/activate
nohup python3 kylo_telegram_bot.py > /tmp/kylo_bot.log 2>&1 &
nohup python3 kylo_telegram_worker.py > /tmp/kylo_worker.log 2>&1 &
```

## Installed Tmux Plugins
- **tpm** — Plugin manager
- **tmux-sensible** — Sensible defaults
- **tmux-resurrect** — Save/restore sessions across restarts
- **tmux-continuum** — Automatic session saving (15 min interval)

## Key Bindings
| Keys | Action |
|---|---|
| `Ctrl+b` `Shift+I` | Install/update plugins |
| `Ctrl+b` `Ctrl+s` | Manual session save |
| `Ctrl+b` `Ctrl+r` | Restore saved session |

## Cloud Backup Location
Session protocol files are stored at:
```
gs://njw-agent-outbox/sessions/YYYY-MM-DD_daily_brief.md
```

## Case Reference
STAA Whistleblower Case 2025-STA-00035 — Nicholas Wilkes v. Sysco Denver, Inc.
