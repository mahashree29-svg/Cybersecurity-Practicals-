# Practical 1 — Installing VirtualBox on Windows

## Objective

Install Oracle VirtualBox on a Windows host so it can run multiple guest operating systems (Parrot Security, Metasploitable2, Windows VMs) in isolated virtual environments. VirtualBox is the foundation for every other practical in this series — without a working hypervisor, there is no lab.

## Why VirtualBox?

VirtualBox is a Type-2 hypervisor: it runs on top of the host OS rather than directly on hardware. For a learning lab on a single laptop, this is exactly what you want:

- **Free and open-source** (GPLv3 base, proprietary Extension Pack for personal use)
- **Cross-platform** — same workflow on Windows, macOS, Linux
- **Snapshots** — revert a VM to a clean state in seconds
- **Internal Networks** — fully isolated virtual networks that cannot reach the host or the internet

The trade-off vs. Type-1 hypervisors (ESXi, Proxmox) is performance, but for beginner labs the difference is irrelevant.

## Prerequisites

| Requirement | Details |
|---|---|
| OS | Windows 10 or 11, 64-bit |
| RAM | 8 GB minimum, 16 GB recommended |
| Disk | 50 GB free |
| CPU virtualization | VT-x (Intel) or AMD-V (AMD) **enabled in BIOS/UEFI** |
| Admin rights | Required for installation |

**Check virtualization is enabled before starting:**
Open Task Manager → Performance tab → CPU. Look for "Virtualization: Enabled". If it says Disabled, reboot into BIOS/UEFI and enable Intel VT-x or AMD-V (the exact menu name varies by manufacturer). Without this, VirtualBox will install but VMs will run unusably slow.

## Installation Steps

### Step 1 — Download VirtualBox

1. Go to https://www.virtualbox.org/wiki/Downloads
2. Under **VirtualBox X.X.X platform packages**, click **Windows hosts**
3. Save the `.exe` installer (around 110 MB)

### Step 2 — Run the Installer

1. Right-click the downloaded `.exe` → **Run as administrator**
2. Welcome screen → **Next**
3. Custom Setup screen → leave default features selected → **Next**
4. Shortcuts options → leave defaults → **Next**

### Step 3 — Accept the Network Interface Warning

The installer will warn that it needs to temporarily reset your network interfaces while it installs the virtual network drivers.

- Close any active downloads, video calls, or remote sessions first
- Click **Yes** to continue
- Your internet will drop for 5–10 seconds and then return

### Step 4 — Install Device Drivers

Windows will pop up a driver installation prompt for **Oracle Corporation Universal Serial Bus**.

- Check **Always trust software from "Oracle Corporation"**
- Click **Install**

This driver is what lets you connect USB devices to your VMs later.

### Step 5 — Install the Extension Pack

The base VirtualBox install is enough to run VMs, but the **Extension Pack** adds:

- USB 2.0 and USB 3.0 support
- VirtualBox Remote Desktop Protocol (VRDP)
- Host webcam passthrough
- PXE boot for Intel network cards
- Disk image encryption

**Steps:**

1. Back at https://www.virtualbox.org/wiki/Downloads
2. Scroll to **Oracle VirtualBox Extension Pack** → click **All supported platforms**
3. Double-click the downloaded `.vbox-extpack` file
4. VirtualBox opens automatically and asks to install → click **Install**
5. Scroll through the license → click **I Agree**

> ⚠️ The Extension Pack is licensed under the **VirtualBox Personal Use and Evaluation License (PUEL)** — free for personal, educational, and evaluation use only. Commercial use requires a paid license from Oracle.

## Verification

Confirm the install worked before moving on.

1. Open **Oracle VM VirtualBox Manager** from the Start menu
2. Go to **Help → About VirtualBox** — confirm version matches what you downloaded
3. Go to **File → Tools → Extension Pack Manager** — the Extension Pack should be listed
4. Go to **File → Tools → Network Manager** — confirm you can see `VirtualBox Host-Only Ethernet Adapter`

If all four checks pass, VirtualBox is ready.

## Troubleshooting

| Problem | Cause | Fix |
|---|---|---|
| Installer fails with "VT-x is not available" | Virtualization disabled in BIOS | Reboot into BIOS, enable Intel VT-x / AMD-V |
| Conflict with Hyper-V or WSL2 | Windows Hyper-V running | Run `bcdedit /set hypervisorlaunchtype off` in admin cmd, reboot. Note: this disables Docker Desktop's WSL2 backend |
| Network drops permanently after install | Network bridge driver issue | Device Manager → Network adapters → reinstall affected adapter |
| VirtualBox Manager won't open | Missing Visual C++ Redistributable | Install Microsoft Visual C++ 2019+ Redistributable |
| Extension Pack install greyed out | Not running as admin | Right-click VirtualBox shortcut → Run as administrator |

## Screenshots

Place sanitized screenshots in `screenshots/` and reference them here:

```
screenshots/
├── 01-download-page.png
├── 02-installer-welcome.png
├── 03-network-warning.png
├── 04-extension-pack-install.png
└── 05-verification-about.png
```

## What I Learned

- The difference between Type-1 and Type-2 hypervisors and when each is appropriate
- Why CPU virtualization extensions (VT-x / AMD-V) must be enabled at the BIOS level — the OS cannot turn them on
- The licensing distinction between the GPL-licensed VirtualBox base and the proprietary Extension Pack (PUEL)
- That Hyper-V and VirtualBox compete for the same low-level hardware access on Windows, which is why running both simultaneously causes issues



## References

- [VirtualBox Official Downloads](https://www.virtualbox.org/wiki/Downloads)
- [VirtualBox User Manual](https://www.virtualbox.org/manual/)
- [VirtualBox PUEL License](https://www.virtualbox.org/wiki/VirtualBox_PUEL)
- [Intel VT-x Overview](https://www.intel.com/content/www/us/en/virtualization/virtualization-technology/intel-virtualization-technology.html)
