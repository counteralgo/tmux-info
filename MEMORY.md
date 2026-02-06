# Memory - Kylo-Khan Project

## GitHub
- **Account:** counteralgo
- **Repo:** [counteralgo/tmux-info](https://github.com/counteralgo/tmux-info) — Mobile tmux config, protocols, audit reports
- Git identity (local): `counteralgo` / `counteralgo@users.noreply.github.com`

## GCS Buckets
- `gs://njw-agent-outbox/` — Cloud backup for session files and audit reports
- `gs://njw-agent-inbox/` — Empty (unused so far)
- `gs://g-france-vm-project-input/` — Document ingestion
- `gs://g-france-vm-project-output/` — OCR results
- `gs://g-france-vm-project-terraform-files/` — Terraform state
- `gs://g-france-project-docs-20260205/` — Project docs

## Tmux Setup (vm4-seed)
- TPM installed at `~/.tmux/plugins/tpm`
- Config at `~/.tmux.conf` — mouse on, resurrect, continuum (15 min auto-save)
- Plugins need `Ctrl+b Shift+I` on first tmux launch to install

## Remote Access (vm4-seed)
- **Guacamole** — Docker (guacamole + guacd + guac-mysql), web UI on port 8080
- **RustDesk** — Docker (hbbs + hbbr), self-hosted relay on ports 21115-21119
- **Tailscale** v1.94.1 — mesh network, 6 devices:
  - vm4-seed (100.78.159.59), vm1-orch (100.108.11.17, often offline)
  - iphone-14, 2x samsung, win-jmjt4ep4eb5

## Repo Contents (tmux-info)
- `README.md` — Mobile ops kit documentation
- `daily_brief.md` — Session restore protocol
- `tmux.conf` — Tmux config backup
- `technical_audit_report.md` — Audit report (markdown)
- `technical_audit_report.pdf` — Audit report (PDF)

## Cloud Backup Pattern
- Session files go to `gs://njw-agent-outbox/sessions/YYYY-MM-DD_filename`
- Audit reports live at root of `gs://njw-agent-outbox/`

## Lessons
- vm4-seed has no global git config — set user.name/email per repo
- tmux 3.3a is installed on vm4-seed
