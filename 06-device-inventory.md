# 06. Device Inventory & Host Registry

এই note-এ সব device, IP, purpose, access level সংরক্ষণ করার পদ্ধতি দেওয়া হবে।

> নোট: public repo-তে real hostnames, real IPs, production details বা secrets কখনো লিখবেন না।

## ১. কেন inventory দরকার?

যখন অনেক device থাকে, তখন track করা কঠিন হয়:

- ক কোন laptop
- কোন device-এ SSH allowed
- কোন host মূল access point
- কোন peer এর IP change হয়েছে

Inventory helps maintain clarity.

## ২. Example host registry

```text
DEVICE_NAME=work-laptop
OS=windows
ROLE=remote-host
MESH_IP=100.64.10.12
USER=arman
STATUS=active
ACCESS=trusted peer only
```

```text
DEVICE_NAME=linux-server
OS=linux
ROLE=jump-host
MESH_IP=100.64.10.30
USER=arman
STATUS=active
ACCESS=trusted peer only
```

```text
DEVICE_NAME=android-phone
OS=android
ROLE=client
MESH_IP=100.64.10.20
USER=arman
STATUS=limited
ACCESS=client-only
```

## ৩. Recommended fields

- device name
- OS
- role
- mesh IP
- SSH user
- allowed peers
- status
- notes

## ৪. Example template

```text
NAME=
OS=
ROLE=
MESH_IP=
SSH_USER=
ALLOWED_PEERS=
STATUS=
NOTES=
```

## ৫. Best practices

- update IP when Tailscale changes
- keep allowed peer list current
- tag devices by role
- avoid secrets in repo

## ৬. Checklist

1. ⬜ all hosts listed
2. ⬜ mesh IP documented
3. ⬜ user documented
4. ⬜ purpose documented
5. ⬜ access level documented

## ৭. Final note

Inventory is the operational memory of your mesh network. It keeps your SSH access simple and maintainable.
