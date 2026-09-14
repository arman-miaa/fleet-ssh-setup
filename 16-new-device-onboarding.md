# 16. New Device Onboarding

This note explains the standard process for bringing a new device into the SSH + Tailscale environment.

> Public-safe note: never commit real IPs, domains, auth keys, or production network values.

## 1. Goal

When a new device joins, the goal is to make it:

- reachable over Tailscale
- safe for SSH access
- limited to trusted peers only
- monitored and documented

## 2. Standard onboarding flow

### Step 1: install networking tools

- install Tailscale
- join the Headscale network
- confirm the mesh IP is assigned

Example:

```powershell
tailscale up --login-server https://<your-headscale-server-url>
```

```bash
sudo tailscale up --login-server https://<your-headscale-server-url>
```

### Step 2: install SSH service

Windows:

```powershell
$ssh = Get-WindowsCapability -Online -Name "OpenSSH.Server~~~~0.0.1.0"
if ($ssh.State -ne "Installed") {
    Add-WindowsCapability -Online -Name $ssh.Name
}
Set-Service sshd -StartupType Automatic
Start-Service sshd
```

Linux:

```bash
sudo apt install openssh-server -y
sudo systemctl enable --now ssh
```

### Step 3: allow trusted peers only

Use firewall rules or UFW to allow only approved mesh peers.

Example placeholders:

```text
ALLOWED_PEER_1=100.64.10.2
ALLOWED_PEER_2=100.64.10.5
```

### Step 4: create or verify user account

Example:

```text
SSH_USER=arman
```

### Step 5: enable key-based auth

- generate SSH key pair
- copy public key to the new host
- disable password auth

### Step 6: verify connectivity

From trusted device:

```bash
ssh arman@100.64.10.12
whoami
hostname
```

## 3. Onboarding checklist

1. ⬜ Tailscale joined
2. ⬜ mesh IP assigned
3. ⬜ SSH service enabled
4. ⬜ firewall allow-list updated
5. ⬜ key auth configured
6. ⬜ login test passed
7. ⬜ config recorded in device inventory

## 4. Example placeholder values

```text
HEADSCALE_URL=https://<your-headscale-server-url>
NEW_DEVICE_NAME=work-laptop
NEW_DEVICE_IP=100.64.10.12
SSH_USER=arman
```

## 5. Final note

A new device should only be considered ready after it is joined, secured, allowed, verified, and documented.
