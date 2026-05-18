# Practical 8- Installing Parrot Security OS in VirtualBox using ISO File

## 📖 Description

This practical demonstrates how to install **Parrot Security OS** in **Oracle VirtualBox** using its official ISO file. Parrot Security OS is a Debian-based Linux distribution specifically designed for **penetration testing**, **digital forensics**, **reverse engineering**, and **anonymity** — making it an ideal lab environment for cybersecurity learners.

Installing Parrot inside a virtual machine allows you to safely experiment with hacking tools and techniques **without affecting your host operating system**.

> ⚠️ **Disclaimer:** This practical is intended for **educational purposes only**. Use Parrot Security OS responsibly and in compliance with local laws.

---

## 🎯 Objective

- Learn how to create a new virtual machine in **VirtualBox**.
- Install **Parrot Security OS** using its ISO image.
- Configure essential VM settings (RAM, virtual disk, boot order).
- Install **VirtualBox Guest Additions** for full-screen and clipboard support.

---

## 🛠️ Requirements

| Requirement | Description |
|-------------|-------------|
| Host OS | Windows / macOS / Linux |
| VirtualBox | Latest version of Oracle VirtualBox |
| Parrot Security OS ISO | [Download Link](https://download.parrot.sh/parrot/iso/4.10/Parrot-security-4.10_amd64.iso) |
| RAM | Minimum 2 GB recommended for the VM |
| Disk Space | Minimum 25 GB free space |

---

## 🚀 Step-by-Step Procedure

### Step 1: Download the Parrot Security OS ISO

Download the official ISO file from:
👉 [https://download.parrot.sh/parrot/iso/4.10/Parrot-security-4.10_amd64.iso](https://download.parrot.sh/parrot/iso/4.10/Parrot-security-4.10_amd64.iso)

> 💡 You can also download the latest version from the official Parrot OS website: [https://parrotsec.org/download/](https://parrotsec.org/download/)

---

### Step 2: Create a New Virtual Machine

1. Open **Oracle VirtualBox**.
2. Click on the **New** button to create a new virtual machine.

---

### Step 3: Configure VM Details

A popup will appear asking for VM details:

#### 🔹 Basic Information
- **Name:** `Parrot Security OS`
- **Type:** `Linux`
- **Version:** `Debian (64-bit)`

Click **Next**.

#### 🔹 RAM (Memory Size)
- Allocate RAM based on your system configuration.
- ✅ **Recommended:** `1.5 GB – 2 GB` (1536–2048 MB).

#### 🔹 Hard Disk
- Select **Create a virtual hard disk now** → click **Create**.

#### 🔹 Hard Disk File Type
- Select **VDI (VirtualBox Disk Image)** → click **Next**.

#### 🔹 Storage on Physical Hard Disk
- Select **Dynamically allocated** → click **Next**.

#### 🔹 File Location and Size
- Set the virtual disk size to **at least 25 GB**.
- Click **Create** to finalize the virtual environment.

---

### Step 4: Start the Virtual Machine

- Select the newly created **Parrot Security OS** VM from the VirtualBox list.
- Click the **Start** button to boot the virtual machine.

---

### Step 5: Attach the ISO File

When the VM starts for the first time, it will prompt for a **start-up disk**:

1. Click the **folder icon** to open the Optical Disk Selector.
2. Click **Add** and browse to the location where the Parrot ISO is stored.
3. Select the ISO file and click **Open**.
4. Click **Start** to begin the installation process.

---

### Step 6: Begin the Installation

When the Parrot boot menu appears:

1. Select **Install** → press **Enter**.
2. Select **Install with GTK GUI** → press **Enter**.

#### 🔹 Localization Setup
- **Language:** Select your preferred language → **Continue**.
- **Country:** Select your country → **Continue**.
- **Keyboard Layout:** Choose layout → **Continue**.

> The installer will then load some additional components.

#### 🔹 User Setup
- Set a **root password** → **Continue**.
- Create a **standard user account** (username + password) → **Continue**.

#### 🔹 Disk Partitioning
- Select **Guided – use entire disk** → **Continue**.
- Select **VBOX HARDDISK** → **Continue**.
- Select **Finish partitioning and write changes to disk** → **Continue**.
- When prompted to write changes, select **Yes** → **Continue**.

#### 🔹 Installation
- The installation will start and may take **25 – 30 minutes** depending on your system.

#### 🔹 GRUB Bootloader
- When prompted to install the **GRUB boot loader**, select **Yes** → **Continue**.
- Select **/dev/sda** as the install location → **Continue**.

#### 🔹 Finish Installation
- After successful installation, click **Continue**.
- The installer will remove live packages, unmount disks, and **reboot** automatically.

---

### Step 7: Install VirtualBox Guest Additions

After booting Parrot for the first time, the OS will run in a **small window** because the VirtualBox Guest Additions are not yet installed.

To enable full-screen mode, shared clipboard, and better performance:

1. In the VirtualBox menu bar, click:
   **Devices → Insert Guest Additions CD Image…**

---

### Step 8: Install Guest Additions from Terminal

Open a terminal inside Parrot and become the root user:

```bash
sudo su
```

Navigate to the mounted Guest Additions CD:

```bash
cd /media/cdrom0
ls
```

Copy the Linux Guest Additions installer to the user's home directory:

```bash
cp VBoxLinuxAdditions.run /home/parrot/
```

Switch to the home directory and verify the file:

```bash
cd /home/parrot/
ls
```

Make it executable (if needed) and run the installer:

```bash
chmod +x VBoxLinuxAdditions.run
./VBoxLinuxAdditions.run
```

> 💡 If the installer reports missing kernel headers, install them first:
> ```bash
> sudo apt update
> sudo apt install build-essential dkms linux-headers-$(uname -r) -y
> ```

After the installation completes, **restart the virtual machine**:

```bash
sudo reboot
```

Once rebooted, Parrot will boot into **full-screen mode** and Guest Additions features (auto-resize, shared clipboard, drag-and-drop) will be enabled.

---

## 📸 Verification

After installation, verify your setup:

```bash
uname -a            # Check kernel version
lsb_release -a      # Confirm Parrot OS version
```

You should see output similar to:

```
Distributor ID: Parrot
Description:    Parrot Security 4.10
Release:        4.10
```

---

## 🧠 Key Learnings

- Created and configured a virtual machine using **VirtualBox**.
- Performed a full installation of **Parrot Security OS** from an ISO image.
- Configured **disk partitioning**, **user accounts**, and **GRUB bootloader**.
- Installed **VirtualBox Guest Additions** for an enhanced user experience.

---

## 🛑 Troubleshooting

| Issue | Solution |
|-------|----------|
| VM stuck on boot | Enable **Virtualization (VT-x / AMD-V)** in BIOS |
| Black screen after boot | Increase **Video Memory** to 128 MB in VM settings |
| Guest Additions fails to compile | Install kernel headers: `sudo apt install linux-headers-$(uname -r)` |
| Cannot copy/paste between host & VM | Enable **Shared Clipboard → Bidirectional** in VM settings |
| Low display resolution | Reinstall Guest Additions and reboot the VM |
| `ISO file not detected` | Re-mount the ISO via **Devices → Optical Drives** |

---

## 🔐 Best Practices

- ✅ Always take a **VirtualBox snapshot** after a clean installation.
- ✅ Update Parrot regularly: `sudo apt update && sudo apt upgrade -y`.
- ✅ Allocate at least **2 CPU cores** for smoother performance.
- ✅ Use **NAT or Host-Only** network adapters for safe practice environments.
- ❌ Avoid using your Parrot VM for personal browsing or sensitive work.

---

## 📌 References

- [Parrot Security OS Official Website](https://parrotsec.org/)
- [Parrot ISO Download Mirror](https://download.parrot.sh/parrot/iso/)
- [Oracle VirtualBox Downloads](https://www.virtualbox.org/wiki/Downloads)
- [VirtualBox Guest Additions Documentation](https://www.virtualbox.org/manual/ch04.html)

---

## 👨‍💻 Author

**Practical performed as part of the Ethical Hacking / Cybersecurity Lab.**

> Maintained as a reference for educational and learning purposes.

---

## 📝 License

This project is released under the [MIT License](LICENSE) — feel free to use and modify for educational purposes.
