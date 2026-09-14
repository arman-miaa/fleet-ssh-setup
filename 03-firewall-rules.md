# 03. Firewall Rules Guide

এই note-এ Windows এবং Linux-এ firewall rule কীভাবে set করতে হয়, তা আলোচনা করা হবে।

> নোট: public repo-তে real peer IP, real subnet বা production network detail রাখবেন না।

## ১. Firewall কেন দরকার?

SSH open করতে গেলে firewall rule set না করলে unwanted source থেকে access আসতে পারে।

So best practice:

- only trusted IPs allowed
- port 22 to trusted peers only
- debug logs review regularly

## ২. Windows Firewall example

```powershell
$peers = @("100.64.10.2", "100.64.10.5")

if (Get-NetFirewallRule -Name "Fleet-SSH" -ErrorAction SilentlyContinue) {
    Remove-NetFirewallRule -Name "Fleet-SSH"
}

New-NetFirewallRule -Name "Fleet-SSH" `
    -DisplayName "Fleet SSH" `
    -Direction Inbound `
    -Action Allow `
    -Protocol TCP `
    -LocalPort 22 `
    -RemoteAddress $peers `
    -Profile Any
```

## ৩. Linux UFW example

```bash
sudo ufw allow from 100.64.10.2 to any port 22
sudo ufw allow from 100.64.10.5 to any port 22
sudo ufw status
```

If you want restrictive approach:

```bash
sudo ufw deny 22/tcp
sudo ufw allow from 100.64.10.2 to any port 22
sudo ufw allow from 100.64.10.5 to any port 22
```

## ৪. Important rule

SSH only trusted peers-কে allow করুন।

```text
ALLOWED_PEERS=100.64.10.2, 100.64.10.5
```

## ৫. Check current rules

Windows:

```powershell
Get-NetFirewallRule -Name "Fleet-SSH"
Get-NetFirewallAddressFilter -PolicyStore ActiveStore
```

Linux:

```bash
sudo ufw status verbose
```

## ৬. Common mistakes

- all traffic to port 22 open রাখা
- wrong peer IP add করা
- Tailscale IP change হলে rule update না করা
- key-based auth না ব্যবহার করা

## ৭. Safe placeholders

```text
TRUSTED_PEER_1=100.64.10.2
TRUSTED_PEER_2=100.64.10.5
SSH_PORT=22
```

## ৮. Checklist

1. ⬜ trusted peer list ready
2. ⬜ firewall rule created
3. ⬜ only port 22 allowed
4. ⬜ test from allowed peer works
5. ⬜ disallowed peer blocked

## ৯. Final note

Firewall + Tailscale + SSH key = strong remote access model.

Always keep your allow-list tight and review it regularly.
