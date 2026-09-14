# Fleet SSH Setup — Windows Laptop over Tailscale / Headscale

এই রিপোজিটরি একটি আধুনিক, নিরাপদ ও রিমোট-ম্যানেজমেন্ট-ফ্রেন্ডলি SSH সেটআপ গাইড। এখানে Windows ল্যাপটপে OpenSSH সার্ভার ইন্সটল করে Tailscale/Headscale মেশ নেটওয়ার্কের মধ্যে নির্দিষ্ট ডিভাইস থেকে SSH-এ ঢোকা যায়।

> নোট: এই README-এ যে IP, ইউজারনেম ও ডিভাইস নাম দেখানো হয়েছে, সেগুলো শুধু ডেমো/ডামি ডেটা। Public repo-তে real IP বা ব্যক্তিগত নাম ব্যবহার করা হয়নি। নিজের সিস্টেমের জন্য এগুলো নিজে বদলে নিন।

## সূচিপত্র

- [১. প্রয়োজনীয়তা ও প্রস্তুতি](#1-প্রয়োজনীয়তা-ও-প্রস্তুতি)
- [২. সেটআপ স্ক্রিপ্ট](#2-সেটআপ-স্ক্রিপ্ট)
- [৩. স্ক্রিপ্ট কী করে](#3-স্ক্রিপ্ট-কী-করে)
- [৪. সিকিউরিটি বেস্ট প্র্যাকটিস](#4-সিকিউরিটি-বেস্ট-প্র্যাকটিস)
- [৫. SSH-এ ঢুকে কী কী করা যাবে](#5-ssh-এ-ঢুকে-কী-কী-করা-যাবে)
- [৬. দ্রুত চেকলিস্ট](#6-দ্রুত-চেকলিস্ট)
- [৭. সমস্যা সমাধান](#7-সমস্যা-সমাধান)

---

## 1. প্রয়োজনীয়তা ও প্রস্তুতি

### ১.১ PowerShell Admin মোড

- Start মেনুতে রাইট-ক্লিক করে `Windows Terminal (Admin)` বা `PowerShell (Admin)` খুলুন
- `Set-StrictMode -Version Latest` বা `$ErrorActionPreference = "Stop"` রাখুন, যাতে স্ক্রিপ্টে কোনো ত্রুটি হলে থেমে যায়

### ১.২ Tailscale / Headscale সেটআপ

স্ক্রিপ্ট চালানোর আগে Tailscale ইনস্টল ও Headscale নেটওয়ার্কে কানেক্টেড থাকতে হবে।

```powershell
winget install tailscale.tailscale
```

Headscale URL আপনার server-এ HTTPS endpoint হতে হবে, উদাহরণ:

```text
https://headscale.example.com
https://hs.yourdomain.com
https://your-server-ip:443
```

তারপর Headscale সার্ভারের সাথে কানেক্ট করুন:

```powershell
tailscale up --login-server https://<your-headscale-server-url>
```

যদি Auth Key থাকে, তাহলে:

```powershell
tailscale up --login-server https://<your-headscale-server-url> --authkey <your-auth-key>
```

> Public repo-তে real URL, real IP, auth key, secret বা production domain কখনো লিখবেন না। `YOUR_HEADSCALE_URL`/`YOUR_AUTH_KEY`-এর মতো placeholder ব্যবহার করুন।

সফল হলে ডিভাইসটি `100.64.x.x` ফরম্যাটের একটি private mesh IP পাবে।

### ১.৩ নতুন IP নোট করুন

```powershell
tailscale ip -4
```

এই IPটি অন্য মেশিনের firewall rule / allowed peer লিস্টে যোগ করতে হবে, যাতে অন্য ডিভাইসগুলো এই Windows ল্যাপটপে SSH-এ ঢুকতে পারে।

### ১.৪ ইউজার অ্যাকাউন্ট

স্ক্রিপ্টের উদাহরণে `Arman` ব্যবহার করা হয়েছে। নিশ্চিত করুন:

- Windows-এ `Arman` ব্যবহারকারী আছে
- এই অ্যাকাউন্টটি Administrator বা যথেষ্ট privilege-সম্পন্ন
- SSH login করার পর কমান্ড চালানোর জন্য দরকারি permissions আছে

### ১.৫ Placeholder template

Public repo-তে নিচের ফরম্যাট ব্যবহার করুন:

```text
HEADSCALE_URL=https://<your-headscale-server-url>
WINDOWS_USERNAME=Arman
TRUSTED_PEER_1=100.64.10.2
TRUSTED_PEER_2=100.64.10.5
REMOTE_SSH_HOST=100.64.10.12
```

---

## 2. সেটআপ স্ক্রিপ্ট

Admin PowerShell-এ নিচের স্ক্রিপ্টটি রান করুন:

```powershell
$ErrorActionPreference = "Stop"

# 1) OpenSSH Server install
$ssh = Get-WindowsCapability -Online -Name "OpenSSH.Server~~~~0.0.1.0"
if ($ssh.State -ne "Installed") {
    Add-WindowsCapability -Online -Name $ssh.Name
}

# 2) Start SSH service
Set-Service sshd -StartupType Automatic
Start-Service sshd
sc.exe failure sshd reset= 86400 actions= restart/5000/restart/15000/restart/30000

# 3) Keep Tailscale running on startup
if (Get-Service Tailscale -ErrorAction SilentlyContinue) {
    Set-Service Tailscale -StartupType Automatic
    Start-Service Tailscale
}

# 4) Allow SSH only from trusted mesh peers
$peers = @("100.64.10.2", "100.64.10.5")
if (Get-NetFirewallRule -Name "Fleet-SSH" -ErrorAction SilentlyContinue) {
    Remove-NetFirewallRule -Name "Fleet-SSH"
}

New-NetFirewallRule -Name "Fleet-SSH" `
    -DisplayName "Fleet SSH" `
    -Direction Inbound `
    -Action Allow `
    -Protocol TCP `
    -LocalPort 22 `
    -RemoteAddress $peers `
    -Profile Any

# 5) Keep laptop awake when on AC power
powercfg /change standby-timeout-ac 0
powercfg /change hibernate-timeout-ac 0

# 6) Final verification
Get-Service sshd,Tailscale -ErrorAction SilentlyContinue
Get-NetTCPConnection -State Listen -LocalPort 22
Write-Host "SSH is ready. Test from trusted peer: ssh Arman@100.64.10.12"
```

> Placeholder version: replace `Arman` with your user, and replace `100.64.10.2`, `100.64.10.5`, `100.64.10.12` with your actual mesh IPs before using in a real environment.

> গুরুত্বপূর্ণ: `$peers`-এ থাকা IP এবং `Arman` নাম নিজস্ব নেটওয়ার্ক অনুযায়ী বদলে নিন। উপরের IP-গুলো ডেমো ডেটা মাত্র।

---

## 3. স্ক্রিপ্ট কী করে

| ধাপ                       | কী কাজ করে                                                          |
| ------------------------- | ------------------------------------------------------------------- |
| OpenSSH Capability চেক    | Windows-এ বিল্ট-ইন OpenSSH Server feature install করে               |
| `sshd` service চালু       | কম্পিউটার রিস্টার্টের পরও SSH সার্ভার চলতে থাকে                     |
| `sc.exe failure`          | সার্ভিস ক্র্যাশ হলে ৫s/১৫s/৩০s-এ auto-restart করে                   |
| Tailscale start-up config | Headscale mesh কানেকশন সবসময়ভাবে লাইভ থাকে                         |
| Firewall rule `Fleet-SSH` | শুধুমাত্র নির্দিষ্ট mesh peer-দের কাছ থেকে Port 22-এ SSH ঢুকতে দেয় |
| `powercfg`                | AC পাওয়ারে থাকলে sleep/hibernate বন্ধ রাখে                         |
| শেষের চেক                 | SSH কনফিগ ঠিক আছে কি না, Port 22 listen করছে কি না তা পরীক্ষা করে   |

### সতর্কতা

- `sleep`/`hibernate` শুধুমাত্র AC power-এ বন্ধ থাকে; ব্যাটারি মোডে ল্যাপটপ ঘুমিয়ে যেতে পারে।
- Firewall rule IP-based, তাই peer IP বদলে গেলে rule আপডেট করতে হবে।
- Default password login অন থাকতে পারে; secure করার জন্য key-based auth setup করা উচিত।

---

## 4. সিকিউরিটি বেস্ট প্র্যাকটিস

### ৪.১ Key-based authentication

Remote machine (যেমন অন্য কম্পিউটার) থেকে:

```bash
ssh-copy-id Arman@100.64.10.12
```

অথবা ম্যানুয়ালি `C:\Users\Arman\.ssh\authorized_keys`-এ public key বসিয়ে দিন।

### ৪.২ PasswordAuthentication বন্ধ করা

`C:\ProgramData\ssh\sshd_config`-এ এই লাইনটি যোগ/এডিট করুন:

```text
PasswordAuthentication no
```

তারপর:

```powershell
Restart-Service sshd
```

### ৪.৩ SSH config hardening

নিরাপত্তার জন্য `sshd_config`-এ কিছু অতিরিক্ত সেটিং ব্যবহার করা যেতে পারে:

```text
PermitRootLogin no
PubkeyAuthentication yes
PasswordAuthentication no
X11Forwarding no
AllowUsers Arman

# Optional
ListenAddress 0.0.0.0
```

### ৪.৪ Firewall best practice

- শুধুমাত্র নির্দিষ্ট peer IP-ই SSH-এ ঢুকতে পারবে
- Port 22 open রাখলেও mesh-private network-এ সীমাবদ্ধ রাখুন
- সময়ে সময়ে firewall rules review করুন

---

## 5. SSH-এ ঢুকে কী কী করা যাবে

একবার SSH কানেক্ট হলে, Windows ল্যাপটপের উপর **কমান্ড-লাইন থেকে প্রায় সব কাজ** করা যাবে:

### ৫.১ System monitoring

- `Get-Process`, `Get-Counter`, `Get-Volume` দিয়ে CPU/RAM/Disk usage দেখা
- `Get-EventLog`, `Get-WinEvent` দিয়ে লগ বিশ্লেষণ
- `Get-Service`, `Start-Service`, `Stop-Service` দিয়ে services control

### ৫.২ File operations

- `scp` বা `sftp` ব্যবহার করে file upload/download
- `Copy-Item`, `Move-Item`, `Remove-Item` দিয়ে folder/file management
- WSL বা external tools দিয়ে large dataset sync করা

### ৫.৩ Script and automation execution

- PowerShell, Python, JavaScript বা অন্য installed runtime script চালানো
- Task Scheduler-এ automation task তৈরি
- Remote training/inference job trigger করা (GPU থাকলে)

### ৫.৪ Remote network access

- Firewall rule add/remove
- Port forwarding / tunneling setup
- jump host হিসাবে ব্যবহার করে অন্য ডিভাইসে SSH-এ পৌঁছানো

### ৫.৫ Power control

- `Restart-Computer`, `shutdown /r` দিয়ে reboot/restart
- power plan change করা
- Wake-on-LAN আলাদাভাবে সেটআপ থাকলে remote wake-up করা

### ৫.৬ যা করা যাবে না

- BIOS/UEFI settings পরিবর্তন
- hardware-level issue diagnose/repair
- সম্পূর্ণ shut down থাকা ডিভাইস চালু করা, যদি Wake-on-LAN না থাকে
- GUI-based কাজ, যদি RDP/VNC আলাদা ভাবে setup না থাকে

---

## 6. দ্রুত চেকলিস্ট

1. ⬜ Tailscale ইনস্টল ও Headscale-এ join
2. ⬜ নতুন mesh IP নোট করে অন্য ডিভাইসে আপডেট
3. ⬜ `Arman` ব্যবহারকারী তৈরি/নিশ্চিত করুন
4. ⬜ Admin PowerShell-এ setup script রান করুন
5. ⬜ Key-based SSH auth সেটআপ করুন
6. ⬜ PasswordAuthentication বন্ধ করুন
7. ⬜ টেস্ট কমান্ড রান করুন: `ssh Arman@100.64.10.12`

---

## 7. সমস্যা সমাধান

### SSH connection refused

- `Get-Service sshd` চেক করুন
- `Get-NetTCPConnection -State Listen -LocalPort 22` দেখুন
- Firewall rule `Fleet-SSH` আছে কি না verify করুন

### Tailscale peer cannot reach host

- `tailscale status` রান করুন
- `tailscale ip -4` দিয়ে ঠিক IP আছে কি না যাচাই করুন
- peer device-এ firewall rule/allowed peer list update হয়েছে কি না দেখুন

### Authentication failed

- `Arman` user exists কিনা verify করুন
- public key ঠিকভাবে `authorized_keys`-এ আছে কি না দেখুন
- `sshd_config`-এ `PasswordAuthentication no` সেট আছে কিনা verify করুন

### Laptop sleeps unexpectedly

- `powercfg /query` দিয়ে current power plan inspect করুন
- AC mode-এ `standby-timeout-ac 0` সেট আছে কি না verify করুন
- laptop power adapter connected আছে কি না দেখুন

---

## প্রাথমিক টেস্ট কমান্ড

```bash
ssh Arman@100.64.10.12
whoami
hostname
Get-ComputerInfo | Select-Object CsName,WindowsVersion
```

যদি এই কমান্ডগুলো সফল হয়, তাহলে SSH setup সঠিকভাবে কাজ করছে।

---

## নির্দেশনা

- এই README-এ প্রদত্ত IP এবং username শুধুমাত্র উদাহরণ
- production environment-এ নিজের ঠিক IP, user name ও headscale URL ব্যবহার করুন
- public repo-তে sensitive data কখনো commit করবেন না
- security-sensitive setup-এ always review firewall, authentication এবং network allow-list
