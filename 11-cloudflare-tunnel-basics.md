# 11. Cloudflare Tunnel Basics

This note explains the idea of Cloudflare Tunnel and how it differs from a Tailscale/Headscale mesh network.

> Public-safe note: never commit real domains, tunnel hostnames, auth tokens, or secret config values.

## 1. What is Cloudflare Tunnel?

Cloudflare Tunnel creates a secure tunnel from a local service to Cloudflare's edge network.

It is useful when you want to expose a service without opening direct inbound ports on your home or office network.

## 2. Typical use cases

- expose a web app through a public hostname
- avoid direct public firewall exposure
- connect internal services through a tunnel
- keep external exposure limited and managed by Cloudflare

## 3. Core idea

Instead of opening port 80/443 directly to the internet, Cloudflare Tunnel keeps a secure outbound connection from your machine to Cloudflare.

Example concept:

```text
Local service -> Cloudflare Tunnel -> Cloudflare edge -> public access
```

## 4. Difference from Tailscale

### Tailscale / Headscale

- private network
- device-to-device connectivity
- mesh-based remote access
- often used for SSH, internal tools, and secure peer access

### Cloudflare Tunnel

- public-facing or externally routed access
- edge-based exposure
- useful for apps and web resources
- not the same as a private SSH mesh

## 5. When to use which

Use Tailscale/Headscale when:

- you want private peer access
- you want SSH over a mesh
- devices should be in one private network

Use Cloudflare Tunnel when:

- you want to expose an app or service externally
- you want a managed public access layer
- you want to avoid opening ports directly

## 6. Example placeholder values

```text
CLOUDFLARE_TUNNEL_NAME=my-app-tunnel
PUBLIC_HOSTNAME=app.example.com
LOCAL_SERVICE=http://localhost:3000
```

## 7. Safety concerns

- never commit real domain names
- never commit tunnel tokens or secret config
- prefer placeholders in public repos

## 8. Final note

Tailscale is for private mesh networking. Cloudflare Tunnel is for external service exposure. They are complementary, but not interchangeable.
