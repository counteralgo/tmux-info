# Termius Mobile Setup Guide

## Prerequisites
- **Tailscale app** installed and connected on your phone
- Tailscale SSH enabled on both VMs
- ACL policy saved at https://login.tailscale.com/admin/acls/file

## Host Configuration

### vm1-orch

| Field | Value |
|---|---|
| **Alias** | `vm1-orch` |
| **Hostname** | `100.108.11.17` |
| **Port** | `22` |
| **Username** | `mobile_admin` |
| **Password** | leave empty |
| **Key** | leave empty / None |

### vm4-seed

| Field | Value |
|---|---|
| **Alias** | `vm4-seed` |
| **Hostname** | `100.78.159.59` |
| **Port** | `22` |
| **Username** | `mobile_admin` |
| **Password** | leave empty |
| **Key** | leave empty / None |

## Steps

1. Open Termius
2. Tap **+** → **New Host**
3. Fill in the fields from the table above
4. Tap **Save**
5. Tap the host to connect
6. Accept the host key fingerprint if prompted
7. You should get a terminal prompt — no password needed

## If It Asks for a Password

Tailscale SSH handles auth automatically, so it shouldn't. But if Termius won't connect without one:

1. Edit the host
2. Look for an **Authentication** section
3. Pick **None** or **Agent** if available
4. If forced to enter something, type any random character — Tailscale ignores it

## Troubleshooting

| Problem | Fix |
|---|---|
| Connection refused | Open Tailscale app on phone, make sure it's connected |
| Connection timed out | VM might be down — check GCP console |
| Authentication failed | Make sure username is `mobile_admin`, not `root` |

## Quick Reference

| VM | IP | DNS Name | User |
|---|---|---|---|
| vm1-orch | 100.108.11.17 | vm1-orch.tail5ceaf1.ts.net | mobile_admin |
| vm4-seed | 100.78.159.59 | vm4-seed.tail5ceaf1.ts.net | mobile_admin |
