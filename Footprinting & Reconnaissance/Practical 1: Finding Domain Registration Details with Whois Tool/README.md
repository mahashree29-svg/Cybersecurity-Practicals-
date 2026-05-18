# Practical 1: Finding Domain Registration Details with Whois Tool

## 📖 Description

Whenever organizations or service providers purchase a domain name or IP address, they submit their registration details to **IANA (Internet Assigned Numbers Authority)**. This data is stored in the **WHOIS database** and is publicly accessible.

The **`whois`** tool in Parrot Security Linux (or any Linux distribution) allows us to query this database and retrieve information about:

- The domain **registrant** (owner)
- The domain **registrar** (where it was registered)
- **Creation, update, and expiration** dates
- **Name servers (DNS)**
- **Administrative and technical contacts**

This is a core technique used in the **footprinting phase** of ethical hacking.

> ⚠️ **Disclaimer:** For educational and authorized testing purposes only.

---

## 🎯 Objective

- Understand the purpose of the WHOIS database.
- Use the `whois` command to gather domain registration details.
- Interpret the returned WHOIS records.

---

## 🛠️ Prerequisites

| Tool | Description |
|------|-------------|
| Parrot Security OS / Kali / Ubuntu | Linux operating system |
| `whois` | WHOIS lookup utility |

### 🔧 Installation (if not already installed)

```bash
sudo apt update
sudo apt install whois -y
```

---

## 🚀 Step-by-Step Procedure

### Step 1: Run a Whois Lookup

Open a terminal and execute the following command, replacing the target with the domain you want to investigate. In this example we target `hackthissite.org`:

```bash
whois hackthissite.org
```

---

## 📋 Sample Output

```
Domain Name: HACKTHISSITE.ORG
Registry Domain ID: D108500000XXXXXX-LROR
Registrar WHOIS Server: whois.namecheap.com
Registrar URL: http://www.namecheap.com
Updated Date: 2024-XX-XX
Creation Date: 2003-XX-XX
Registry Expiry Date: 2026-XX-XX
Registrar: NameCheap, Inc.
Registrar IANA ID: 1068
Name Server: NS1.EXAMPLE.COM
Name Server: NS2.EXAMPLE.COM
DNSSEC: unsigned
```

---

## 🧠 Key Information You Can Extract

| Field | Why It Matters |
|-------|----------------|
| **Registrant** | Identifies the owner of the domain |
| **Registrar** | Where the domain was purchased |
| **Creation Date** | Age of the domain (older = more trusted) |
| **Expiry Date** | When the domain needs renewal |
| **Name Servers** | DNS infrastructure used by the domain |
| **Contact Emails** | Useful for further OSINT / social engineering tests |

---

## 🛑 Troubleshooting

| Issue | Solution |
|-------|----------|
| `whois: command not found` | Install it: `sudo apt install whois` |
| `Redacted for Privacy` shown | Domain owner uses WHOIS privacy protection (GDPR-compliant) |
| Empty output | Try a different WHOIS server with `whois -h whois.iana.org <domain>` |

---

## 📌 References

- [ICANN WHOIS Lookup](https://lookup.icann.org/)
- [WHOIS Protocol RFC 3912](https://datatracker.ietf.org/doc/html/rfc3912)
- [Parrot Security OS](https://www.parrotsec.org/)
