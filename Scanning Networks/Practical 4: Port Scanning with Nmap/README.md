# Practical 4: Port Scanning with Nmap

<p align="center">
  <img src="https://img.shields.io/badge/Tool-Nmap-blue?style=for-the-badge&logo=gnubash&logoColor=white" alt="Nmap">
  <img src="https://img.shields.io/badge/OS-Parrot%20%7C%20Kali-green?style=for-the-badge&logo=linux&logoColor=white" alt="Linux">
  <img src="https://img.shields.io/badge/Category-Cybersecurity-red?style=for-the-badge&logo=hackthebox&logoColor=white" alt="Cybersecurity">
  <img src="https://img.shields.io/badge/Level-Beginner-yellow?style=for-the-badge" alt="Level">
</p>

## 📌 Description

**Nmap** (Network Mapper) is a powerful, multi-purpose open-source tool used for network discovery and security auditing. In this practical, we explore the different options Nmap provides to perform **port scanning** on target IPs and customize scans for various scenarios.

With Nmap you can find out:

- 🔓 Which ports are **open** on a target
- 🛠️ What **services** are running on those ports and their **versions**
- 💻 The target's **operating system** details
- 🛡️ **Firewall** presence and filtering rules
- 🌐 The **network path** to the target

---

## 📋 Table of Contents

- [Prerequisites](#-prerequisites)
- [Scans Covered](#-scans-covered)
  - [Scan 1: Regular Scan (SYN Stealth / Half-Open)](#scan-1-regular-scan-syn-stealth--half-open)
  - [Scan 2: TCP Connect Scan (Full Connect)](#scan-2-tcp-connect-scan-full-connect)
  - [Scan 3: Service / Version Detection](#scan-3-service--version-detection)
  - [Scan 4: OS Detection](#scan-4-os-detection)
  - [Scan 5: FIN Scan](#scan-5-fin-scan)
  - [Scan 6: XMAS Scan](#scan-6-xmas-scan)
  - [Scan 7: NULL Scan](#scan-7-null-scan)
  - [Scan 8: Aggressive Scan](#scan-8-aggressive-scan)
  - [Scan 9: UDP Port Scan](#scan-9-udp-port-scan)
  - [Scan 10: Custom Port Scan](#scan-10-custom-port-scan)
  - [Scan 11: Traceroute with Nmap](#scan-11-traceroute-with-nmap)
- [Saving Output](#-saving-output)
- [Quick Reference](#-quick-reference)
- [Legal & Ethical Note](#%EF%B8%8F-legal--ethical-note)
- [References](#-references)

---

## ⚙️ Prerequisites

- Linux distribution (Parrot, Kali, Ubuntu, etc.)
- `nmap` installed
- Root / `sudo` privileges (required for SYN, OS, UDP and stealth scans)

Install Nmap:

```bash
sudo apt update
sudo apt install nmap -y
```

Verify the installation:

```bash
nmap --version
```

> 💡 **Note:** Even when you provide a **domain name** to Nmap, it does not scan the *website* — it scans the **server (computer) hosting** that website.

---

## 🎯 Scans Covered

### Scan 1: Regular Scan (SYN Stealth / Half-Open)

The default Nmap scan when run as root. Sends a SYN packet, listens for SYN/ACK, then sends RST — never completing the TCP handshake. Hence "half-open" and harder to log.

**Syntax**

```bash
nmap <target>
nmap -sS <target>
```

**Examples**

```bash
sudo nmap 192.168.0.137
sudo nmap -sS example.com
```

---

### Scan 2: TCP Connect Scan (Full Connect)

Completes the full TCP three-way handshake. Slower and noisier than `-sS`, but does not require root.

**Syntax**

```bash
nmap -sT <target>
```

**Examples**

```bash
nmap -sT example.com
nmap -sT 192.168.0.137
```

> ⚠️ If you get an error like *"host may be down or disabled ICMP"*, add `-Pn` to skip host discovery:
>
> ```bash
> nmap -sT -Pn example.com
> ```

---

### Scan 3: Service / Version Detection

Probes open ports to determine what application (and version) is running on them.

**Syntax**

```bash
nmap -sV <target>
```

**Examples**

```bash
nmap -sV example.com
nmap -sV 192.168.0.137
```

---

### Scan 4: OS Detection

Uses TCP/IP stack fingerprinting to guess the target's operating system.

**Syntax**

```bash
sudo nmap -O <target>
```

**Examples**

```bash
sudo nmap -O example.com
sudo nmap -O 192.168.0.137
```

---

### Scan 5: FIN Scan

Sends packets with only the **FIN** flag set. Closed ports respond with RST; open ports drop the packet. Useful for evading certain firewalls.

**Syntax**

```bash
sudo nmap -sF <target>
```

**Examples**

```bash
sudo nmap -sF example.com
sudo nmap -sF 192.168.0.137 -v
```

---

### Scan 6: XMAS Scan

Sends packets with **FIN, PSH, URG** flags set (lit up like a Christmas tree 🎄). Same logic as FIN scan but slightly different signature.

**Syntax**

```bash
sudo nmap -sX <target>
```

**Examples**

```bash
sudo nmap -sX example.com
sudo nmap -sX 192.168.0.137 -v
```

---

### Scan 7: NULL Scan

Sends packets with **no flags** set. Open ports drop them; closed ports send RST.

**Syntax**

```bash
sudo nmap -sN <target>
```

**Examples**

```bash
sudo nmap -sN example.com
sudo nmap -sN 192.168.0.137 -v
```

---

### Scan 8: Aggressive Scan

Combines OS detection, version detection, script scanning, and traceroute — the most comprehensive single-flag scan.

**Syntax**

```bash
sudo nmap -A <target>
```

**Examples**

```bash
sudo nmap -A example.com
sudo nmap -A 192.168.0.137 -v
```

> 💡 Add `-v` to **any** Nmap command to see detailed (verbose) output.

---

### Scan 9: UDP Port Scan

Scans UDP ports. UDP scans are slower and less reliable than TCP because UDP is connectionless.

**Syntax**

```bash
sudo nmap -sU <target>
```

**Examples**

```bash
sudo nmap -sU example.com
sudo nmap -sU 192.168.0.137
```

---

### Scan 10: Custom Port Scan

Scan specific ports, ranges, or comma-separated lists instead of Nmap's default 1000 ports.

**Syntax**

```bash
nmap -p <ports> <target>
```

**Examples**

```bash
nmap -p 80 example.com               # Single port
nmap 192.168.0.137 -p 80-85          # Port range
nmap 49.204.90.43 -p 80,81,85,21,443 # Specific ports
nmap -p- 192.168.0.137               # All 65535 ports
```

---

### Scan 11: Traceroute with Nmap

Maps the network path (hops) between you and the target.

**Syntax**

```bash
sudo nmap --traceroute <target>
```

**Examples**

```bash
sudo nmap --traceroute example.com
sudo nmap --traceroute 192.168.0.137 -v
```

---

## 💾 Saving Output

Nmap supports four output formats:

| Flag             | Format                  | Use Case                              |
|------------------|-------------------------|---------------------------------------|
| `-oN file.txt`   | Normal (human-readable) | Reading / reporting                   |
| `-oG file.gnmap` | Greppable               | `grep` / `awk` parsing                |
| `-oX file.xml`   | XML                     | Import into Metasploit, Nessus, etc.  |
| `-oA basename`   | All three at once       | Comprehensive logging                 |

**Example**

```bash
nmap -A 192.168.0.137 -oA full_scan_results
```

---

## 📝 Quick Reference

| Flag           | Purpose                              |
|----------------|--------------------------------------|
| `-sS`          | SYN stealth scan (default for root)  |
| `-sT`          | TCP full-connect scan                |
| `-sU`          | UDP scan                             |
| `-sV`          | Service / version detection          |
| `-O`           | OS detection                         |
| `-A`           | Aggressive (OS + version + script + traceroute) |
| `-sF`          | FIN scan                             |
| `-sX`          | XMAS scan                            |
| `-sN`          | NULL scan                            |
| `-p`           | Specify ports                        |
| `-Pn`          | Skip host discovery (ping)           |
| `-v`           | Verbose output                       |
| `--traceroute` | Show network path to target          |
| `-oA`          | Save output in all formats           |

---

## ⚖️ Legal & Ethical Note

> ⚠️ **Only scan systems you own or have written authorization to scan.** Unauthorized scanning may violate computer-misuse laws in your jurisdiction.
>
> For safe practice, use Nmap's public sandbox host: **`scanme.nmap.org`**.

---

## 🔗 References

- 📖 Nmap Official Documentation: <https://nmap.org/book/man.html>
- 🎯 Port Scanning Techniques: <https://nmap.org/book/man-port-scanning-techniques.html>
- 🧠 Nmap Cheat Sheet: <https://nmap.org/book/man-briefoptions.html>

---

<p align="center">
  Made with 💻 for cybersecurity learners
</p>
