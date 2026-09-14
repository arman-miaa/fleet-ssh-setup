# 12. Tailscale vs Cloudflare Tunnel

This note compares the two main remote access patterns used in modern setups.

> Public-safe note: real IPs, domains, tunnel tokens, hostnames, and secrets should never be committed to a public repo.

## 1. Short answer

- Tailscale: private network, peer-to-peer mesh, good for SSH and internal access
- Cloudflare Tunnel: external/public service routing through Cloudflare, good for web app exposure

## 2. Core difference

### Tailscale / Headscale

```text
Device A <-> Device B <-> Device C
```

This is a private network that acts like a secure mesh.

### Cloudflare Tunnel

```text
Local service -> Cloudflare Tunnel -> Cloudflare Edge -> public access
```

This is a tunnel for exposing services without direct public port exposure.

## 3. Use each one for different goals

Use Tailscale when:

- you need SSH to a laptop or server
- you want private peer access
- you want all devices on one private network

Use Cloudflare Tunnel when:

- you want a web app reachable from the internet
- you want to avoid direct port forwarding
- you want Cloudflare-managed edge routing

## 4. Example comparison table

```text
Scenario                      | Recommended choice
---------------------------------------------------------
SSH to laptop over private mesh | Tailscale / Headscale
Expose web app publicly        | Cloudflare Tunnel
Private device-to-device access | Tailscale / Headscale
Avoid direct port exposure     | Cloudflare Tunnel
```

## 5. They can work together

A modern setup may include both:

- Tailscale for private SSH and infrastructure access
- Cloudflare Tunnel for public-facing apps

## 6. Example placeholders

```text
TAILSCALE_HEADSCALE_URL=https://<your-headscale-server-url>
SSH_HOST=100.64.10.12
CLOUDFLARE_HOSTNAME=app.example.com
```

## 7. Final note

If your goal is secure remote command access to machines, Tailscale/Headscale is the better fit. If your goal is public web access without exposing ports, Cloudflare Tunnel is the better fit.
