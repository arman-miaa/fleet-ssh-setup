# 17. Security Audit Checklist

This note is a practical review checklist for validating that a remote SSH setup is still secure.

> Public-safe note: do not include real domains, real IPs, secret keys, or production access values.

## 1. Access review

- Are only trusted peers allowed to connect?
- Is SSH restricted to a specific user?
- Is password authentication disabled?
- Is root login disabled?
- Is a key-based login policy enforced?

## 2. Network review

- Are Tailscale peers current?
- Did any mesh IP change recently?
- Is the firewall allow-list still correct?
- Is port 22 only allowed from approved peers?

## 3. Host review

- Is the SSH service running?
- Are logs being checked?
- Are there failed login attempts?
- Has the device been rebooted and retested?

## 4. Example checklist

```text
[ ] PasswordAuthentication no
[ ] PubkeyAuthentication yes
[ ] PermitRootLogin no
[ ] AllowUsers arman
[ ] Firewall allow-list reviewed
[ ] Tailscale status checked
[ ] SSH key uploaded and verified
[ ] Device inventory updated
```

## 5. Example placeholders

```text
SSH_USER=arman
ALLOWED_PEER_1=100.64.10.2
ALLOWED_PEER_2=100.64.10.5
```

## 6. Final note

A secure setup is not just installed once. It must be reviewed regularly to remain safe and reliable.
