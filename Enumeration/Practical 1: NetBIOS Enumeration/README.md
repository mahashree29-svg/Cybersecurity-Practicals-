# Practical 1: NetBIOS Enumeration

<p align="center">
  <img src="https://img.shields.io/badge/Topic-NetBIOS%20Enumeration-blue?style=for-the-badge&logo=windows&logoColor=white" alt="NetBIOS">
  <img src="https://img.shields.io/badge/OS-Windows%20%7C%20Parrot%20%7C%20Kali-green?style=for-the-badge&logo=linux&logoColor=white" alt="OS">
  <img src="https://img.shields.io/badge/Category-Cybersecurity-red?style=for-the-badge&logo=hackthebox&logoColor=white" alt="Cybersecurity">
  <img src="https://img.shields.io/badge/Level-Beginner-yellow?style=for-the-badge" alt="Level">
</p>

## 📌 Description

**NetBIOS** (Network Basic Input/Output System) is a legacy protocol that allows applications on different computers to communicate over a Local Area Network (LAN). Even today, it is widely used by **Windows file and printer sharing** services.

In this practical we **enumerate NetBIOS information** of file/service-sharing devices connected to the target system. NetBIOS enumeration can reveal:

- 💻 Computer (NetBIOS) names
- 👥 Workgroup / Domain names
- 👤 Logged-in users
- 📁 Shared folders & printers
- 🌐 MAC addresses

> ℹ️ NetBIOS Name Service typically runs on **UDP port 137**, Datagram Service on **UDP 138**, and Session Service on **TCP 139**.

---

## 📋 Table of Contents

- [Prerequisites](#-prerequisites)
- [Steps](#-steps)
  - [Step 1: Enumerate NetBIOS Names on Windows](#step-1-enumerate-netbios-names-on-windows)
  - [Step 2: View Cached NetBIOS Information](#step-2-view-cached-netbios-information)
  - [Step 3: Enumerate from Parrot Linux with nbtscan](#step-3-enumerate-from-parrot-linux-with-nbtscan)
- [Understanding NetBIOS Suffixes](#-understanding-netbios-suffixes)
- [Common nbtstat & nbtscan Options](#-common-nbtstat--nbtscan-options)
- [Legal & Ethical Note](#%EF%B8%8F-legal--ethical-note)
- [References](#-references)

---

## ⚙️ Prerequisites

| Operating System | Tool Required |
|------------------|---------------|
| Windows          | `nbtstat` (built-in) |
| Parrot / Kali Linux | `nbtscan` |

Install `nbtscan` on Linux (if not already present):

```bash
sudo apt update
sudo apt install nbtscan -y
```

Verify the installation:

```bash
nbtscan -h
```

---

## 🎯 Steps

### Step 1: Enumerate NetBIOS Names on Windows

Open **Command Prompt** (`cmd`) and execute the following command. It displays the NetBIOS names of devices connected to the target IP.

**Syntax**

```cmd
nbtstat -A <target IP>
```

**Example**

```cmd
nbtstat -A 192.168.0.137
```

**Sample Output**

```
   NetBIOS Remote Machine Name Table

   Name               Type         Status
---------------------------------------------
   DESKTOP-ABC123  <00>  UNIQUE      Registered
   WORKGROUP       <00>  GROUP       Registered
   DESKTOP-ABC123  <20>  UNIQUE      Registered
   WORKGROUP       <1E>  GROUP       Registered

   MAC Address = 00-1A-2B-3C-4D-5E
```

> 💡 Use **uppercase `-A`** to query by IP address, and **lowercase `-a`** to query by NetBIOS name.

---

### Step 2: View Cached NetBIOS Information

The following command displays the local NetBIOS name cache, including resolved names and their IP addresses.

**Syntax**

```cmd
nbtstat -c
```

**Sample Output**

```
   Local Area Connection:
   Node IpAddress: [192.168.0.10]   Scope Id: []

      NetBIOS Remote Cache Name Table

      Name              Type       Host Address   Life [sec]
   -------------------------------------------------------------
   DESKTOP-ABC123 <20>  UNIQUE     192.168.0.137     420
   FILESERVER     <20>  UNIQUE     192.168.0.50      540
```

---

### Step 3: Enumerate from Parrot Linux with nbtscan

`nbtscan` scans a range of IP addresses and prints NetBIOS information for each responding host. It is much faster than `nbtstat` because it can scan **multiple hosts at once**.

**Syntax**

```bash
nbtscan <target | range>
```

**Examples**

```bash
nbtscan 192.168.0.137              # Single IP
nbtscan 192.168.0.0/24             # Entire /24 subnet
nbtscan 192.168.0.1-254            # IP range
```

**Sample Output**

```
Doing NBT name scan for addresses from 192.168.0.0/24

IP address       NetBIOS Name     Server    User             MAC address
------------------------------------------------------------------------
192.168.0.137    DESKTOP-ABC123   <server>  <unknown>        00-1a-2b-3c-4d-5e
192.168.0.50     FILESERVER       <server>  <unknown>        aa-bb-cc-dd-ee-ff
192.168.0.10     PRINTER01        <server>  <unknown>        11-22-33-44-55-66
```

**Verbose mode**

```bash
nbtscan -v 192.168.0.0/24
```

**Hash-style (one host per line, for parsing)**

```bash
nbtscan -h 192.168.0.0/24
```

---

## 🔢 Understanding NetBIOS Suffixes

NetBIOS names are 16 characters long — 15 for the name and 1 hex byte (the "suffix") that identifies the resource type. Common suffixes:

| Suffix  | Type   | Meaning                              |
|---------|--------|--------------------------------------|
| `<00>`  | UNIQUE | Workstation service (computer name)  |
| `<00>`  | GROUP  | Workgroup / Domain name              |
| `<03>`  | UNIQUE | Messenger service / logged-in user   |
| `<20>`  | UNIQUE | File Server service (file sharing)   |
| `<1B>`  | UNIQUE | Domain Master Browser                |
| `<1C>`  | GROUP  | Domain controllers                   |
| `<1D>`  | UNIQUE | Master Browser                       |
| `<1E>`  | GROUP  | Browser service elections            |

> 🎯 Spotting **`<20>`** is a strong indicator that **file sharing** is enabled on the target — useful in security assessments.

---

## 📝 Common nbtstat & nbtscan Options

### `nbtstat` (Windows)

| Option | Description |
|--------|-------------|
| `-A <IP>`   | List NetBIOS table of remote machine by **IP** |
| `-a <name>` | List NetBIOS table of remote machine by **name** |
| `-c`        | Display local NetBIOS name cache |
| `-n`        | Display local NetBIOS names |
| `-r`        | Resolved names by broadcast & WINS |
| `-R`        | Purge and reload the name cache |
| `-s`        | Show sessions with destination IP |
| `-S`        | Show sessions with destination name |

### `nbtscan` (Linux)

| Option | Description |
|--------|-------------|
| `-v`   | Verbose output |
| `-r`   | Use the local port 137 for scans (often required to receive responses) |
| `-h`   | Print results in hash-style (parseable) |
| `-q`   | Suppress banners and errors |
| `-f <file>` | Read target list from file |

---

## ⚖️ Legal & Ethical Note

> ⚠️ **Only enumerate systems you own or have explicit written authorization to test.** Unauthorized NetBIOS enumeration may violate computer-misuse laws in your jurisdiction.

---

## 🔗 References

- 📖 Microsoft `nbtstat` documentation: <https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/nbtstat>
- 📖 nbtscan manual: `man nbtscan`
- 📚 NetBIOS suffix list (RFC 1001 / 1002): <https://datatracker.ietf.org/doc/html/rfc1001>


---

<p align="center">
  Made with 💻 for cybersecurity learners
</p>
