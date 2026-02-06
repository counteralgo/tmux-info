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
