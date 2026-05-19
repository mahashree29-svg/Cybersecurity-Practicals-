# Practical 2: Network Scanning with Netdiscover

## Description

`netdiscover` is a terminal-based active/passive network scanner that uses the **ARP protocol** to discover live hosts on a local network. Because it relies on ARP, it works only within the **local broadcast domain** (same subnet / LAN segment).

> ⚠️ **Known limitation:** If multiple users run `netdiscover` on the same network at the same time, ARP responses can collide and produce inaccurate or incomplete results.

---

## Prerequisites

- Parrot Linux (or any Debian-based distribution)
- `netdiscover` installed on the system
- Root / `sudo` privileges (ARP scanning requires raw sockets)

Install it (if not already present):

```bash
sudo apt update
sudo apt install netdiscover -y
```

Identify your active network interface first:

```bash
ip a
```

Typical interface names: `eth0`, `wlan0`, `enp0s3`.

---

## Step 1: Run Netdiscover

In the Parrot Linux terminal, type the following command:

```bash
sudo netdiscover -i <interface>
```

### Example

```bash
sudo netdiscover -i eth0
```

This launches a live, full-screen view that continuously updates as hosts are discovered.

Press **`Q`** at any time to stop the scan and exit.

---

## Useful Options

| Option         | Description                                              |
|----------------|----------------------------------------------------------|
| `-i <iface>`   | Specify the network interface to use                     |
| `-r <range>`   | Scan a specific range, e.g. `-r 192.168.1.0/24`          |
| `-l <file>`    | Read a list of ranges from a file                        |
| `-p`           | Passive mode (only sniff ARP traffic, do not send)       |
| `-f`           | Fast mode (scans `.0`, `.1`, `.100`, `.254` of each subnet) |
| `-N`           | Do not print header                                      |
| `-P`           | Print results in plain text and exit (useful for piping) |

### Example — Scan a specific range

```bash
sudo netdiscover -i eth0 -r 192.168.1.0/24
```

### Example — Passive mode (no packets sent)

```bash
sudo netdiscover -i eth0 -p
```

---

## Sample Output

```
 Currently scanning: 192.168.1.0/24   |   Screen View: Unique Hosts

 IP            At MAC Address      Count   Vendor
 -----------------------------------------------------------------
 192.168.1.1   00:1a:2b:3c:4d:5e     1     TP-Link Technologies
 192.168.1.5   aa:bb:cc:dd:ee:ff     1     Apple, Inc.
 192.168.1.12  11:22:33:44:55:66     1     Dell Inc.
```

Each row shows the **live IP**, its **MAC address**, the **packet count**, and the **vendor** (resolved from the MAC OUI).

---

## Saving Results

`netdiscover` does not have a built-in export flag, so redirect stdout to capture results:

```bash
sudo netdiscover -i eth0 -P > netdiscover_results.txt
```

The `-P` flag prints results in plain text and exits, making the file clean and easy to import into other tools.

---

## Troubleshooting

- **`command not found`** → install it with `sudo apt install netdiscover -y`.
- **No hosts discovered** → confirm the interface is correct with `ip a` and that it is connected to the LAN.
- **Permission denied** → run with `sudo`; ARP scanning needs raw socket privileges.
- **Wireless interface not working** → some Wi-Fi drivers/cards do not support raw ARP injection; try wired Ethernet.

---

## Legal & Ethical Note

> ⚠️ **Only scan networks you own or have written authorization to scan.** Unauthorized scanning of third-party networks may violate computer-misuse laws in your jurisdiction.

---

## References

- Netdiscover man page: `man netdiscover`
- Project page: https://github.com/netdiscover-scanner/netdiscover
- Hacker School: https://www.hackerschool.in
