# Practical 3: Ping Sweeping with Nmap

## Description

**Nmap** (Network Mapper) is an open-source scanning tool that performs host and service discovery on networks of any size and returns results very quickly. Nmap can scan **both private and public** IP ranges and supports multiple discovery techniques such as ICMP, TCP SYN/ACK, UDP, and ARP.

A **ping sweep** is the process of sending probes to a range of IP addresses to determine which hosts are alive. In Nmap, the `-sn` flag performs host discovery **without** running a port scan.

Results can also be saved in several file formats for reporting or for use by other tools.

---

## Prerequisites

- Parrot Linux (or any Linux distribution)
- `nmap` installed on the system

Install it (if not already present):

```bash
sudo apt update
sudo apt install nmap -y
```

Verify the installation:

```bash
nmap --version
```

---

## Step 1: Perform a Ping Sweep

In the Parrot Linux terminal, type the following command:

```bash
nmap -sn 192.168.1.1/24
```

### Command Breakdown

| Part              | Meaning                                                 |
|-------------------|---------------------------------------------------------|
| `nmap`            | The Network Mapper utility                              |
| `-sn`             | Host discovery only (no port scan) — formerly `-sP`     |
| `192.168.1.1/24`  | CIDR range covering `192.168.1.0` – `192.168.1.255`     |

> 💡 On the **local subnet** Nmap automatically uses **ARP** for discovery (most accurate). For **remote** networks it falls back to ICMP echo, TCP SYN to port 443, TCP ACK to port 80, and ICMP timestamp.

> ℹ️ Running Nmap with `sudo` is recommended — it enables ARP scans on local networks and raw-packet probes for remote hosts.

---

## Other Ways to Specify Targets

| Target               | Meaning                       |
|----------------------|-------------------------------|
| `192.168.1.1`        | Single host                   |
| `192.168.1.1-50`     | Range (1 to 50)               |
| `192.168.1.0/24`     | Entire /24 subnet             |
| `scanme.nmap.org`    | Hostname                      |
| `-iL targets.txt`    | List of targets from a file   |
| `--exclude 192.168.1.1` | Exclude specific host(s)   |

---

## Sample Output

```
Starting Nmap 7.94 ( https://nmap.org )
Nmap scan report for 192.168.1.1
Host is up (0.0021s latency).
MAC Address: 00:1A:2B:3C:4D:5E (TP-Link)
Nmap scan report for 192.168.1.5
Host is up (0.0015s latency).
MAC Address: AA:BB:CC:DD:EE:FF (Apple)
Nmap scan report for 192.168.1.12
Host is up (0.0030s latency).
Nmap done: 256 IP addresses (3 hosts up) scanned in 4.21 seconds
```

---

## Saving Results to Different File Formats

Nmap supports several output formats — useful for reports or for feeding results into other tools.

| Flag               | Format                    | Use Case                  |
|--------------------|---------------------------|---------------------------|
| `-oN file.txt`     | Normal (human-readable)   | Reading / reporting       |
| `-oG file.gnmap`   | Greppable                 | `grep` / `awk` parsing    |
| `-oX file.xml`     | XML                       | Importing into other tools (e.g. Metasploit, Nessus) |
| `-oA basename`     | All three formats at once | Comprehensive logging     |

### Examples

Save to a normal text file:

```bash
nmap -sn 192.168.1.0/24 -oN pingsweep.txt
```

Save to all three formats at once:

```bash
nmap -sn 192.168.1.0/24 -oA pingsweep_results
```

This produces:
- `pingsweep_results.nmap`
- `pingsweep_results.gnmap`
- `pingsweep_results.xml`

---

## Useful Variants

| Command                                         | Purpose                                            |
|-------------------------------------------------|----------------------------------------------------|
| `nmap -sn -PE 192.168.1.0/24`                   | Force ICMP echo (ping) only                        |
| `nmap -sn -PS22,80,443 192.168.1.0/24`          | TCP SYN ping on common ports                       |
| `nmap -sn -PA80 192.168.1.0/24`                 | TCP ACK ping (useful through stateful firewalls)   |
| `nmap -sn -n 192.168.1.0/24`                    | Skip reverse-DNS resolution (faster)               |
| `sudo nmap -sn -PR 192.168.1.0/24`              | Force ARP ping (LAN only)                          |

---

## Troubleshooting

- **`command not found`** → install with `sudo apt install nmap -y`.
- **All hosts shown as down** → some networks block ICMP; try `-PS80,443` or `-PA80`.
- **No MAC addresses in output** → MAC info is only resolved on the **local subnet**; run with `sudo`.
- **Slow scans** → add `-n` to skip DNS, or `-T4` to increase timing aggressiveness.

---

## Legal & Ethical Note

> ⚠️ **Only scan networks and hosts you own or have written authorization to scan.** Unauthorized scanning may violate computer-misuse laws in your jurisdiction. For practice, use `scanme.nmap.org` — it is explicitly provided for testing.

---

## References

- Nmap official documentation: https://nmap.org/book/man.html
- Host-discovery reference: https://nmap.org/book/host-discovery.html
- Output format reference: https://nmap.org/book/man-output.html
- Hacker School: https://www.hackerschool.in
