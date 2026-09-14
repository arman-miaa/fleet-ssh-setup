# 19. SSH Key Management

This note covers the safe lifecycle of SSH keys: generating them, copying them, rotating them, and removing stale access.

> Public-safe note: never commit real private keys, secret passphrases, or production host data.

## 1. Why key management matters

SSH works best with key-based authentication because it is safer and easier to control than password login. Over time, keys can be lost, copied to the wrong machine, or left behind on devices that are no longer trusted.

## 2. Generate a key pair

Windows PowerShell:

```powershell
ssh-keygen -t ed25519 -C "device-name@example"
```

Linux/macOS:

```bash
ssh-keygen -t ed25519 -C "device-name@example"
```

Typical output goes to:

```text
~/.ssh/id_ed25519
~/.ssh/id_ed25519.pub
```

On Windows:

```text
C:\Users\Arman\.ssh\id_ed25519
C:\Users\Arman\.ssh\id_ed25519.pub
```

## 3. Install the public key on the remote host

From the client machine:

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub arman@100.64.10.12
```

Windows PowerShell alternative:

```powershell
type $env:USERPROFILE\.ssh\id_ed25519.pub | ssh arman@100.64.10.12 "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
```

> Replace the example mesh IP with your trusted host IP. Keep the example public-safe and private details out of a public repo.

## 4. Check the SSH configuration

On the host:

```bash
ls -la ~/.ssh
cat ~/.ssh/authorized_keys
```

Make sure the key belongs to the intended user and the file permissions are correct.

## 5. Rotate keys regularly

When a laptop is replaced or a key is exposed, rotate it:

1. generate a new key pair
2. add the new public key to the server
3. remove the old public key from `authorized_keys`
4. test access with the new key
5. keep the old key only until validation is complete

Example:

```bash
ssh-keygen -t ed25519 -C "new-device@example"
```

Then remove the stale key from the server host:

```bash
nano ~/.ssh/authorized_keys
```

Delete the old key line and save.

## 6. Remove stale access from known hosts

If a remote host identity changes, clear the cached fingerprint:

```bash
ssh-keygen -R 100.64.10.12
```

On Windows, check the host key cache in the SSH config area as needed and remove outdated entries.

## 7. Recommended key hygiene

- use one key per device or purpose when practical
- keep private keys on the device that owns them
- do not copy private keys to shared folders
- protect keys with a strong passphrase if your workflow allows it
- review authorized keys regularly

## 8. Example placeholder values

```text
SSH_KEY_PATH=~/.ssh/id_ed25519
REMOTE_USER=arman
REMOTE_HOST=100.64.10.12
```

## 9. Final note

SSH key management is not a one-time setup. It is part of the ongoing security hygiene of the whole remote-access system.
