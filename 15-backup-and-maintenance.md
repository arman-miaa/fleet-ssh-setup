# 15. Backup & Maintenance

This note covers the routine operational tasks needed to keep remote infrastructure healthy.

> Public-safe note: keep real secrets, domains, and hostnames out of a public repo.

## 1. Why maintenance matters

A remote SSH setup is not a one-time install. It requires:
- periodic checks
- config review
- service restarts
- backup of key files
- update verification

## 2. Backup essentials

Keep backups of:
- SSH config files
- public/private key files
- firewall rule exports
- Tailscale config and notes
- network inventory list

Windows example:

```powershell
Copy-Item C:\ProgramData\ssh\sshd_config C:\Backups\sshd_config.bak
```

Linux example:

```bash
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
```

## 3. Maintenance routine

Recommended weekly or monthly steps:
- review SSH access
- verify Tailscale status
- review firewall allow-list
- verify public key validity
- test SSH connection from a trusted peer

## 4. Example checklist

1. ⬜ SSH service is healthy
2. ⬜ Tailscale is connected
3. ⬜ firewall is still restricted
4. ⬜ key-based auth still works
5. ⬜ backup copy exists
6. ⬜ access list is current

## 5. Example placeholders

```text
HOST_NAME=work-laptop
SSH_USER=arman
SSH_CONFIG_BACKUP=/backup/sshd_config
TUNNEL_OR_HOST=100.64.10.12
```

## 6. Final note

Good maintenance keeps a secure remote access setup stable. A small routine is far cheaper than a full recovery process.
