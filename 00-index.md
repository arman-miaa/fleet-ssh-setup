# 00. Repo Index

This repository is organized as a step-by-step learning path for secure remote access using Tailscale/Headscale + SSH.

> Public-safe note: all real hostnames, IPs, URLs, auth keys, and secrets are intentionally replaced with placeholders.

## Learning order

1. [README.md](README.md) — overview and main setup flow
2. [PHONE_SETUP.md](PHONE_SETUP.md) — phone-based access scenarios
3. [LINUX_SETUP.md](LINUX_SETUP.md) — Linux SSH setup guide
4. [01-headscale-setup.md](01-headscale-setup.md) — Headscale basics and setup
5. [02-ssh-security-hardening.md](02-ssh-security-hardening.md) — secure SSH configuration
6. [03-firewall-rules.md](03-firewall-rules.md) — allow-list and firewall practice
7. [04-troubleshooting.md](04-troubleshooting.md) — common failure modes and fixes
8. [05-command-cheat-sheet.md](05-command-cheat-sheet.md) — repeated commands and quick checks
9. [06-device-inventory.md](06-device-inventory.md) — host and networking inventory
10. [07-access-policy.md](07-access-policy.md) — access policy and risk controls
11. [08-recovery-and-rollback.md](08-recovery-and-rollback.md) — restore path when access breaks
12. [09-ssh-client-quickstart.md](09-ssh-client-quickstart.md) — quick client connection guide
13. [10-tailscale-basics.md](10-tailscale-basics.md) — private mesh networking basics
14. [11-cloudflare-tunnel-basics.md](11-cloudflare-tunnel-basics.md) — tunnel overview and comparison
15. [12-tailscale-vs-cloudflare.md](12-tailscale-vs-cloudflare.md) — Tailscale vs Cloudflare decision guide
16. [13-monitoring-and-logs.md](13-monitoring-and-logs.md) — health checks and operational visibility
17. [14-ssh-tunneling.md](14-ssh-tunneling.md) — SSH port forwarding and jump host patterns
18. [15-backup-and-maintenance.md](15-backup-and-maintenance.md) — routine upkeep and recovery readiness16. [16-new-device-onboarding.md](16-new-device-onboarding.md) — standard process for joining a new device
17. [17-security-audit-checklist.md](17-security-audit-checklist.md) — repeatable review checklist
18. [18-remote-access-playbook.md](18-remote-access-playbook.md) — recovery process when SSH stops working
## Recommended reading flow

Start with the overview in [README.md](README.md), then continue in order from [01-headscale-setup.md](01-headscale-setup.md) onward.

## Core idea

The working pattern is:

```text
Headscale network -> trusted peer -> SSH service -> secured remote command access
```

## Public repo safety

Never commit:

- real IP addresses
- real domains
- real auth keys
- secrets
- production server names

Use placeholders instead:

```text
HEADSCALE_URL=https://<your-headscale-server-url>
SSH_USER=arman
REMOTE_HOST=100.64.10.12
```

## Final note

This repo is meant to be a clean knowledge base and setup guide, not an operational secret store.
