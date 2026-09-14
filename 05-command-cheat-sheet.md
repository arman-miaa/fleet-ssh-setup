# 05. Command Cheat Sheet

এই note-এ frequently used SSH, Tailscale, Windows/Linux commands একসাথে দেওয়া আছে।

> নোট: sample values are placeholders only, not real environment data.

## ১. Tailscale commands

Check status:

```bash
tailscale status
```

Show mesh IP:

```bash
tailscale ip -4
```

Join Headscale:

```bash
tailscale up --login-server https://<your-headscale-server-url>
```

## ২. SSH commands

Connect:

```bash
ssh arman@100.64.10.12
```

Connect with port:

```bash
ssh arman@100.64.10.20 -p 8022
```

Check whoami:

```bash
whoami
```

Check hostname:

```bash
hostname
```

## ৩. Windows commands

Check service:

```powershell
Get-Service sshd
```

Start service:

```powershell
Start-Service sshd
```

Firewall check:

```powershell
Get-NetFirewallRule -All
```

Power plan check:

```powershell
powercfg /query
```

## ৪. Linux commands

Check service:

```bash
sudo systemctl status ssh
```

Start service:

```bash
sudo systemctl enable --now ssh
```

Firewall status:

```bash
sudo ufw status
```

Port check:

```bash
sudo ss -tulpn | grep 22
```

## ৫. Example placeholder values

```text
SSH_USER=arman
REMOTE_HOST=100.64.10.12
ALLOWED_PEER=100.64.10.2
SSH_PORT=22
```

## ৬. Recommended minimal test flow

```bash
ssh arman@100.64.10.12
whoami
hostname
uname -a
```

Windows test:

```powershell
whoami
Get-Service
Get-Process
```

## ৭. Final note

These commands are your practical toolbox. Use them for verification, debugging, and routine remote administration.
