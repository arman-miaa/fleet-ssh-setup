# Fleet SSH Setup — Windows Laptop over Tailscale/Headscale

Windows ল্যাপটপে OpenSSH সার্ভার সেটআপ করার স্ক্রিপ্ট ও গাইড, যাতে Tailscale/Headscale মেশ নেটওয়ার্কের নির্দিষ্ট ডিভাইস (DGX, Mac mini) থেকে নিরাপদে SSH করা যায়।

## সূচিপত্র
- [প্রি-রিকুইজিট](#১-প্রি-রিকুইজিট-স্ক্রিপ্ট-চালানোর-আগে-করতে-হবে)
- [সেটআপ স্ক্রিপ্ট](#সেটআপ-স্ক্রিপ্ট)
- [স্ক্রিপ্ট কী করে](#২-মূল-স্ক্রিপ্ট-কী-করে-লাইন-বাই-লাইন)
- [অতিরিক্ত নিরাপত্তা](#৩-অতিরিক্ত-নিরাপত্তা-সেটআপ-ঐচ্ছিক-কিন্তু-সুপারিশকৃত)
- [রিমোট ক্যাপাবিলিটি](#৪-কানেক্ট-হয়ে-গেলে--ডিভাইসে-হাত-না-দিয়ে-কী-কী-করা-যাবে)
- [চেকলিস্ট](#৫-দ্রুত-চেকলিস্ট-নতুন-ল্যাপটপে)

---

## ১. প্রি-রিকুইজিট (স্ক্রিপ্ট চালানোর আগে করতে হবে)

### ১.১ PowerShell Administrator মোডে খুলুন
- Start মেনুতে right-click → "Terminal (Admin)" বা "Windows PowerShell (Admin)"
- `$ErrorActionPreference = "Stop"` থাকায় যেকোনো এরর হলে স্ক্রিপ্ট থেমে যাবে — এটা ইচ্ছাকৃত সেফটি।

### ১.২ Tailscale ইনস্টল ও Headscale নেটওয়ার্কে জয়েন
স্ক্রিপ্টটি ধরে নেয় Tailscale **আগে থেকেই ইনস্টল ও কানেক্টেড**। নতুন ল্যাপটপে এটা প্রথমে করতে হবে:

```powershell
winget install tailscale.tailscale
```

ইনস্টলের পর, আপনার Headscale সার্ভারের সাথে জয়েন করুন:

```powershell
tailscale up --login-server https://<আপনার-headscale-সার্ভার-URL>
```

- এটা চালালে একটা লগইন লিংক আসবে, ব্রাউজারে খুলে অথরাইজ করতে হবে (অথবা Headscale অ্যাডমিন প্রি-অথ কী দিলে `--authkey` ব্যবহার করুন)।
- সফল হলে এই ল্যাপটপ একটা নতুন `100.64.x.x` টাইপ প্রাইভেট IP পাবে।

### ১.৩ নতুন IP নোট করুন এবং অন্য মেশিনে আপডেট করুন
```powershell
tailscale ip -4
```
- এই IP-টা DGX আর Mac mini-র ফায়ারওয়াল/allowed-peer লিস্টে যোগ করতে হবে, নাহলে ওরা এই নতুন ল্যাপটপে SSH করতে পারবে না (one-way visibility problem)।

### ১.৪ ইউজার অ্যাকাউন্ট
স্ক্রিপ্টে `Roni` ইউজারনেম হার্ডকোড করা আছে টেস্ট কমান্ডে। নিশ্চিত করুন:
- এই নামের লোকাল/মাইক্রোসফট অ্যাকাউন্ট আছে
- Administrator বা প্রয়োজনীয় গ্রুপে আছে (নাহলে SSH দিয়ে লগইন করলেও অনেক কমান্ড এক্সেস পাবে না)

---

## সেটআপ স্ক্রিপ্ট

Administrator PowerShell-এ এটা রান করুন:

```powershell
$ErrorActionPreference = "Stop"

# Install and start SSH.
$ssh = Get-WindowsCapability -Online -Name "OpenSSH.Server~~~~0.0.1.0"
if ($ssh.State -ne "Installed") {
    Add-WindowsCapability -Online -Name $ssh.Name
}
Set-Service sshd -StartupType Automatic
Start-Service sshd
sc.exe failure sshd reset= 86400 actions= restart/5000/restart/15000/restart/30000

# Keep the existing Headscale connection running.
if (Get-Service Tailscale -ErrorAction SilentlyContinue) {
    Set-Service Tailscale -StartupType Automatic
    Start-Service Tailscale
}

# Allow SSH from DGX and Mac mini through the mesh.
$peers = @("100.64.0.2", "100.64.0.5")
if (Get-NetFirewallRule -Name "Fleet-SSH" -ErrorAction SilentlyContinue) {
    Remove-NetFirewallRule -Name "Fleet-SSH"
}
New-NetFirewallRule -Name "Fleet-SSH" -DisplayName "Fleet SSH" `
    -Direction Inbound -Action Allow -Protocol TCP -LocalPort 22 `
    -RemoteAddress $peers -Profile Any

# Prevent sleep while connected to power.
powercfg /change standby-timeout-ac 0
powercfg /change hibernate-timeout-ac 0

# Show whether SSH is ready.
Get-Service sshd,Tailscale -ErrorAction SilentlyContinue
Get-NetTCPConnection -State Listen -LocalPort 22
Write-Host "Ready for SSH testing from DGX: ssh Roni@100.64.0.6"
```

> **নোট:** `$peers` লিস্টের IP ও `Roni` ইউজারনেম আপনার নিজের সেটআপ অনুযায়ী বদলে নিন।

---

## ২. মূল স্ক্রিপ্ট কী করে (লাইন-বাই-লাইন)

| ধাপ | কাজ |
|---|---|
| OpenSSH ক্যাপাবিলিটি চেক ও ইনস্টল | Windows-এর বিল্ট-ইন SSH সার্ভার ফিচার অন করে |
| `sshd` সার্ভিস অটোস্টার্ট + চালু | রিবুটের পরও SSH সার্ভার নিজে থেকে চলবে |
| `sc.exe failure` রিকভারি রুল | সার্ভিস ক্র্যাশ করলে ৫সে./১৫সে./৩০সে. পর পর অটো-রিস্টার্ট |
| Tailscale সার্ভিস অটোস্টার্ট + চালু | মেশ নেটওয়ার্ক কানেকশন পার্সিস্ট্যান্ট রাখে |
| `Fleet-SSH` ফায়ারওয়াল রুল | শুধু DGX (`100.64.0.2`) আর Mac mini (`100.64.0.5`) থেকে পোর্ট 22-এ ঢুকতে দেয়; আগের রুল থাকলে মুছে নতুন করে বসায় |
| `powercfg` কমান্ড দুটো | AC পাওয়ারে থাকলে sleep/hibernate বন্ধ, যাতে সবসময় রিচেবল থাকে |
| শেষের চেকগুলো | সার্ভিস স্ট্যাটাস, পোর্ট 22 লিসেনিং কিনা, আর টেস্ট SSH কমান্ড দেখায় |

### সতর্কতা / সাইড এফেক্ট
- **শুধু AC পাওয়ারে sleep বন্ধ** — ব্যাটারিতে চললে ল্যাপটপ ঘুমিয়ে যাবে, তখন SSH আনরিচেবল হবে।
- **ফায়ারওয়াল রুল IP-বেসড**, key-based auth নয় — যদি DGX/Mac mini-র IP কোনোভাবে বদলে যায় (Tailscale re-key, ডিভাইস রিসেট), এই রুল আর কাজ করবে না, রুল আপডেট করতে হবে।
- **পাসওয়ার্ড authentication ডিফল্টে অন থাকতে পারে** — নিরাপত্তার জন্য key-based auth সেটআপ করে পাসওয়ার্ড লগইন বন্ধ করে দেওয়া ভালো (নিচে দেখুন)।

---

## ৩. অতিরিক্ত নিরাপত্তা সেটআপ (ঐচ্ছিক কিন্তু সুপারিশকৃত)

### Key-based auth সেটআপ (পাসওয়ার্ডবিহীন, বেশি নিরাপদ)
DGX/Mac mini থেকে:
```bash
ssh-copy-id Roni@100.64.0.6
```
অথবা ম্যানুয়ালি পাবলিক কী `C:\Users\Roni\.ssh\authorized_keys`-এ বসিয়ে দিন।

### পাসওয়ার্ড auth বন্ধ (key বসানোর পর)
`C:\ProgramData\ssh\sshd_config` ফাইলে:
```
PasswordAuthentication no
```
তারপর: `Restart-Service sshd`

---

## ৪. কানেক্ট হয়ে গেলে — ডিভাইসে হাত না দিয়ে কী কী করা যাবে

একবার SSH দিয়ে ঢুকতে পারলে, DGX বা Mac mini থেকে এই উইন্ডোজ ল্যাপটপে **প্রায় সব কিছু** কমান্ড লাইনে করা যায়:

### ৪.১ সিস্টেম মনিটরিং ও ম্যানেজমেন্ট
- CPU/RAM/ডিস্ক ইউজেজ চেক (`Get-Process`, `Get-Counter`, `Get-Volume`)
- ইভেন্ট লগ দেখা (`Get-EventLog`, `Get-WinEvent`)
- চলমান সার্ভিস/প্রসেস স্টার্ট-স্টপ-কিল করা (`Start-Service`, `Stop-Process`)
- সিস্টেম ইনফো, আপটাইম, ইনস্টলড সফটওয়্যার লিস্ট দেখা

### ৪.২ ফাইল অপারেশন
- `scp` বা `sftp` দিয়ে ফাইল আপলোড/ডাউনলোড
- ফাইল/ফোল্ডার তৈরি, মোছা, কপি, মুভ, পারমিশন বদলানো
- বড় ডেটাসেট বা মডেল ফাইল সিঙ্ক করা (rsync-এর মতো টুল উইন্ডোজেও চালানো যায় WSL দিয়ে)

### ৪.৩ প্রোগ্রাম/স্ক্রিপ্ট রান করা
- PowerShell/Python/অন্য যেকোনো ইনস্টলড ভাষার স্ক্রিপ্ট রিমোটলি এক্সিকিউট করা
- শিডিউলড টাস্ক তৈরি বা ট্রিগার করা (`Task Scheduler` কমান্ড লাইন থেকে)
- GPU থাকলে সেখানে ট্রেনিং/ইনফারেন্স জব চালানো ও লগ মনিটর করা

### ৪.৪ নেটওয়ার্ক ও রিমোট এক্সেস
- নতুন ফায়ারওয়াল রুল বা পোর্ট ফরওয়ার্ড সেটআপ
- অন্য সার্ভিস (যেমন ওয়েব সার্ভার, ডাটাবেস) রিমোটলি চালু/বন্ধ করা
- ওই ল্যাপটপকে **jump host** হিসেবে ব্যবহার করে নেটওয়ার্কের অন্য ডিভাইসে পৌঁছানো (SSH tunneling/port forwarding দিয়ে)

### ৪.৫ পাওয়ার/সিস্টেম কন্ট্রোল
- রিমোট রিস্টার্ট/শাটডাউন (`Restart-Computer`, `shutdown /r`)
- পাওয়ার প্ল্যান বদলানো
- (Wake-on-LAN আলাদাভাবে সেটআপ থাকলে) বন্ধ থাকা ডিভাইস অন করাও সম্ভব — কিন্তু সেটা এই স্ক্রিপ্টের অংশ না, আলাদা করে করতে হবে

### ৪.৬ যা করা যাবে **না** (ফিজিক্যাল এক্সেস ছাড়া)
- BIOS/UEFI সেটিংস পরিবর্তন
- হার্ডওয়্যার-লেভেল সমস্যা (যেমন ব্যাটারি, RAM ফল্ট) ডায়াগনোজ/ফিক্স
- সম্পূর্ণ বন্ধ (shutdown, hibernate না) থাকা ডিভাইস চালু করা — যদি না Wake-on-LAN আলাদাভাবে কনফিগার করা থাকে
- GUI-নির্ভর কিছু কাজ, যদি না RDP/VNC-এর মতো আলাদা রিমোট-ডেস্কটপ টুল সেটআপ করা থাকে (SSH দিয়ে ডিফল্টে শুধু কমান্ড-লাইন এক্সেস পাওয়া যায়)

---

## ৫. দ্রুত চেকলিস্ট (নতুন ল্যাপটপে)

1. ⬜ Tailscale ইনস্টল + Headscale-এ জয়েন
2. ⬜ নতুন IP নোট করে DGX/Mac mini-তে আপডেট
3. ⬜ `Roni` ইউজার অ্যাকাউন্ট তৈরি/কনফার্ম
4. ⬜ মূল স্ক্রিপ্ট Admin PowerShell-এ রান
5. ⬜ Key-based SSH auth সেটআপ
6. ⬜ পাসওয়ার্ড auth বন্ধ
7. ⬜ DGX থেকে টেস্ট: `ssh Roni@100.64.0.6`
