# 07. Access Policy & Best Practices

এই note-এ SSH access policy কীভাবে setup করবেন, তা ব্যাখ্যা করা হবে।

> নোট: public repo-তে real IP, domain, auth key, live endpoint বা secret রাখা যাবে না।

## ১. Why access policy matters

SSH access without policy becomes risky. You need to define:

- who can connect
- from which devices
- which port is allowed
- whether key-based auth is required

## ২. Minimal policy example

```text
HOST=work-laptop
ALLOWED_PEERS=100.64.10.2, 100.64.10.5
PORT=22
AUTH=SSH key only
USER=arman
```

## ৩. Recommended policy

- only trusted peers can connect
- password auth disabled
- key auth required
- root login disabled
- allow-user restricted to single safe user

## ৪. Example rule set

```text
ALLOW_SSH_FROM=100.64.10.2
ALLOW_SSH_FROM=100.64.10.5
DISABLE_PASSWORD_LOGIN=yes
REQUIRE_SSH_KEY=yes
ALLOW_USER=arman
```

## ৫. Access review checklist

1. ⬜ Who is allowed?
2. ⬜ Which IPs are allowed?
3. ⬜ What port is open?
4. ⬜ Is key-based auth required?
5. ⬜ Is password login disabled?
6. ⬜ Was the rule reviewed recently?

## ৬. Good default policy

```text
SSH is allowed only from verified Tailscale peers.
Password login is disabled.
Key-based authentication is mandatory.
User access is restricted to a single named account.
```

## ৭. Final note

Access policy is what keeps your remote environment safe. Without a clear policy, small mistakes turn into big security holes.
