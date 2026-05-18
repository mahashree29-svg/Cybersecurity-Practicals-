# Practical 1: Network Scanning with Angry IP Scanner

## Overview

Angry IP Scanner is a fast, lightweight, cross-platform graphical tool used to scan IP addresses and ports. It can scan ranges of **private** or **public** IPs using different protocols, perform port scanning, and export results to a file for further analysis or reporting.

In this practical, we will:
- Install Angry IP Scanner on **Parrot Linux**
- Perform network scanning on a range of IPs
- Discover live devices and open ports
- Export the scan results to a text file for use in other VA / port-scanning tools

---

## Prerequisites

- Parrot Linux (or any Debian-based Linux distribution)
- Internet connectivity
- Root or `sudo` privileges
- Java Runtime Environment (usually bundled with the `.deb` package)

---

## Step 1: Download Angry IP Scanner

Visit the official download page:

🔗 https://angryip.org/download/

For Parrot Linux, download the appropriate **`.deb`** package based on your system architecture (32-bit or 64-bit).

## Step 2: Save the File

When the browser prompts, choose **Save File**. By default it will be stored in the `Downloads` directory of the current user.

## Step 3: Navigate to the Downloads Directory

Open a terminal and change into the directory where the file was saved:

```bash
cd /root/Downloads/
```

Verify the file is present:

```bash
ls -l ipscan*.deb
```

## Step 4: Install the Package

Install the `.deb` package using `dpkg`:

```bash
sudo dpkg -i ipscan_*_amd64.deb
```

If you encounter missing dependencies, fix them with:

```bash
sudo apt-get install -f
```

> Replace the filename with the exact one you downloaded (e.g. `ipscan_3.9.1_amd64.deb`).

## Step 5: Launch Angry IP Scanner

After installation, search for **Angry IP Scanner** in the installed applications menu and launch it.

Alternatively, start it from the terminal:

```bash
ipscan
```

---

## Performing a Scan

1. In the main window, enter the **IP range** to scan:
   - Start IP (e.g. `192.168.1.1`)
   - End IP (e.g. `192.168.1.254`)
2. (Optional) Configure **Tools → Preferences** to enable additional fetchers such as:
   - Ping
   - Hostname
   - Ports (specify port list, e.g. `21,22,23,80,443,3389`)
   - MAC address / vendor
3. Click **Start** to begin scanning.
4. Wait for the scan to complete. Live hosts are shown in **blue**, dead hosts in **red**.

---

## Exporting Scan Results

After the scan completes:

1. Go to **Scan → Export**.
2. Choose the format (TXT, CSV, XML, or IP-Port list).
3. Save the file (e.g. `scan_results.txt`).

The exported file can be fed into other vulnerability-assessment or port-scanning tools such as **Nmap**, **Nessus**, or **OpenVAS** for deeper analysis.

---

## Common Scan Targets

| Type      | Example Range                   | Use Case                          |
|-----------|---------------------------------|-----------------------------------|
| Private   | `192.168.1.1 – 192.168.1.254`   | Local LAN device discovery        |
| Private   | `10.0.0.1 – 10.0.0.254`         | Corporate / VPN networks          |
| Public    | Authorized IP range only        | External asset discovery (with permission) |

> ⚠️ **Always obtain written authorization before scanning networks or IP addresses that you do not own.** Unauthorized scanning may violate computer-misuse laws.

---

## Troubleshooting

- **`dpkg` reports dependency errors** → run `sudo apt-get install -f`.
- **Application does not launch** → ensure Java is installed: `sudo apt install default-jre`.
- **All hosts show as dead** → many hosts block ICMP; switch fetcher to **TCP ping** or add a common port (e.g. 80, 443) under Preferences.
- **No MAC addresses shown** → MAC fetching only works on the **local subnet**.

---

## References

- Official site: https://angryip.org/
- Documentation: https://github.com/angryip/ipscan/wiki
- Hacker School: https://www.hackerschool.in
