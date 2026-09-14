# 02. SSH Security Hardening Guide

এই note-এ SSH-কে secure করার উপায় আলোচনা করা হবে।

> নোট: public repo-তে real host, real IP, real user name, auth key বা production config কখনো লিখবেন না।

## ১. SSH কেন secure করতে হয়?

SSH access open রেখে দিলে:

- brute force attack হতে পারে
- password-based login risk বাড়ে
- unauthorized peer access সম্ভব

তাই SSH-কে harden করা দরকার।

## ২. Key-based authentication

Password login-এর বদলে SSH key ব্যবহার করা best practice।

Client-এ key generate:

```bash
ssh-keygen -t ed25519 -C "arman@client"
```

Server-এ public key copy:

```bash
ssh-copy-id arman@100.64.10.12
```

Windows PowerShell-এ:

```powershell
ssh-copy-id arman@100.64.10.12
```

## ৩. Disable password login

Server config file:

```text
C:\ProgramData\ssh\sshd_config
```

Linux-এ:

```text
/etc/ssh/sshd_config
```

Add or update:

```text
PasswordAuthentication no
PubkeyAuthentication yes
```

Then restart:

Windows:

```powershell
Restart-Service sshd
```

Linux:

```bash
sudo systemctl restart ssh
```

## ৪. Allow only specific users

```text
AllowUsers arman
```

This prevents random users from logging in.

## ৫. Disable root login

```text
PermitRootLogin no
```

## ৬. Example hardened config

```text
Port 22
Protocol 2
PasswordAuthentication no
PubkeyAuthentication yes
PermitRootLogin no
AllowUsers arman
X11Forwarding no
```

## ৭. Recommended approach

- use SSH key
- disable password auth
- restrict user list
- use Tailscale mesh IP allow-list
- monitor failed login attempts

## ৮. Example placeholders

```text
SSH_USER=arman
REMOTE_HOST=100.64.10.12
ALLOWED_USER=arman
```

## ৯. Checklist

1. ⬜ ssh key generated
2. ⬜ public key copied
3. ⬜ password auth disabled
4. ⬜ root login disabled
5. ⬜ trusted user allowed
6. ⬜ service restarted

## ১০. Final note

SSH security hardening ঠিক করতে পারলে remote access much safer হয়। Tailscale + SSH key + firewall restriction এই তিনটি একসাথে কাজ করলে best result পাওয়া যায়।
