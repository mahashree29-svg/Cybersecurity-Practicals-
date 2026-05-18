# Practical 7: Subdomain Enumeration 

## 📖 Description

**Sublist3r** is a popular **Python-based open-source tool** designed for the **enumeration of subdomains** of a target website. It leverages multiple search engines and public services to gather subdomain data — including:

- 🌐 **Google**, **Bing**, **Yahoo**, **Baidu**, **Ask**
- 🔍 **Netcraft**, **Virustotal**, **ThreatCrowd**
- 📡 **DNSdumpster**, **ReverseDNS**
- 🛠️ Integrated **brute-force** capability using `subbrute`

Subdomain enumeration is a critical step during the **reconnaissance phase** of penetration testing — discovering hidden subdomains often reveals **forgotten servers**, **staging environments**, or **admin panels** that may have weaker security controls than the main website.

> ⚠️ **Disclaimer:** This practical is intended for **educational and authorized testing purposes only**.

---

## 🎯 Objective

- Clone and set up the Sublist3r tool from GitHub.
- Enumerate subdomains of a target domain.
- Understand the importance of subdomain discovery in penetration testing.

---

## 🛠️ Prerequisites

| Tool | Description |
|------|-------------|
| Parrot Security OS / Kali Linux | Linux distribution |
| `git` | To clone the repository |
| `python3` & `pip3` | Required for execution |
| Internet connection | Required |

---

## 🚀 Step-by-Step Procedure

### Step 1: Visit the Sublist3r GitHub Repository

Open your browser and visit:
👉 [https://github.com/aboul3la/Sublist3r](https://github.com/aboul3la/Sublist3r)

Click on the green **`Code`** (or **Clone or Download**) button. Copy the clone URL:

```
https://github.com/aboul3la/Sublist3r.git
```

---

### Step 2: Clone the Repository

Open a terminal and execute:

```bash
git clone https://github.com/aboul3la/Sublist3r.git
```

This will download Sublist3r to your current directory.

---

### Step 3: Navigate Into the Sublist3r Directory

```bash
cd Sublist3r
ls
```

You should see files including:

```
sublist3r.py
requirements.txt
subbrute/
README.md
LICENSE
```

---

### Step 4: Install Required Python Dependencies

```bash
pip3 install -r requirements.txt
```

> 💡 If you encounter a missing module error during execution, install it manually:
> ```bash
> pip3 install dnspython requests argparse
> ```

---

### Step 5: View the Help Menu

```bash
python3 sublist3r.py --help
```

Sample output:

```
       ____        _     _ _     _   _____
      / ___| _   _| |__ | (_)___| |_|___ / _ __
      \___ \| | | | '_ \| | / __| __| |_ \| '__|
       ___) | |_| | |_) | | \__ \ |_ ___) | |
      |____/ \__,_|_.__/|_|_|___/\__|____/|_|

                # Coded By Ahmed Aboul-Ela - @aboul3la

usage: sublist3r.py [-h] -d DOMAIN [-b [BRUTEFORCE]] [-p PORTS] [-v [VERBOSE]]
                    [-t THREADS] [-e ENGINES] [-o OUTPUT] [-n]
```

---

### Step 6: Enumerate Subdomains

#### 🔹 Basic Syntax

```bash
python3 sublist3r.py -d <target-domain>
```

#### 🔹 Example

```bash
python3 sublist3r.py -d hackthissite.org
```

---

### Step 7: Advanced Usage Options

| Flag | Description | Example |
|------|-------------|---------|
| `-d` | Target domain (required) | `-d example.com` |
| `-b` | Enable subbrute brute-force module | `-b` |
| `-p` | Scan specified ports of found subdomains | `-p 80,443` |
| `-v` | Verbose output | `-v` |
| `-t` | Number of threads for brute-force | `-t 50` |
| `-e` | Specify search engines | `-e google,bing` |
| `-o` | Save results to a file | `-o results.txt` |
| `-n` | Disable colors in output | `-n` |

#### 🔹 Example: Full Scan with Brute-force and File Output

```bash
python3 sublist3r.py -d hackthissite.org -b -t 50 -o subdomains.txt
```

#### 🔹 Example: Use Only Specific Search Engines

```bash
python3 sublist3r.py -d hackthissite.org -e google,bing,virustotal
```

#### 🔹 Example: Scan Discovered Subdomains for Open Ports

```bash
python3 sublist3r.py -d hackthissite.org -p 80,443,8080
```

---

## 📋 Sample Output

```
[-] Enumerating subdomains now for hackthissite.org
[-] Searching now in Baidu..
[-] Searching now in Yahoo..
[-] Searching now in Google..
[-] Searching now in Bing..
[-] Searching now in Ask..
[-] Searching now in Netcraft..
[-] Searching now in DNSdumpster..
[-] Searching now in Virustotal..
[-] Searching now in ThreatCrowd..
[-] Searching now in SSL Certificates..
[-] Searching now in PassiveDNS..

[-] Total Unique Subdomains Found: 24

www.hackthissite.org
mail.hackthissite.org
forum.hackthissite.org
ftp.hackthissite.org
api.hackthissite.org
shop.hackthissite.org
...
```

---

## 🧠 Key Learnings

- Cloned and installed a real-world OSINT tool from GitHub.
- Performed **automated subdomain enumeration** using multiple data sources.
- Understood the value of subdomain discovery in expanding the **attack surface**.
- Saved structured results for later analysis.

---

## 🛑 Troubleshooting

| Issue | Solution |
|-------|----------|
| `ModuleNotFoundError: No module named 'dns'` | `pip3 install dnspython` |
| `python: command not found` | Use `python3` explicitly |
| Slow / hanging output | Use fewer threads: `-t 10` |
| No subdomains found | Verify domain is correct; try `-e all` |
| `requests.exceptions.SSLError` | Update certs: `pip3 install --upgrade certifi` |

---

## 🔍 Alternative Subdomain Enumeration Tools

| Tool | Description |
|------|-------------|
| [Amass](https://github.com/OWASP/Amass) | OWASP project — most comprehensive |
| [Subfinder](https://github.com/projectdiscovery/subfinder) | Fast, written in Go |
| [Assetfinder](https://github.com/tomnomnom/assetfinder) | Lightweight CLI tool |
| [Findomain](https://github.com/Findomain/Findomain) | Cross-platform, very fast |
| [crt.sh](https://crt.sh) | Web-based — uses SSL certificate logs |

---

## 📌 References

- [Sublist3r GitHub Repository](https://github.com/aboul3la/Sublist3r)
- [OWASP Amass](https://owasp.org/www-project-amass/)
- [crt.sh — Certificate Transparency Search](https://crt.sh)
- [Parrot Security OS](https://www.parrotsec.org/)
