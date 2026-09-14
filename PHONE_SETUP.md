# Phone SSH Setup Guide

এই ফাইলটি দুইটি আলাদা scenario-cover করে:

1. Phone থেকে PC/Laptop control করা
2. Phone-to-phone access only

> নোট: public repo-তে real IP, real domain, auth key, product network detail কখনো লিখবেন না। এখানে সবকিছু dummy/placeholder-ভিত্তিক।

---

## A. Phone থেকে PC/Laptop control করা

এটা মূলত এমন setup যেখানে ফোন থেকে Windows/Linux laptop-এ SSH করে command run করা হয়।

### ১. Scenario

ফোন থেকে:

```bash
ssh arman@100.64.10.12
```

তারপর laptop-এ:

```powershell
Get-ComputerInfo
whoami
hostname
Get-Process
```

এভাবে phone থেকে laptop control করা যায়, command execute করা যায়, file check করা যায়, service monitor করা যায়।

### ২. Recommended flow

- Tailscale app install করুন
- Headscale URL add করুন
- Laptop-এ SSH server run করুন
- Laptop-এ firewall allow-list set করুন
- Phone-এ SSH client install করুন
- Phone থেকে SSH connect করুন

### ৩. Example commands

From phone terminal app:

```bash
ssh arman@100.64.10.12
```

Windows laptop-এ remote check:

```powershell
whoami
Get-Service
Get-Process
Restart-Computer -WhatIf
```

Linux laptop-এ remote check:

```bash
whoami
hostname
uname -a
systemctl status ssh
```

### ৪. Safe placeholders

```text
HEADSCALE_URL=https://<your-headscale-server-url>
REMOTE_USER=arman
REMOTE_HOST=100.64.10.12
TRUSTED_PEER_1=100.64.10.2
TRUSTED_PEER_2=100.64.10.5
```

### ৫. Important note

Phone থেকে laptop control করা practical, কিন্তু laptop/server-ই বেশি stable। Phone-এর battery, sleep mode, app lifecycle অনেক issue তৈরি করতে পারে।

---

## B. Only Phone-to-Phone access

এটা আলাদা scenario: শুধু ফোন থেকে ফোনে access, বা ফোনের মধ্যে direct SSH/Tailscale access।

### ১. Scenario

Android phone A থেকে Android phone B-তে SSH/remote access করতে চাইলে:

- dual phone environment
- Tailscale mesh on both
- SSH server installed on target phone
- key-based auth enabled

Example:

```bash
ssh arman@100.64.10.20 -p 8022
```

### ২. Realistic setup

#### Android phone as host

Termux example:

```bash
pkg update && pkg install openssh
sshd
```

Check:

```bash
sshd -T
```

Target phone IP example:

```text
PHONE_HOST=100.64.10.20
SSH_PORT=8022
```

#### Accessing from another phone

```bash
ssh arman@100.64.10.20 -p 8022
```

### ৩. Limitations

Phone-to-phone access খুব সীমিত:

- battery drains quickly
- background daemons may stop
- app restrictions apply
- not ideal for long-running services
- iPhone this is even harder

### ৪. Safe placeholders

```text
HEADSCALE_URL=https://<your-headscale-server-url>
PHONE_A_USER=arman
PHONE_B_HOST=100.64.10.20
PHONE_B_PORT=8022
TRUSTED_PHONE_PEER=100.64.10.2
```

---

## C. Which one should you choose?

- Phone -> PC/Laptop control: recommended for real remote administration
- Phone -> Phone only: okay for simple access, but not ideal for stable long-term use

### Best option

For reliable management, use:

- PC/Laptop or Linux server as host
- Phone as client

---

## D. Quick checklist

### Phone -> PC/Laptop

1. ⬜ Tailscale installed on phone
2. ⬜ Headscale URL configured
3. ⬜ SSH server on laptop enabled
4. ⬜ firewall allow-list set
5. ⬜ phone SSH client connects successfully

### Phone -> Phone only

1. ⬜ both phones on Tailscale
2. ⬜ target phone SSH server active
3. ⬜ port set and trusted access allowed
4. ⬜ key-based authentication used
5. ⬜ access test passed

---

## E. Final note

Phone থেকে PC/Laptop control করার idea আছে, কিন্তু direct phone-to-phone access আলাদা topic। Public repo-তে real network details, real IP, auth key, বা production URL কখনো দিবেন না।
