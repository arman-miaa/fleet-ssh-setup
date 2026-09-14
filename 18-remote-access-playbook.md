# 18. Remote Access Playbook

This note is the operations guide for restoring access when a remote SSH setup breaks.

> Public-safe note: never store real auth keys, production IPs, real domains, or secrets in a public repo.

## 1. When access breaks, follow this order

### Step 1: confirm the device is online

Check Tailscale status and IP assignment:

```bash
tailscale status
tailscale ip -4
```

Windows:

```powershell
tailscale status
tailscale ip -4
```

### Step 2: confirm SSH service is running

Windows:

```powershell
Get-Service sshd
Get-NetTCPConnection -State Listen -LocalPort 22
```

Linux:

```bash
sudo systemctl status ssh
sudo ss -tulpn | grep 22
```

### Step 3: verify firewall allow-list

- check allowed peer IPs
- check port 22
- check whether the peer IP changed

### Step 4: verify key-based auth

- confirm `authorized_keys` exists
- confirm password auth is disabled if desired
- confirm the user account exists

### Step 5: verify user and host details

```text
SSH_USER=arman
REMOTE_HOST=100.64.10.12
```

## 2. Recovery checklist

1. ⬜ Tailscale status okay
2. ⬜ device reachable on mesh
3. ⬜ SSH service active
4. ⬜ port 22 listening
5. ⬜ firewall allow-list updated
6. ⬜ user and keys valid
7. ⬜ test SSH connection again

## 3. Final note

A quick, repeatable recovery playbook saves time and reduces guesswork when remote access breaks.
