# 09. SSH Client Quick Start

এই note-এ SSH client কীভাবে connect করে, তা দ্রুত ব্যাখ্যা করা হবে।

> নোট: public repo-তে real URLs, real IPs, auth keys, বা production setup কখনো রাখবেন না।

## ১. Client types

You can connect from:

- Windows PowerShell / Terminal
- Linux terminal
- macOS terminal
- Android SSH client app
- phone shell apps with SSH support

## ২. Basic connection format

```bash
ssh <user>@<host>
```

Example:

```bash
ssh arman@100.64.10.12
```

With custom port:

```bash
ssh arman@100.64.10.20 -p 8022
```

## ৩. Windows example

```powershell
ssh arman@100.64.10.12
whoami
hostname
Get-Process
```

## ৪. Linux example

```bash
ssh arman@100.64.10.12
whoami
hostname
uname -a
```

## ৫. macOS example

```bash
ssh arman@100.64.10.12
whoami
hostname
```

## ৬. SSH key usage

Generate key:

```bash
ssh-keygen -t ed25519 -C "arman@client"
```

Use key when connecting:

```bash
ssh -i ~/.ssh/id_ed25519 arman@100.64.10.12
```

## ৭. Check connection health

```bash
ssh -v arman@100.64.10.12
```

Verbose mode helps debug:

- wrong host
- auth failure
- firewall deny
- service not listening

## ৮. Safe placeholders

```text
SSH_USER=arman
REMOTE_HOST=100.64.10.12
SSH_PORT=22
SSH_KEY=~/.ssh/id_ed25519
```

## ৯. Quick troubleshooting

If connect fails, check:

- host is reachable via Tailscale
- port 22 is open
- SSH service is running
- firewall allows the peer
- key is generated and copied

## ১০. Minimal test sequence

```bash
ssh arman@100.64.10.12
whoami
hostname
uptime
```

## ১১. Final note

The client connection step is the final proof that the whole chain is working: Headscale joined → SSH service running → firewall allowed → key auth configured → client can connect.
