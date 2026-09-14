# 04. Troubleshooting Guide

এই note-এ SSH/Tailscale setup-এ সাধারণ সমস্যা ও fix step দেওয়া হবে।

> নোট: public repo-তে real IP, domain, auth key বা production access detail কখনো রাখবেন না।

## ১. SSH connection refused

Check:

```powershell
Get-Service sshd
Get-NetTCPConnection -State Listen -LocalPort 22
```

Linux:

```bash
sudo systemctl status ssh
sudo ss -tulpn | grep 22
```

Fix:

- service start করুন
- port 22 listen আছে কিনা দেখুন
- firewall rule verify করুন

## ২. Tailscale peer cannot reach host

Check:

```bash
tailscale status
tailscale ip -4
```

Fix:

- node online আছে কিনা
- mesh IP assigned আছে কিনা
- firewall allow-list update হয়েছে কিনা

## ৩. Authentication failed

Check:

- user exists কিনা
- SSH public key copied আছে কিনা
- `PasswordAuthentication no` set আছে কিনা

Windows:

```powershell
whoami
Get-ChildItem C:\Users\Arman\.ssh
```

Linux:

```bash
ls -la ~/.ssh
cat ~/.ssh/authorized_keys
```

## ৪. Laptop sleeps unexpectedly

Check:

Windows:

```powershell
powercfg /query
```

Fix:

```powershell
powercfg /change standby-timeout-ac 0
powercfg /change hibernate-timeout-ac 0
```

## ৫. Firewall blocks SSH

Windows:

```powershell
Get-NetFirewallRule -All
```

Linux:

```bash
sudo ufw status verbose
```

Fix:

- allowed peer IP set করুন
- correct port 22 allow করুন
- before/after test করুন

## ৬. Tailscale IP changed

If peer IP changed, firewall rule may fail.

Fix:

- update allow-list
- re-check `tailscale ip -4`
- test again from allowed peer

## ৭. Quick diagnosis checklist

1. ⬜ SSH service running?
2. ⬜ port 22 listening?
3. ⬜ Tailscale status okay?
4. ⬜ peer IP correct?
5. ⬜ key auth configured?
6. ⬜ firewall allow-list updated?

## ৮. Final note

Remote access problems usually come from 3 places: service status, Tailscale connectivity, or firewall policy. Check those in order.
