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

## Mosh (Mobile Shell)

Mosh is like SSH but built for phones. It handles disconnects, network switches (WiFi to cellular), and lag automatically. Your session stays alive even when your phone loses signal.

### Why Use Mosh

| | SSH | Mosh |
|---|---|---|
| WiFi to cellular switch | Disconnects | Stays connected |
| Phone goes to sleep | Disconnects | Stays connected |
| High latency | Feels sluggish | Shows keystrokes instantly |
| Packet loss | Freezes | Handles it gracefully |

### Enable Mosh in Termius

1. Open Termius
2. Tap on your host (vm4-seed or vm1-orch)
3. Tap **Edit** (pencil icon)
4. Find the **Mosh** toggle — turn it **on**
5. Save and connect

That's it. Termius handles the rest. Mosh is installed on both VMs.

### Mosh from a Terminal

```bash
# Instead of: ssh mobile_admin@100.78.159.59
mosh mobile_admin@100.78.159.59

# Or for vm1-orch:
mosh mobile_admin@100.108.11.17
```

### Mosh Server Versions

| VM | Mosh Version | UDP Ports |
|---|---|---|
| vm4-seed | 1.4.0 | 60000-61000 (no firewall) |
| vm1-orch | 1.3.2 | 60000-61000 (UFW allowed on tailscale0) |

## Quick Reference

| VM | IP | DNS Name | User |
|---|---|---|---|
| vm1-orch | 100.108.11.17 | vm1-orch.tail5ceaf1.ts.net | mobile_admin |
| vm4-seed | 100.78.159.59 | vm4-seed.tail5ceaf1.ts.net | mobile_admin |
