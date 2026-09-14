# 14. SSH Tunneling

This note explains the idea of SSH tunneling and when it is useful.

> Public-safe note: do not store production hostnames, ports, or real tunnel config in a public repo.

## 1. What is SSH tunneling?

SSH tunneling lets you forward traffic through an encrypted SSH connection.

It can be used for:

- port forwarding
- secure local proxying
- reaching internal services through a trusted SSH host
- using a remote machine as a jump host

## 2. Common use cases

- connect to a service only available inside a private network
- reach a local server through a trusted host
- redirect application traffic through a secure SSH tunnel

## 3. Simple example

Forward a remote port to your local machine:

```bash
ssh -L 8080:internal-service:80 arman@100.64.10.12
```

This means:

- local port 8080 is forwarded
- traffic goes through the SSH host
- remote service remains hidden behind the mesh

## 4. Reverse tunnel example

```bash
ssh -R 8081:localhost:3000 arman@100.64.10.12
```

This allows a remote side to reach a service on the source machine.

## 5. Important caution

SSH tunneling is powerful, but it must be used carefully:

- restrict to trusted peers
- use key-based access only
- avoid exposing tunnels unnecessarily

## 6. Example placeholders

```text
SSH_USER=arman
JUMP_HOST=100.64.10.12
FORWARDED_LOCAL_PORT=8080
REMOTE_SERVICE_PORT=80
```

## 7. Final note

SSH tunneling is a useful tool for secure access when direct access is not possible. It is powerful but should stay within a controlled access policy.
