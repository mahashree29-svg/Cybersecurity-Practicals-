# Practical 4: DNS Enumeration

## Objective

In this practical we use the `dnsenum` tool to perform DNS enumeration against a target domain. The goal is to discover name servers (NS), mail servers (MX), host records, subdomains, and — if a zone transfer is permitted — the entire DNS zone for the target.

---

## Tool Overview

`dnsenum` is a multi-threaded Perl script designed to gather as much DNS information about a domain as possible.

**Capabilities:**

- Retrieve the host's A record
- Retrieve name servers (NS)
- Retrieve mail (MX) records
- Attempt AXFR (zone transfer) on every name server
- Brute-force subdomains using a wordlist
- Perform reverse lookups on discovered IP ranges (PTR records)
- Identify network ranges via WHOIS and run `nmap` against them (optional)

---

## Prerequisites

- A Linux system (Kali Linux ships with `dnsenum` pre-installed).
- `dnsenum` installed:

  ```bash
  which dnsenum
  dnsenum --help
  ```

- If not installed:

  ```bash
  sudo apt update
  sudo apt install dnsenum -y
  ```

- A target domain you are authorized to test (use `zonetransfer.me` — a public test domain provided by DigiNinja that intentionally allows zone transfers).

---

## Step 1: Basic DNS Enumeration

Run `dnsenum` against the target domain:

```bash
dnsenum example.com
```

For a practical, more interesting example we will use `zonetransfer.me`, which is configured to demonstrate DNS misconfigurations:

```bash
dnsenum zonetransfer.me
```

**Sample output (truncated):**

```
dnsenum VERSION:1.2.6

-----   zonetransfer.me   -----


Host's addresses:
__________________

zonetransfer.me.                          7200   IN   A   5.196.105.14


Name Servers:
______________

nsztm1.digi.ninja.                        7200   IN   A   81.4.108.41
nsztm2.digi.ninja.                        7200   IN   A   167.88.42.94


Mail (MX) Servers:
___________________

ASPMX.L.GOOGLE.COM.                       3422   IN   A   74.125.24.26
ALT1.ASPMX.L.GOOGLE.COM.                  3422   IN   A   74.125.131.27
ALT2.ASPMX.L.GOOGLE.COM.                  3422   IN   A   142.250.27.27
ASPMX2.GOOGLEMAIL.COM.                    3422   IN   A   142.251.9.26
ASPMX3.GOOGLEMAIL.COM.                    3422   IN   A   142.251.111.26


Trying Zone Transfers and getting Bind Versions:
_________________________________________________


Trying Zone Transfer for zonetransfer.me on ns nsztm1.digi.ninja ...

zonetransfer.me.                7200   IN   SOA   nsztm1.digi.ninja. robin.digi.ninja.
zonetransfer.me.                7200   IN   HINFO "Casio fx-700G" "Windows XP"
zonetransfer.me.                7200   IN   TXT   "google-site-verification=tyP28J7JAUHA9fw2sHXMgcCC0I6XBmmoVi04VlMewxA"
zonetransfer.me.                7200   IN   MX    0 ASPMX.L.GOOGLE.COM.
zonetransfer.me.                7200   IN   A     5.196.105.14
zonetransfer.me.                7200   IN   NS    nsztm1.digi.ninja.
zonetransfer.me.                7200   IN   NS    nsztm2.digi.ninja.
14.105.196.5.IN-ADDR.ARPA.zonetransfer.me. 7200 IN PTR www.zonetransfer.me.
asfdbauthdns.zonetransfer.me.   7900   IN   AFSDB 1 asfdbbox.zonetransfer.me.
asfdbbox.zonetransfer.me.       7200   IN   A     127.0.0.1
canberra-office.zonetransfer.me. 7200  IN   A     202.14.81.230
dc-office.zonetransfer.me.      7200   IN   A     143.228.181.132
deadbeef.zonetransfer.me.       7201   IN   AAAA  dead:beaf::
email.zonetransfer.me.          2222   IN   NAPTR 1 1 "P" "E2U+email" "" email.zonetransfer.me.zonetransfer.me.
home.zonetransfer.me.           7200   IN   A     127.0.0.1
internal.zonetransfer.me.       300    IN   NS    intns1.zonetransfer.me.
intns1.zonetransfer.me.         300    IN   A     81.4.108.41
office.zonetransfer.me.         7200   IN   A     4.23.39.254
robinwood.zonetransfer.me.      302    IN   TXT   "Robin Wood"
sip.zonetransfer.me.            3333   IN   NAPTR 2 3 "P" "E2U+sip" "!^.*$!sip:customer-service@zonetransfer.me!" .
sqli.zonetransfer.me.           300    IN   TXT   "' or 1=1 --"
sshock.zonetransfer.me.         7200   IN   TXT   "() { :]}; echo ShellShocked"
staging.zonetransfer.me.        7200   IN   CNAME www.sydneyoperahouse.com.
www.zonetransfer.me.            7200   IN   A     5.196.105.14
xss.zonetransfer.me.            300    IN   TXT   "'><script>alert('Boo')</script>"
zonetransfer.me.                7200   IN   SOA   nsztm1.digi.ninja. robin.digi.ninja.
```

### What this output tells us

| Section            | Information Obtained                                                       |
| ------------------ | -------------------------------------------------------------------------- |
| Host's addresses   | IP of the apex domain (`5.196.105.14`)                                     |
| Name Servers       | Authoritative DNS servers (`nsztm1.digi.ninja`, `nsztm2.digi.ninja`)       |
| Mail (MX) Servers  | Mail handlers (Google Workspace in this case)                              |
| Zone Transfer      | **Successful AXFR** — the entire DNS zone was disclosed                    |
| Discovered hosts   | `internal`, `office`, `staging`, `home`, `dc-office`, etc.                 |
| Other leaks        | TXT records expose `ShellShock` test strings, XSS/SQLi entries, HINFO data |

---

## Step 2: Brute-Force Subdomain Enumeration

If a zone transfer is denied, `dnsenum` can brute-force subdomains against a wordlist.

```bash
dnsenum --enum -f /usr/share/dnsenum/dns.txt example.com
```

Useful flags:

| Flag          | Purpose                                                       |
| ------------- | ------------------------------------------------------------- |
| `--enum`      | Shortcut: enable threads + subdomain bruteforce + reverse lookup |
| `-f <file>`   | Wordlist for subdomain bruteforce                             |
| `--threads N` | Number of parallel threads                                    |
| `-r`          | Recurse into discovered subdomains                            |
| `--noreverse` | Skip reverse lookups                                          |
| `-o <file>`   | Save output to an XML file                                    |

**Sample output (truncated):**

```
Brute forcing with /usr/share/dnsenum/dns.txt:
_______________________________________________

mail.example.com.                         300    IN   A   93.184.216.34
www.example.com.                          300    IN   A   93.184.216.34
ftp.example.com.                          300    IN   A   93.184.216.34
vpn.example.com.                          300    IN   A   203.0.113.45
dev.example.com.                          300    IN   A   203.0.113.46
```

---

## Step 3: Manual Verification with `dig` / `host`

You can verify `dnsenum` findings using standard DNS tools.

**Get name servers:**

```bash
dig NS zonetransfer.me +short
```

```
nsztm1.digi.ninja.
nsztm2.digi.ninja.
```

**Get mail servers:**

```bash
dig MX zonetransfer.me +short
```

```
0 ASPMX.L.GOOGLE.COM.
10 ALT1.ASPMX.L.GOOGLE.COM.
10 ALT2.ASPMX.L.GOOGLE.COM.
20 ASPMX2.GOOGLEMAIL.COM.
20 ASPMX3.GOOGLEMAIL.COM.
```

**Attempt a zone transfer manually:**

```bash
dig AXFR zonetransfer.me @nsztm1.digi.ninja
```

If AXFR succeeds you will see every record in the zone, exactly as `dnsenum` did.

**TXT records (often leak SPF, DKIM, verification tokens):**

```bash
dig TXT zonetransfer.me +short
```

---

## Step 4: Save Output for Reporting

Save results to an XML file for inclusion in a report:

```bash
dnsenum -o report.xml zonetransfer.me
```

Inspect the file:

```bash
ls -lh report.xml
head -40 report.xml
```

---

## Observations / Conclusion

Using `dnsenum` against the target, we successfully extracted:

1. The apex A record and authoritative name servers.
2. MX (mail) records — useful for OSINT and phishing reconnaissance.
3. A full zone transfer (AXFR) — revealing internal hostnames such as `internal`, `office`, `dc-office`, and `staging`.
4. Misconfiguration leftovers in TXT records (test strings, verification tokens).

This information enables an attacker to:

- Map the target's external attack surface (web, VPN, mail, dev/staging).
- Identify internal naming conventions for later social engineering.
- Pivot to discovered IPs/subdomains for service enumeration.
- Spoof or phish more convincingly using MX/SPF information.

---

## Defensive Recommendations

- **Disable AXFR on public name servers.** Restrict zone transfers to specific secondary NS IPs only:

  ```
  allow-transfer { 192.0.2.10; 192.0.2.11; };
  ```

- Use **split-horizon DNS** so internal hostnames are not visible to external resolvers.
- Avoid descriptive subdomains (`backup`, `oldsite`, `staging`, `dev`) that hint at unhardened systems.
- Remove unused or test records (HINFO, debug TXT entries) from production zones.
- Monitor authoritative DNS logs for repeated AXFR or brute-force NXDOMAIN patterns.
- Rate-limit DNS queries (RRL) to slow enumeration tools.

---

## Disclaimer

This practical is for **educational purposes** only and must be performed only on domains you own or have explicit written authorization to test (`zonetransfer.me` is intentionally provided for this kind of learning). Unauthorized DNS enumeration or zone-transfer probing may violate computer-misuse laws such as the Information Technology Act in India and the Computer Fraud and Abuse Act in the US.
