# 10. Tailscale Basics

This note explains the fundamentals of Tailscale and how it fits into a private mesh network.

> Public-safe note: never store real domains, real IPs, auth keys, or production secrets in a public repo.

## 1. What is Tailscale?

Tailscale is a VPN-style mesh network that connects devices over encrypted tunnels.

It is useful for:

- secure private connectivity between machines
- remote access without exposing public IPs
- joining laptops, phones, and servers into one network

## 2. Why use Tailscale?

Tailscale helps you avoid exposing machines directly to the internet while still allowing access from trusted devices.

Typical benefits:

- no public IP required for each host
- easier peer-based access control
- private communication between devices
- works well with SSH and internal services

## 3. Tailscale identity model

Each device is given a node identity and usually a private mesh IP such as:

```text
100.64.x.x
```

Example placeholder values:

```text
100.64.10.2
100.64.10.5
100.64.10.12
```

## 4. How it works in this repo

A device joins the mesh using a Headscale or Tailscale coordination server and then gets a private IP.

Example flow:

```bash
tailscale up --login-server https://<your-headscale-server-url>
```

Then you can access the device by its private mesh IP:

```bash
ssh arman@100.64.10.12
```

## 5. Tailscale vs Headscale

- Tailscale: official client and product
- Headscale: self-hosted control server for Tailscale clients

In short:

- Tailscale provides the client and mesh networking
- Headscale provides the self-hosted coordination layer

## 6. Common usage in this repo

Tailscale is used to:

- connect a laptop to the private mesh
- allow a trusted peer to reach SSH
- make access private and controlled

## 7. Safety checklist

- use placeholders in public repos
- do not commit auth keys
- do not expose production server URLs
- restrict peer access with firewall rules

## 8. Quick example

```text
HEADSCALE_URL=https://<your-headscale-server-url>
MESH_IP=100.64.10.12
SSH_USER=arman
```

## 9. Final note

Tailscale is the networking layer. SSH is the remote access method. Firewall policy is the safety gate. Together they create a clean remote control setup.
