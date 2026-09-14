# 08. Recovery & Rollback Guide

এই note-এ SSH/Tailscale failure হলে কী করবেন—এ topic covered থাকবে।

> নোট: public repo-তে real server URL, real IP, auth key বা production config লিখবেন না।

## ১. SSH fails but host is online

Check in order:

1. SSH service running?
2. port 22 listening?
3. firewall allows peer IP?
4. Tailscale mesh is connected?
5. user and key are valid?

Windows:

```powershell
Get-Service sshd
Get-NetTCPConnection -State Listen -LocalPort 22
Get-NetFirewallRule -All
```

Linux:

```bash
sudo systemctl status ssh
sudo ss -tulpn | grep 22
sudo ufw status
```

## ২. Tailscale disconnects

Check:

```bash
tailscale status
tailscale ip -4
```

Fix:

- reconnect with correct login server
- check auth key validity
- ensure server endpoint is reachable

## ৩. Firewall blocks access

If a peer IP changed or rule was overwritten:

- update allow-list
- confirm target host IP matches Tailscale IP
- retry connection

## ৪. Password-based login is still enabled

Fix:

```text
PasswordAuthentication no
PubkeyAuthentication yes
```

Then restart SSH service.

## ৫. Recovery checklist

1. ⬜ service checked
2. ⬜ port checked
3. ⬜ Tailscale checked
4. ⬜ peer IP verified
5. ⬜ firewall list updated
6. ⬜ key auth checked

## ৬. Rollback plan

If a change breaks access:

- revert last SSH config change
- remove or edit the new firewall rule
- confirm Tailscale connectivity
- test with a known-good peer

## ৭. Final note

Recovery is faster when you follow a fixed order: service → network → firewall → auth → policy. That sequence prevents random guesswork.
