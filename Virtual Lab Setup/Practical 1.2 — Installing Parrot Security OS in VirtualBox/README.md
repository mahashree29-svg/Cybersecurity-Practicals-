# Practical 1.2 — Installing Parrot Security OS in VirtualBox

## Objective

Import Parrot Security OS into VirtualBox as a guest virtual machine. Parrot Security is the attacker workstation for every offensive practical in this series — it ships with 800+ pre-installed tools for penetration testing, digital forensics, reverse engineering, and OSINT.

## Why Parrot Security (and not Kali)?

Parrot and Kali are the two mainstream pentest distros. Both are Debian-based, both include the same core tools. Differences worth knowing:

| Aspect | Parrot Security | Kali Linux |
|---|---|---|
| Base | Debian Stable | Debian Testing |
| Default desktop | KDE Plasma 6 (since v7.0) | Xfce |
| RAM footprint | Lighter | Heavier |
| User model | Non-root by default | Non-root by default (since 2020) |
| Best for | Privacy + pentesting | Pure pentesting |

Either works. The course uses Parrot, so we use Parrot.

## Prerequisites

| Requirement | Details |
|---|---|
| Host | VirtualBox 7.x already installed (see Practical 1.1) |
| RAM | 4 GB minimum allocated to the VM, 8 GB recommended |
| Disk | 40 GB free for the VM |
| CPU virtualization | VT-x or AMD-V enabled in BIOS |
| Internet | Required for first-time download (7.8 GB) |

## Choose Your Install Method

The official Parrot project distributes two formats. Pick one:

| Method | File | When to use |
|---|---|---|
| **OVA** (pre-built VM) | `.ova` | Fastest — import and run. Used when an OVA is available. |
| **ISO** (full installer) | `.iso` | More control — choose partitions, encryption, desktop variant. Always current. |

> ⚠️ The course manual references `Parrot-security4.10_virtual.ova` from 2020. That image is **six years out of date** and should not be used — its kernel and tools have known unpatched CVEs. Always download the current release from `parrotsec.org`.

The current stable release at the time of writing is **Parrot OS 7.2 (May 2026)**.

## Method A — Import OVA (if available)

### Step 1 — Download the OVA

1. Go to https://parrotsec.org/download/
2. Choose **Security Edition** → **Virtual Machines** → **VirtualBox**
3. Save the `.ova` file (around 8 GB)
4. **Verify the SHA-256 hash** against the value published on the Parrot download page:
   ```
   certutil -hashfile Parrot-security-7.2_amd64.ova SHA256
   ```
   The output must match the official hash exactly. If it doesn't, delete the file and re-download — never run an unverified pentest distro image.

### Step 2 — Import into VirtualBox

1. Open **Oracle VM VirtualBox Manager**
2. **File → Import Appliance** (or just double-click the `.ova` file)
3. Browse to the downloaded `.ova` → **Next**
4. Review the appliance settings:
   - Name: `Parrot-Security-7.2`
   - CPU: 2 cores minimum (4 if you can spare them)
   - RAM: 4096 MB minimum (8192 MB recommended)
   - Disk: ~80 GB virtual disk (dynamic — only uses what's actually written)
5. Tick **Reinitialize the MAC address of all network cards** — important so it doesn't collide with the source VM's MAC
6. Click **Finish**

Import takes 5–15 minutes depending on disk speed.

### Step 3 — First Boot

1. Select the new VM in VirtualBox → click **Start**
2. Default credentials (these vary by release — check the Parrot download page):
   - User: `parrot`
   - Password: `parrot`
3. **Immediately change the password:**
   ```bash
   passwd
   ```
4. Update the system before doing anything else:
   ```bash
   sudo parrot-upgrade
   ```

## Method B — Install from ISO

Use this if no current OVA is published, or if you want a custom install.

### Step 1 — Download the ISO

1. Go to https://parrotsec.org/download/ → **Security Edition** → **ISO**
2. Save the `.iso` file (around 7.8 GB)
3. Verify the SHA-256 hash exactly as above

### Step 2 — Create a New VM

1. VirtualBox Manager → **New**
2. Settings:
   - Name: `Parrot-Security-7.2`
   - Type: `Linux`
   - Version: `Debian (64-bit)`
   - RAM: 4096 MB minimum
   - Create a virtual hard disk now → VDI → Dynamically allocated → 40 GB
3. After creation, select VM → **Settings**:
   - **System → Processor:** 2 CPUs, enable PAE/NX
   - **Display:** Video Memory 128 MB, enable 3D acceleration
   - **Storage:** click the empty optical drive → choose disk → select the Parrot ISO
   - **Network:** Adapter 1 = NAT for now (we'll change to Internal Network later)

### Step 3 — Install Parrot

1. Start the VM — it boots into the live environment
2. Double-click **Install Parrot** on the desktop
3. Walk through the Calamares installer:
   - Language, region, keyboard
   - Partitions: **Erase disk** (this affects only the virtual disk, not your host)
   - User account: pick a strong password
   - Confirm and install (10–20 minutes)
4. When prompted, **remove the ISO** before reboot:
   - VirtualBox menu → Devices → Optical Drives → Remove disk from virtual drive
5. Reboot into the installed system

## Post-Install Hardening

Before using the VM for any practical:

```bash
# Update everything
sudo parrot-upgrade

# Install Guest Additions for clipboard, shared folders, better display
sudo apt install -y virtualbox-guest-utils virtualbox-guest-x11

# Verify your IP and network
ip a
```

## Take a Snapshot

This is the single most important step.

1. VirtualBox Manager → select the VM → **Snapshots** tab → **Take**
2. Name: `clean-install-updated`
3. Description: `Fresh Parrot 7.2, fully updated, before any practical work`

If anything breaks later (a mis-typed `rm`, a malware sample escaping a folder, a botched config), you revert to this snapshot in 10 seconds instead of reinstalling.

## Verification

Confirm everything works:

```bash
# 1. Check version
cat /etc/os-release

# 2. Check tools are present
which nmap nikto sqlmap hydra metasploit-framework

# 3. Check network
ping -c 3 8.8.8.8
```

If all three succeed, the VM is ready.

## Troubleshooting

| Problem | Cause | Fix |
|---|---|---|
| OVA import fails with "VERR_NOT_SUPPORTED" | OVA version mismatch | Update VirtualBox to 7.x or newer |
| Black screen on first boot | 3D acceleration conflict | VM Settings → Display → disable 3D acceleration |
| VM extremely slow | Not enough RAM, or VT-x disabled | Increase RAM to 4 GB+, verify VT-x in BIOS |
| Mouse trapped inside VM | Guest Additions not installed | Install `virtualbox-guest-utils virtualbox-guest-x11` |
| Resolution stuck at 800×600 | Guest Additions missing | Same fix as above, then reboot the VM |
| Can't paste between host and guest | Clipboard sharing off | VM Settings → General → Advanced → Shared Clipboard = Bidirectional |
| Kernel panic during ISO boot | UEFI/BIOS firmware mismatch | VM Settings → System → toggle Enable EFI |

## Screenshots

Put sanitized screenshots in `screenshots/` and reference them here:

```
screenshots/
├── 01-parrot-download-page.png
├── 02-virtualbox-import-appliance.png
├── 03-appliance-settings-review.png
├── 04-first-boot-login.png
├── 05-post-update-terminal.png
└── 06-snapshot-taken.png
```

Blur or crop your real username, hostname, and IP address before committing.

## What I Learned

- The difference between an OVA (pre-built appliance) and an ISO (installer media), and the trade-offs of each
- Why MAC address reinitialization matters when importing the same OVA more than once
- That hash verification is non-negotiable for any pentest distro — a tampered image with root access to your network is a worst-case scenario
- The importance of taking a snapshot **before** any destructive practical, not after

## References

- [Parrot Security Official Download](https://parrotsec.org/download/)
- [Parrot OS Documentation](https://parrotsec.org/docs/)
- [Parrot OS 7.0 Release Notes](https://parrotsec.org/blog/) — KDE Plasma 6 switch
- [VirtualBox OVF/OVA Import Guide](https://www.virtualbox.org/manual/ch01.html#ovf)
- [Debian Stable Release Information](https://www.debian.org/releases/stable/)
