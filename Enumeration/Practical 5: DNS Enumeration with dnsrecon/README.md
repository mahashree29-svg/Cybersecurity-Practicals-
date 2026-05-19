# Practical 5: DNS Enumeration with dnsrecon

## Objective

In this practical we use the `dnsrecon` tool to perform DNS enumeration against a target domain. The goals are to:

1. Discover services running on the target's DNS infrastructure via SRV records (VoIP, LDAP, Kerberos, etc.).
2. Attempt zone transfers (AXFR) against the target's name servers.
3. Enumerate standard records (A, AAAA, MX, NS, SOA, TXT) and brute-force subdomains.

---

## Tool Overview

`dnsrecon` is a Python-based DNS enumeration tool that supports a wide range of techniques. It is more flexible than `dnsenum`, with multiple attack types selectable via the `-t` flag.

**Common attack types (`-t`):**

| Type     | Purpose                                                          |
| -------- | ---------------------------------------------------------------- |
| `std`    | Standard records: SOA, NS, A, AAAA, MX, TXT, SPF                 |
| `axfr`   | Try a zone transfer against every authoritative NS                |
| `srv`    | Enumerate SRV records (services like SIP, LDAP, Kerberos, XMPP)   |
| `brt`    | Brute-force subdomains/hostnames using a wordlist                 |
| `rvl`    | Reverse-lookup an IP range (PTR records)                          |
| `goo`    | Google-based search for subdomains                                |
| `bing`   | Bing-based search for subdomains                                  |
| `crt`    | Use crt.sh (Certificate Transparency logs) to find subdomains     |
| `snoop`  | Cache snooping against a recursive resolver                       |
| `zonewalk` | Walk a DNSSEC-signed zone via NSEC records                      |

---

## Prerequisites

- A Linux system (Kali Linux ships with `dnsrecon` pre-installed).
- Confirm installation:

  ```bash
  which dnsrecon
  dnsrecon --help | head -20
  ```

- If not installed:

  ```bash
  sudo apt update
  sudo apt install dnsrecon -y
  ```

- A target domain you are authorized to test. We use `zonetransfer.me` — a public domain provided for DNS-recon learning that intentionally exposes records and allows AXFR.

---

## Step 1: Enumerate SRV Records (Services Including VoIP)

SRV records advertise services such as SIP/VoIP, LDAP, Kerberos, XMPP, autodiscover, and STUN. They are highly valuable for an attacker because they map directly to hostnames and ports.

```bash
dnsrecon -t srv -d example.com
```

Where:

- `-t srv` — attack type is **SRV record enumeration**
- `-d example.com` — target domain

Using a real test domain:

```bash
dnsrecon -t srv -d zonetransfer.me
```

**Sample output:**

```
[*] Performing General Enumeration against: zonetransfer.me...
[*] Checking for Zone Transfer...
[*] Resolving SRV Records.
[*]      SRV _sip._tcp.zonetransfer.me      www.zonetransfer.me     5.196.105.14   5060
[*]      SRV _sip._udp.zonetransfer.me      www.zonetransfer.me     5.196.105.14   5060
[*]      SRV _sips._tcp.zonetransfer.me     www.zonetransfer.me     5.196.105.14   5061
[*]      SRV _h323cs._tcp.zonetransfer.me   www.zonetransfer.me     5.196.105.14   1720
[*]      SRV _h323ls._udp.zonetransfer.me   www.zonetransfer.me     5.196.105.14   1719
[*] 5 Records Found
```

**Interpretation:**

| Record                  | Service Type            | Port  |
| ----------------------- | ----------------------- | ----- |
| `_sip._tcp`             | SIP (VoIP)              | 5060  |
| `_sip._udp`             | SIP (VoIP)              | 5060  |
| `_sips._tcp`            | SIP over TLS            | 5061  |
| `_h323cs._tcp`          | H.323 call signaling    | 1720  |
| `_h323ls._udp`          | H.323 location service  | 1719  |

These directly reveal that the target runs **VoIP services** — useful for further attacks (SIP user enumeration, registration hijacking, RTP eavesdropping).

For a typical enterprise/Active Directory target you would commonly see SRV records like:

```
_ldap._tcp.dc._msdcs.example.com   dc01.example.com    10.0.0.10   389
_kerberos._tcp.example.com         dc01.example.com    10.0.0.10   88
_gc._tcp.example.com               dc01.example.com    10.0.0.10   3268
_autodiscover._tcp.example.com     mail.example.com    192.0.2.5   443
```

These pinpoint Domain Controllers, Global Catalogs, Kerberos KDCs, and Exchange/Autodiscover servers.

---

## Step 2: Attempt Zone Transfer (AXFR)

A zone transfer disclosure leaks every DNS record in the zone — a critical misconfiguration.

```bash
dnsrecon -t axfr -d zonetransfer.me
```

**Sample output (truncated):**

```
[*] Testing NS Servers for Zone Transfer
[*] Checking for Zone Transfer for zonetransfer.me name servers
[*] Resolving SOA Record
[*]      SOA nsztm1.digi.ninja 81.4.108.41
[*] Resolving NS Records
[*] NS Servers found:
[*]      NS nsztm1.digi.ninja 81.4.108.41
[*]      NS nsztm2.digi.ninja 167.88.42.94
[*] Removing any duplicate NS server IP Addresses...
[*]
[*] Trying NS server 81.4.108.41
[+] 81.4.108.41 Has port 53 TCP Open
[+] Zone Transfer was successful!!
[*]      NS @ nsztm1.digi.ninja 81.4.108.41
[*]      NS @ nsztm2.digi.ninja 167.88.42.94
[*]      SOA nsztm1.digi.ninja 81.4.108.41
[*]      MX @ ASPMX.L.GOOGLE.COM 74.125.24.26
[*]      MX @ ALT1.ASPMX.L.GOOGLE.COM 74.125.131.27
[*]      A  zonetransfer.me 5.196.105.14
[*]      A  www.zonetransfer.me 5.196.105.14
[*]      A  office.zonetransfer.me 4.23.39.254
[*]      A  canberra-office.zonetransfer.me 202.14.81.230
[*]      A  dc-office.zonetransfer.me 143.228.181.132
[*]      A  internal.zonetransfer.me 81.4.108.41
[*]      A  intns1.zonetransfer.me 81.4.108.41
[*]      A  home.zonetransfer.me 127.0.0.1
[*]      A  asfdbbox.zonetransfer.me 127.0.0.1
[*]      AAAA deadbeef.zonetransfer.me dead:beaf::
[*]      CNAME staging.zonetransfer.me www.sydneyoperahouse.com
[*]      TXT robinwood.zonetransfer.me "Robin Wood"
[*]      TXT sqli.zonetransfer.me "' or 1=1 --"
[*]      TXT sshock.zonetransfer.me "() { :]}; echo ShellShocked"
[*]      TXT xss.zonetransfer.me "'><script>alert('Boo')</script>"
[*]      HINFO @ "Casio fx-700G" "Windows XP"
```

If AXFR is blocked, the output will instead be:

```
[-] Zone Transfer Failed for nsztm1.digi.ninja
[-] DNS Server Refused the Zone Transfer
```

---

## Step 3: Standard Record Enumeration

Run a general scan to gather SOA, NS, A, AAAA, MX, TXT, and SPF records.

```bash
dnsrecon -d zonetransfer.me
```

(`-t std` is the default if no `-t` is specified.)

**Sample output:**

```
[*] Performing General Enumeration against: zonetransfer.me...
[*]      SOA nsztm1.digi.ninja 81.4.108.41
[*]      NS  nsztm1.digi.ninja 81.4.108.41
[*]      NS  nsztm2.digi.ninja 167.88.42.94
[*]      MX  ASPMX.L.GOOGLE.COM 74.125.24.26
[*]      MX  ALT1.ASPMX.L.GOOGLE.COM 74.125.131.27
[*]      A   zonetransfer.me 5.196.105.14
[*]      TXT google-site-verification=tyP28J7JAUHA9fw2sHXMgcCC0I6XBmmoVi04VlMewxA
[*]      SPF v=spf1 include:_spf.google.com ~all
```

---

## Step 4: Brute-Force Subdomain Enumeration

If AXFR is denied, brute-forcing subdomains is the next best technique.

```bash
dnsrecon -t brt -d example.com -D /usr/share/wordlists/dnsmap.txt
```

| Flag | Meaning                                |
| ---- | -------------------------------------- |
| `-t brt` | Attack type: brute force            |
| `-d`     | Target domain                       |
| `-D`     | Wordlist for subdomain names        |

**Sample output:**

```
[*] Performing host and subdomain brute force against example.com
[*]      A  mail.example.com 93.184.216.34
[*]      A  www.example.com 93.184.216.34
[*]      A  ftp.example.com 93.184.216.34
[*]      A  dev.example.com 203.0.113.46
[*]      A  vpn.example.com 203.0.113.45
[*] 5 Records Found
```

---

## Step 5: Subdomains via Certificate Transparency

Find subdomains by querying public CT logs — passive and very effective.

```bash
dnsrecon -t crt -d example.com
```

**Sample output:**

```
[*] Performing Crt.sh Search Enumeration against example.com
[*]      A   admin.example.com 198.51.100.20
[*]      A   api.example.com   198.51.100.21
[*]      A   shop.example.com  198.51.100.22
[*]      A   mail.example.com  93.184.216.34
[*] 4 Records Found
```

---

## Step 6: Save Output for Reporting

`dnsrecon` supports XML, JSON, CSV, and SQLite output for reporting.

```bash
dnsrecon -d zonetransfer.me -t axfr -x report.xml
dnsrecon -d zonetransfer.me -t std  -j report.json
dnsrecon -d zonetransfer.me -t std  -c report.csv
```

Check the files:

```bash
ls -lh report.*
```

---

## Quick Reference

```bash
# 1. SRV / service enumeration (VoIP, AD, mail)
dnsrecon -t srv  -d example.com

# 2. Zone transfer
dnsrecon -t axfr -d example.com

# 3. Standard records
dnsrecon -d example.com

# 4. Subdomain brute force
dnsrecon -t brt -d example.com -D /usr/share/wordlists/dnsmap.txt

# 5. Certificate Transparency
dnsrecon -t crt -d example.com

# 6. Reverse lookup an IP range
dnsrecon -r 192.0.2.0/24

# 7. DNSSEC zone walking via NSEC
dnsrecon -t zonewalk -d example.com
```

---

## Observations / Conclusion

Using `dnsrecon` against the target, we successfully extracted:

1. **SRV records** revealing VoIP/SIP services on ports 5060/5061 and H.323 on 1719/1720.
2. **A successful zone transfer (AXFR)** disclosing internal hostnames (`internal`, `office`, `dc-office`, `staging`).
3. **Standard records** — SOA, NS, MX, TXT, SPF — exposing the mail-handling provider and verification tokens.
4. Additional subdomains via brute force and Certificate Transparency.

This data enables an attacker to:

- Target VoIP/SIP infrastructure (`svmap`, `sipvicious`, registration hijacking).
- Identify Domain Controllers and Kerberos KDCs in enterprise environments.
- Map internal naming conventions for social engineering.
- Build a focused list of secondary targets for service enumeration.

---

## Defensive Recommendations

- **Disable public AXFR.** Restrict zone transfers to known secondary NS IPs:

  ```
  allow-transfer { 192.0.2.10; };
  ```

- Use **split-horizon DNS** — keep internal SRV/A records on internal resolvers only.
- Avoid placing internal service hints (Kerberos, LDAP, SIP, autodiscover) in public zones.
- Periodically audit Certificate Transparency logs for unintended subdomain exposure.
- Remove descriptive subdomains (`dev`, `staging`, `backup`, `old`) from public zones.
- Apply DNS rate limiting (RRL) on authoritative servers to slow brute-force enumeration.
- Monitor DNS logs for repeated NXDOMAIN spikes (signature of brute-force enumeration).

---

## Disclaimer

This practical is for **educational purposes** only and must be performed only on domains you own or have explicit written authorization to test. `zonetransfer.me` is intentionally provided by its owner for DNS-recon learning. Unauthorized DNS reconnaissance may violate computer-misuse laws such as the Information Technology Act in India and the Computer Fraud and Abuse Act in the US.
