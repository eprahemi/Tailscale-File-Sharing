<p align="center">
  <img src="https://img.shields.io/badge/OS-Fedora%20Linux-294172?style=for-the-badge&logo=fedora" alt="Fedora">
  <img src="https://img.shields.io/badge/OS-Windows%2011-0078D4?style=for-the-badge&logo=windows" alt="Windows">
  <img src="https://img.shields.io/badge/Tailscale-Enabled-3B82F6?style=for-the-badge&logo=tailscale" alt="Tailscale">
  <img src="https://img.shields.io/badge/Samba-4.24-FF6600?style=for-the-badge&logo=samba" alt="Samba">
</p>

<h1 align="center">
  🔗 Cross-Platform File Sharing
  <br>
  <sub>Tailscale + Samba — Fedora ↔ Windows</sub>
</h1>

<p align="center">
  <strong>Access your entire Windows C: drive from Nautilus (Fedora)<br>
  and your Fedora home directory from File Explorer (Windows)<br>
  — all over a secure Tailscale VPN.</strong>
</p>

<p align="center">
  <img src="https://img.shields.io/github/license/eprahemi/Tailscale-File-Sharing?style=flat-square" alt="License">
  <img src="https://img.shields.io/github/stars/eprahemi/Tailscale-File-Sharing?style=flat-square" alt="Stars">
  <img src="https://img.shields.io/badge/prerequisites-2%20machines%20on%20Tailscale-blue?style=flat-square" alt="Prerequisites">
</p>

---

## 📖 Table of Contents

- [🎯 What You'll Get](#-what-youll-get)
- [📋 Prerequisites](#-prerequisites)
- [🔑 Finding Your Info (IP & Username)](#-finding-your-info-ip--username)
- [🐧 Fedora Setup](#-fedora-setup)
- [🪟 Windows Setup](#-windows-setup)
- [🧪 Verify It Works](#-verify-it-works)
- [📌 Quick Reference](#-quick-reference)
- [❓ Troubleshooting](#-troubleshooting)
- [📄 License](#-license)

---

## 🎯 What You'll Get

```
┌──────────────────────┐              Tailscale               ┌──────────────────────┐
│                      │◄────────────────────────────────────►│                      │
│   🐧 Fedora Linux    │                                       │   🪟 Windows 11       │
│                      │                                       │                      │
│  ┌────────────────┐  │                                       │  ┌────────────────┐  │
│  │   Nautilus     │  │   smb://WINDOWS_IP/C$/                │  │ File Explorer  │  │
│  │  (see Windows) │◄┼──────────────────────────────────────┼─►│  (see Fedora)   │  │
│  └────────────────┘  │   \\FEDORA_IP\FEDORA_USER             │  └────────────────┘  │
└──────────────────────┘                                       └──────────────────────┘

  WINDOWS_IP = 100.103.197.14   FEDORA_IP = 100.66.222.92   FEDORA_USER = eprahemi
```

| Direction          | What You Access | Application        |
|--------------------|-----------------|--------------------|
| 🪟 Windows → 🐧 Fedora | Your Fedora home directory | File Explorer on Windows |
| 🐧 Fedora → 🪟 Windows | Your entire Windows C: drive | Nautilus on Fedora |

**No cloud. No middleman. Direct peer-to-peer over Tailscale.**

---

## 📋 Prerequisites

| # | Requirement | How to Check |
|---|-------------|--------------|
| 1 | Both machines on the **same Tailscale network** | `tailscale status` — both must appear |
| 2 | Tailscale installed & running on both | `tailscale version` |
| 3 | Both can ping each other | `ping FEDORA_IP` (e.g. `ping 100.66.222.92`) |
| 4 | Internet connection (for initial setup) | — |
| 5 | Admin rights on both machines | Required for installs |

---

## 🔑 Finding Your Info (IP & Username)

### 🐧 On Fedora

**Your username:**
```bash
whoami
```
*Example output: `eprahemi`*

**Your Tailscale IP:**
```bash
tailscale ip -4
```
*Example output: `100.66.222.92`*

**Your hostname:**
```bash
hostname
```
*Example output: `eprahemi-lenevo-fedora`*

### 🪟 On Windows

**Your username:**
```powershell
whoami
```
*Example output: `eprahemi-msi\fsown`*

> [!TIP]
> If you see a Microsoft account email here, you'll need to create a **local user** for SMB (covered in Windows Setup Step 3).

**Your Tailscale IP:**
```powershell
tailscale ip -4
```
*Example output: `100.103.197.14`*

**Your computer name:**
```powershell
hostname
```
*Example output: `eprahemi-msi-windows`*

### Quick Reference Card

| What | Fedora Command | Windows Command |
|------|---------------|-----------------|
| Username | `whoami` | `whoami` |
| Tailscale IP | `tailscale ip -4` | `tailscale ip -4` |
| Hostname | `hostname` | `hostname` |

---

## 🐧 Fedora Setup

> Complete these steps **on your Fedora machine**.

### Step 1 — Install Tailscale

```bash
sudo dnf config-manager --add-repo https://pkgs.tailscale.com/stable/fedora/tailscale.repo
sudo dnf install tailscale -y
sudo systemctl enable tailscaled --now
sudo tailscale up
```

A browser window will open — sign in with your Google, Microsoft, or GitHub account.

### Step 2 — Get Your Fedora Tailscale IP

```bash
tailscale ip -4
```

📌 **Make a note of this IP** — you'll use it on Windows.

```
Your Fedora IP  →  100.x.x.x     (e.g. 100.66.222.92)
```

### Step 3 — Install & Configure Samba

Install Samba:

```bash
sudo dnf install samba samba-client -y
```

Set your SMB password (used when connecting from Windows):

```bash
sudo smbpasswd -a YOUR_USER      # YOUR_USER = your Fedora username (e.g. eprahemi)
```

Start Samba and allow it through the firewall:

```bash
sudo systemctl enable smb --now
sudo firewall-cmd --add-service=samba --permanent
sudo firewall-cmd --reload
```

Enable SELinux to allow Samba access to home directories:

```bash
sudo setsebool -P samba_enable_home_dirs on
```

✅ **Fedora is ready.** Your home directory is now shared.

### Step 4 — Connect to Windows from Nautilus

On your **Fedora machine**, open **Files (Nautilus)**:

1. Click **Other Locations** in the sidebar
2. In the **Connect to Server** box, enter:

```
smb://WINDOWS_IP/C$/     # WINDOWS_IP = your Windows Tailscale IP (e.g. 100.103.197.14)
```

3. Click **Connect**. When prompted:

| Field     | Value                                |
|-----------|--------------------------------------|
| Username  | `smbuser` (the user you'll create on Windows) |
| Password  | *(the password you set for smbuser on Windows)* |

4. Press **`Ctrl + D`** to bookmark it for quick access.

> [!TIP]
> Replace `C$` with `Downloads`, `Users`, or any other share name to limit access to a specific folder.

---

## 🪟 Windows Setup

> Complete these steps **on your Windows machine**.

### Step 1 — Install Tailscale

1. Download the installer: [tailscale.com/download-windows](https://tailscale.com/download-windows)
2. Run the installer
3. Click **Sign in** in the system tray — use the **same account** as Fedora

### Step 2 — Get Your Windows Tailscale IP

Open **PowerShell**:

```powershell
tailscale ip -4
```

📌 **Make a note of this IP** — you'll use it on Fedora.

```
Example: Your Windows IP → 100.103.197.14
```

### Step 3 — Create a Local SMB User

> **Why this is needed:** Microsoft accounts don't work well with SMB authentication. A dedicated local user avoids login issues.

Open **PowerShell as Administrator**:

```powershell
net user smbuser eprahemi /add
```
```powershell
net localgroup Administrators smbuser /add
```

### Step 4 — Remove Remote Admin Restrictions

> **Why this is needed:** Windows blocks admin shares (`C$`) over the network by default. This registry key removes that restriction.

In **PowerShell (Admin)**:

```powershell
reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /v LocalAccountTokenFilterPolicy /t REG_DWORD /d 1 /f
```

**🔁 Restart your computer** for the change to take effect.

### Step 5 — Connect to Fedora from File Explorer

On your **Windows machine**, open **File Explorer**:

1. In the address bar, type:

```
\\FEDORA_IP\FEDORA_USER     # FEDORA_IP = your Fedora IP (e.g. 100.66.222.92), FEDORA_USER = your username (e.g. eprahemi)
```

2. Press Enter. When prompted:

| Field     | Value                                       |
|-----------|---------------------------------------------|
| Username  | `FEDORA_USER` (your Fedora username, e.g. `eprahemi`) |
| Password  | *(the SMB password you set on Fedora)*      |

3. **Map as network drive** for permanent access (optional):
   - Right-click **This PC** → **Map network drive**
   - Pick a drive letter (e.g., `Z:`)
   - Paste `\\FEDORA_IP\FEDORA_USER`     # e.g. \\100.66.222.92\eprahemi
   - Check **Connect using different credentials** if needed

---

## 🧪 Verify It Works

### From Fedora (test Windows share)

```bash
smbclient //WINDOWS_IP/C$/ -U smbuser     # e.g. //100.103.197.14/C$/
```

You'll be prompted for the password — enter the one you set for `smbuser`. If you see a file listing, ✅ success.

### From Windows (test Fedora share)

```powershell
net use Z: \\FEDORA_IP\FEDORA_USER     # e.g. \\100.66.222.92\eprahemi
```

If the command completes without errors, ✅ success.

---

## 📌 Quick Reference

### Connection Strings

```
┌──────────────────────────────────────────────────────────────┐
│                     CONNECTION STRINGS                        │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  🐧 Fedora → Windows (in Nautilus):                          │
│  smb://WINDOWS_IP/C$/                                        │
│  (e.g. smb://100.103.197.14/C$/)                             │
│                                                              │
│  🪟 Windows → Fedora (in File Explorer):                     │
│  \\FEDORA_IP\FEDORA_USER                                     │
│  (e.g. \\100.66.222.92\eprahemi)                             │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Common Variables

| Variable       | Description                    | Example              |
|----------------|--------------------------------|----------------------|
| `WINDOWS_IP`   | Tailscale IP of Windows machine | `100.103.197.14`    |
| `FEDORA_IP`    | Tailscale IP of Fedora machine  | `100.66.222.92`     |
| `FEDORA_USER`  | Your Fedora Linux username      | `eprahemi`           |
| `smbuser`      | Windows SMB user (you create)   | `smbuser`            |

### Useful Commands

| Action | Command |
|--------|---------|
| Check Tailscale status | `tailscale status` |
| Get Tailscale IP | `tailscale ip -4` |
| Check Samba status | `sudo systemctl status smb` |
| Restart Samba | `sudo systemctl restart smb` |
| List firewall services | `sudo firewall-cmd --list-services` |
| Check SELinux status | `getenforce` |

---

## ❓ Troubleshooting

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| ⏱ **Timeout / can't connect** | Tailscale not running or firewall blocking | `ping FEDORA_IP` and `ping WINDOWS_IP` from each machine<br>`sudo systemctl status smb` — is Samba running?<br>`sudo firewall-cmd --list-services` — is `samba` listed? |
| 🚫 **Access denied (Windows → Fedora)** | SELinux blocking Samba | Run `sudo setsebool -P samba_enable_home_dirs on`<br>Check `valid users` in `/etc/samba/smb.conf` |
| 🚫 **Access denied (Fedora → Windows)** | UAC remote restriction | Confirm `LocalAccountTokenFilterPolicy` registry key is set<br>Reboot Windows<br>Verify `smbuser` is in Administrators group |
| 🔑 **Authentication failed** | Wrong password or Microsoft account | Use the SMB-specific password, not your Windows PIN<br>Use local `smbuser`, not your Microsoft account |
| 📁 **Permission denied on folder** | Share permissions not set | Windows: right-click folder → Properties → Sharing → grant `smbuser` access |
| 🔇 **Can't browse shares** | SMB1 disabled (this is normal) | Use direct path instead of browsing |

---

## 📄 License

This guide is provided under the **MIT License**.

Feel free to use, modify, and distribute it. If you found it helpful, a ⭐ on GitHub is appreciated!
