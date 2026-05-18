# Practical 2: Extracting Emails and Subdomains using theHarvester

## 📖 Description

**theHarvester** is one of the most popular **OSINT (Open Source Intelligence)** tools used during the early stages of a penetration test or red-team engagement. It collects publicly available information about a target domain from various **search engines** and **public data sources**.

It can extract:

- 📧 **Employee emails**
- 🌐 **Subdomains**
- 🖥️ **Hosts / IP addresses**
- 👤 **Employee names**
- 🚪 **Open ports & banners**
- 🔗 **URLs and virtual hosts**

Data is gathered from **Google, Bing, Yahoo, DuckDuckGo, LinkedIn, Shodan, Hunter, GitHub**, and many other public sources.

> ⚠️ **Disclaimer:** For educational and authorized testing purposes only.

---

## 🎯 Objective

- Understand passive OSINT techniques.
- Use `theHarvester` to gather emails, subdomains, and hosts.
- Compare results across different search engines / data sources.

---

## 🛠️ Prerequisites

| Tool | Description |
|------|-------------|
| Parrot Security OS / Kali Linux | Pre-installed with `theHarvester` |
| `theHarvester` | OSINT data-gathering tool |
| Internet connection | Required for search engine queries |

### 🔧 Installation (if not already installed)

```bash
sudo apt update
sudo apt install theharvester -y
```

Or install the latest version from GitHub:

```bash
git clone https://github.com/laramies/theHarvester.git
cd theHarvester
pip3 install -r requirements/base.txt
```

---

## 🚀 Step-by-Step Procedure

### Step 1: View the Help Menu

```bash
theHarvester -h
```

### Step 2: Basic Usage Syntax

```bash
theHarvester -d <target-domain> -b <data-source> -l <limit>
```

| Flag | Description |
|------|-------------|
| `-d` | Target domain |
| `-b` | Data source (google, bing, duckduckgo, all, etc.) |
| `-l` | Limit number of results |
| `-f` | Save output to HTML/XML file |

### Step 3: Example Commands

#### 🔹 Gather emails & subdomains from Google
```bash
theHarvester -d hackthissite.org -b google -l 500
```

#### 🔹 Gather from all available sources
```bash
theHarvester -d hackthissite.org -b all -l 500
```

#### 🔹 Save results to an HTML report
```bash
theHarvester -d hackthissite.org -b all -l 500 -f report.html
```

#### 🔹 Use DuckDuckGo (no API key required)
```bash
theHarvester -d hackthissite.org -b duckduckgo
```

---

## 📋 Sample Output

```
*******************************************************************
*  _   _                                        _                 *
* | |_| |__   ___    /\  /\__ _ _ ____   _____ ___ ___| |_ ___ _ __  *
* | __| '_ \ / _ \  / /_/ / _` | '__\ \ / / _ / __/ __| __/ _ \ '__|  *
* | |_| | | |  __/ / __  / (_| | |   \ V /  __\__ \__ \ ||  __/ |     *
*  \__|_| |_|\___| \/ /_/ \__,_|_|    \_/ \___|___/___/\__\___|_|     *
*                                                                  *
*                  theHarvester 4.x.x                              *
*                                                                  *
*******************************************************************

[*] Target: hackthissite.org

[*] Searching Google.
[*] Searching Bing.
[*] Searching DuckDuckGo.

[*] Emails found: 5
---------------------
admin@hackthissite.org
contact@hackthissite.org
support@hackthissite.org
...

[*] Hosts found: 12
---------------------
www.hackthissite.org:198.148.81.135
mail.hackthissite.org:198.148.81.140
forum.hackthissite.org:198.148.81.150
...
```

---

## 🧠 Key Learnings

- Learned to perform **passive OSINT** without directly interacting with the target.
- Discovered how publicly indexed data can leak sensitive organizational info.
- Compared results from multiple search engines for thorough coverage.

---

## 🛑 Troubleshooting

| Issue | Solution |
|-------|----------|
| `theHarvester: command not found` | Install: `sudo apt install theharvester` |
| API key errors (Shodan/Hunter) | Add API keys in `api-keys.yaml` |
| `429 Too Many Requests` | Search engine rate-limit — wait or use a VPN |
| No results returned | Try `-b duckduckgo` or `-b all` for broader coverage |

---

## 📌 References

- [theHarvester GitHub](https://github.com/laramies/theHarvester)
- [OSINT Framework](https://osintframework.com/)
- [Parrot Security OS](https://www.parrotsec.org/)
