# 13. Monitoring & Logs

This note covers the operational side of maintaining a remote SSH-enabled host.

> Public-safe note: never commit real hostnames, real IPs, production logs, or secrets.

## 1. Why monitoring matters

Once a laptop or Linux host is reachable over SSH, you should monitor:
- service status
- uptime
- CPU and memory usage
- disk health
- firewall state
- failed login attempts

## 2. Windows monitoring commands

```powershell
Get-Service sshd
Get-Process
Get-Counter "\Processor(_Total)\% Processor Time"
Get-Volume
Get-WinEvent -LogName Security -MaxEvents 20
```

## 3. Linux monitoring commands

```bash
systemctl status ssh
uptime
free -h
df -h
journalctl -u ssh -n 50
```

## 4. Log review checklist

- SSH service logs
- failed login attempts
- Tailscale status logs
- firewall log review
- unusual remote access patterns

## 5. Example placeholders

```text
HOST_NAME=work-laptop
SSH_USER=arman
MESH_IP=100.64.10.12
LOG_SOURCE=sshd
```

## 6. Best practices

- keep logs for review
- monitor service health regularly
- watch for repeated failed login attempts
- check Tailscale status after reboot

## 7. Final note

Remote access without monitoring is fragile. Monitoring makes the setup operationally safe and maintainable.
