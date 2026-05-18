# Metasploitable 2 Installation on VirtualBox

This practical covers the installation and basic configuration of **Metasploitable 2**, a deliberately vulnerable Linux virtual machine, inside **Oracle VirtualBox**. It is intended as a follow-up to the earlier practical on installing Parrot Security OS, and serves as a target machine for penetration testing exercises.

> ⚠️ **Disclaimer:** Metasploitable 2 is intentionally vulnerable. **Never** expose it to the public internet. Use it only inside an isolated lab environment for educational purposes.

---

## 📖 About Metasploitable 2

Metasploitable 2 is a vulnerable virtual machine designed for security training, testing exploits, and practicing with the Metasploit Framework. It focuses on **system-level vulnerabilities** that can be exploited to learn the fundamentals of ethical hacking, vulnerability assessment, and post-exploitation techniques.

---

## 🧰 Tools Required

| Tool | Purpose |
|------|---------|
| Oracle VirtualBox | Virtualization platform |
| Metasploitable 2 `.vmdk` | Pre-built vulnerable VM disk image |

**Download link:** [Metasploitable 2 on SourceForge](https://sourceforge.net/projects/metasploitable/files/Metasploitable2/)

---

## 🚀 Installation Steps

### Step 1 — Create a New Virtual Machine

1. Download the Metasploitable 2 archive from the link above and **extract** the `.zip` file.
2. Open **VirtualBox** and click **New** in the top-left corner.
3. Fill in the following details:
   - **Name:** `Metasploitable2`
   - **Type:** `Linux`
   - **Version:** `Other Linux (64-bit)`
4. Click **Continue**.

### Step 2 — Allocate Memory

- Set the **Memory size** to **1024 MB** (1 GB).
- Click **Continue**.

### Step 3 — Attach the Existing Virtual Hard Disk

1. Choose **Use an existing virtual hard disk file**.
2. Browse to the extracted `Metasploitable2` directory and select the `Metasploitable.vmdk` file.
3. Click **Create** to finish.

### Step 4 — Configure Network Settings

Before starting the VM, configure the network adapter so other machines (e.g., your Parrot Security VM) can reach it:

| Setting | Value |
|---------|-------|
| Attached to | `Bridged Adapter` |
| Promiscuous Mode | `Allow All` |

> 💡 If you want a more isolated lab, you can alternatively use a **Host-Only Adapter** — this keeps the vulnerable machine off your local network entirely.

### Step 5 — Boot the Machine

1. Select **Metasploitable2** from the VirtualBox menu.
2. Click **Start**.
3. Log in with the default credentials:

```
Login:    msfadmin
Password: msfadmin
```

---

## 🔍 Verifying the Setup

Once logged in, check the VM's IP address with:

```bash
ifconfig
```

From your attacker machine (e.g., Parrot Security), confirm connectivity:

```bash
ping <metasploitable-ip>
```

A quick port scan should reveal the many open services available for practice:

```bash
nmap -sV <metasploitable-ip>
```

---

## 🧪 Common Services to Explore

Metasploitable 2 exposes a wide range of vulnerable services, including:

- FTP (vsftpd 2.3.4 — backdoor)
- SSH (OpenSSH 4.7)
- Telnet
- SMTP
- HTTP (Apache + TWiki, phpMyAdmin, DVWA, Mutillidae)
- SMB (Samba)
- MySQL, PostgreSQL
- Distccd, Java RMI, IRC (UnrealIRCd backdoor)

These services will be used in later practicals covering Metasploit Framework exploitation.

---

## 🛡️ Safety Notes

- Keep the VM on a **Bridged / Host-Only** network — never port-forward it to the internet.
- Take a **snapshot** before practicing exploits so you can roll back to a clean state.
- Always shut down the VM when not in use.

---

## 📚 References

- [Metasploitable 2 — SourceForge](https://sourceforge.net/projects/metasploitable/files/Metasploitable2/)
- [Rapid7 — Metasploitable 2 Exploitability Guide](https://docs.rapid7.com/metasploit/metasploitable-2/)
- [VirtualBox Documentation](https://www.virtualbox.org/wiki/Documentation)

---

## 📝 License

This repository documents an educational practical for learning purposes only. The author assumes no responsibility for misuse of the tools or techniques described.
