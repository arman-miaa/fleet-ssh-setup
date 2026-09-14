# 01. Headscale Setup Guide

এই note-এ Headscale কী, কেন দরকার, আর কীভাবে setup করতে হয়—এটাই ব্যাখ্যা করা হবে।

> নোট: public repo-তে real IP, production domain, auth key, secret বা live server URL কখনো দেবেন না। এখানে সবকিছু placeholder-ভিত্তিক।

## ১. Headscale কী?

Headscale হলো Tailscale-এর open-source self-hosted alternative।

এর মাধ্যমে আপনি:

- private mesh network তৈরি করতে পারেন
- device গুলোকে একসাথে connect করতে পারেন
- SSH, web, internal services access করতে পারেন
- public internet-এর বদলে private network ব্যবহার করতে পারেন

## ২. কেন Headscale দরকার?

যদি আপনি Windows laptop বা Linux machine থেকে রিমোট SSH access চান, তাহলে Headscale খুব দরকারি হতে পারে।

কারণ:

- public IP expose করতে হয় না
- firewall রুল কম জটিল থাকে
- trusted peers-কে সহজে allow করা যায়
- phone, laptop, server একসাথে একই private mesh-এ থাকতে পারে

## ৩. Basic architecture

```text
Client device  ---> Headscale server ---> Tailscale mesh
Laptop           ---> Headscale server ---> Tailscale mesh
Phone            ---> Headscale server ---> Tailscale mesh
Linux server     ---> Headscale server ---> Tailscale mesh
```

## ৪. Required components

- Headscale server running
- HTTPS-enabled URL or server endpoint
- client device connected via Tailscale
- auth key or login flow

## ৫. Example URL format

```text
https://headscale.example.com
https://hs.yourdomain.com
https://<your-headscale-server-url>
```

Public repo-তে এই ধরনের real value রাখবেন না। শুধু placeholder ব্যবহার করুন:

```text
HEADSCALE_URL=https://<your-headscale-server-url>
```

## ৬. Client connection example

```powershell
tailscale up --login-server https://<your-headscale-server-url>
```

Linux-এ:

```bash
sudo tailscale up --login-server https://<your-headscale-server-url>
```

Auth key দিয়ে:

```powershell
tailscale up --login-server https://<your-headscale-server-url> --authkey <your-auth-key>
```

## ৭. Verify the device joined the mesh

```powershell
tailscale ip -4
```

Linux-এ:

```bash
tailscale ip -4
```

Expected output style:

```text
100.64.x.x
```

## ৮. Example placeholder values

```text
HEADSCALE_URL=https://<your-headscale-server-url>
AUTH_KEY=<your-auth-key>
MESH_IP=100.64.10.12
```

## ৯. Security best practice

- production URL never push to public repo
- auth key never commit in git
- use environment variables or local secret files
- restrict who can join your Headscale network

## ১০. Checklist

1. ⬜ Headscale server installed
2. ⬜ HTTPS URL configured
3. ⬜ Tailscale client joined
4. ⬜ mesh IP assigned
5. ⬜ trusted peers added
6. ⬜ secrets not committed

## ১১. Final note

Headscale setup হলো SSH access chain-এর first step। এটা ঠিকভাবে না হলে পরের security, firewall, SSH access সবকিছু unstable হয়ে যায়।
