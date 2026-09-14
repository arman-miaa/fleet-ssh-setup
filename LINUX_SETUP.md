# Linux SSH Setup Guide

এই ফাইলটি Linux machine-এ SSH + Tailscale + firewall setup-এর একটি solid, production-friendly guide।

> নোট: public repo-তে real IP, real domain, auth key, বা production access details কখনো লিখবেন না। এখানে সবকিছু dummy/placeholder-ভিত্তিক।

## ১. Overview

Linux-এ SSH server setup-এর logic Windows version-এর মতোই, কিন্তু commands এবং service management আলাদা।

Typical setup:

- install OpenSSH server
- enable sshd service
- join Tailscale network
- allow trusted peers only
- use SSH keys instead of password login

## ২. Install OpenSSH server

Ubuntu/Debian:

```bash
sudo apt update
sudo apt install openssh-server -y
```

CentOS/RHEL:

```bash
sudo dnf install openssh-server -y
```

Arch:

```bash
sudo pacman -S openssh
```

## ৩. Enable and start SSH service

```bash
sudo systemctl enable --now ssh
sudo systemctl status ssh
```

Check port listening:

```bash
sudo ss -tulpn | grep 22
```

## ৪. Tailscale / Headscale setup

Install Tailscale:

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

Connect to Headscale:

```bash
sudo tailscale up --login-server https://<your-headscale-server-url>
```

If using auth key:

```bash
sudo tailscale up --login-server https://<your-headscale-server-url> --authkey <your-auth-key>
```

Check your mesh IP:

```bash
tailscale ip -4
```

Example dummy values:

```text
100.64.10.30
100.64.10.31
```

## ৫. Firewall rule

UFW example:

```bash
sudo ufw allow 22/tcp
sudo ufw allow from 100.64.10.2 to any port 22
sudo ufw allow from 100.64.10.5 to any port 22
sudo ufw status
```

If you want to restrict more tightly:

```bash
sudo ufw deny 22/tcp
sudo ufw allow from 100.64.10.2 to any port 22
sudo ufw allow from 100.64.10.5 to any port 22
```

## ৬. SSH hardening

Edit config:

```bash
sudo nano /etc/ssh/sshd_config
```

Recommended settings:

```text
Port 22
PasswordAuthentication no
PubkeyAuthentication yes
PermitRootLogin no
AllowUsers arman
```

Reload SSH:

```bash
sudo systemctl restart ssh
```

## ৭. Key-based auth

Create SSH key pair on client:

```bash
ssh-keygen -t ed25519 -C "arman@client"
```

Copy public key to remote Linux host:

```bash
ssh-copy-id arman@100.64.10.30
```

## ৮. Remote access test

From trusted peer machine:

```bash
ssh arman@100.64.10.30
whoami
hostname
uname -a
```

## ৯. Example placeholders

```text
HEADSCALE_URL=https://<your-headscale-server-url>
SSH_USER=arman
REMOTE_LINUX_HOST=100.64.10.30
TRUSTED_PEER_1=100.64.10.2
TRUSTED_PEER_2=100.64.10.5
```

## ১০. Quick checklist

1. ⬜ OpenSSH server installed
2. ⬜ SSH service enabled
3. ⬜ Tailscale joined Headscale
4. ⬜ mesh IP captured
5. ⬜ firewall allow-list updated
6. ⬜ key-based authentication configured
7. ⬜ password login disabled
8. ⬜ peer-to-peer SSH test passed

## ১১. Recommendation

Linux-এ setup করা সবচেয়ে stable, secure, and practical. Phone-এ SSH host setup feasible but less reliable. For public repo usage, always use placeholder values and avoid real network details.
