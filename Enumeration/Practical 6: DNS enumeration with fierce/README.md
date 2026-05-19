# Practical 6: DNS Enumeration with fierce

## Objective

In this practical we use the `fierce` tool to enumerate DNS records of a target domain. `fierce` first attempts a zone transfer (AXFR) against each authoritative name server. If that fails, it falls back to brute-forcing subdomains using a wordlist (its built-in list contains ~2280 entries), and can also perform reverse lookups on nearby IPs to discover related hosts.

---

## Tool Overview

`fierce` is a semi-lightweight DNS reconnaissance tool originally written by RSnake in Perl, now reimplemented in Python and shipped with Kali Linux. It is designed as a "locator" — it finds non-contiguous IP space and hostnames belonging to a target organization.

**What `fierce` does, in order:**

1. Look up the SOA and NS records of the target.
2. Try an AXFR (zone transfer) against every name server found.
3. If AXFR fails, brute-force subdomains using a wordlist.
4. For every host discovered, probe nearby IPs in the same `/24` (or configurable range) for reverse DNS — to find sibling hosts even outside the target's subdomains.

---

## Prerequisites

- Kali Linux or any Linux distro with `fierce` installed.
- Confirm installation:

  ```bash
  which fierce
  fierce --help | head -20
  ```

- If not installed:

  ```bash
  sudo apt update
  sudo apt install fierce -y
  ```

  Or via pip:

  ```bash
  pip install fierce
  ```

> **Note on syntax:** older versions used `fierce -dns example.com` (Perl). The current Python version uses `fierce --domain example.com`. Both forms are shown below.

---

## Step 1: Basic DNS Enumeration

Run `fierce` against the target domain:

```bash
fierce -dns juggyboy.com
```

Equivalent modern syntax:

```bash
fierce --domain juggyboy.com
```

For a working demonstration that actually produces meaningful output (including a successful zone transfer), we will use `zonetransfer.me`, which is provided by its owner specifically for DNS-recon learning:

```bash
fierce --domain zonetransfer.me
```

**Sample output (zone transfer succeeds):**

```
NS: nsztm1.digi.ninja. nsztm2.digi.ninja.
SOA: nsztm1.digi.ninja. (81.4.108.41)
Zone: success
{
  "zonetransfer.me.": "5.196.105.14",
  "www.zonetransfer.me.": "5.196.105.14",
  "office.zonetransfer.me.": "4.23.39.254",
  "internal.zonetransfer.me.": "81.4.108.41",
  "intns1.zonetransfer.me.": "81.4.108.41",
  "asfdbbox.zonetransfer.me.": "127.0.0.1",
  "canberra-office.zonetransfer.me.": "202.14.81.230",
  "dc-office.zonetransfer.me.": "143.228.181.132",
  "home.zonetransfer.me.": "127.0.0.1",
  "staging.zonetransfer.me.": "CNAME www.sydneyoperahouse.com.",
  "deadbeef.zonetransfer.me.": "dead:beaf::"
}
```

**Sample output (zone transfer denied — falls back to brute force):**

```
NS: ns1.juggyboy.com. ns2.juggyboy.com.
SOA: ns1.juggyboy.com. (192.0.2.10)
Zone: failure
Wildcard: failure
Brute-forcing with default wordlist (2280 entries)...

Found: mail.juggyboy.com.   (203.0.113.20)
Found: www.juggyboy.com.    (203.0.113.21)
Found: ftp.juggyboy.com.    (203.0.113.22)
Found: vpn.juggyboy.com.    (203.0.113.45)
Found: dev.juggyboy.com.    (198.51.100.10)
Found: webmail.juggyboy.com.(203.0.113.23)

Nearby:
{
  "203.0.113.19": "gateway.juggyboy.com.",
  "203.0.113.24": "intranet.juggyboy.com.",
  "203.0.113.46": "vpn-bak.juggyboy.com."
}
```

### What this output tells us

| Section          | Information Obtained                                                           |
| ---------------- | ------------------------------------------------------------------------------ |
| NS / SOA         | Authoritative name servers and zone owner                                      |
| Zone: success    | The NS allowed AXFR — every record in the zone is disclosed                    |
| Brute-force      | Valid subdomains found via wordlist (`mail`, `vpn`, `dev`, `webmail`, …)       |
| Nearby           | Additional hostnames discovered via reverse DNS on neighboring IPs (`gateway`, `intranet`, `vpn-bak`) |

---

## Step 2: Brute Force with a Custom Wordlist

If the default 2280-entry wordlist isn't producing enough results, supply your own:

```bash
fierce --domain example.com --subdomain-file /usr/share/wordlists/dnsmap.txt
```

Useful wordlists shipped with Kali:

- `/usr/share/wordlists/dnsmap.txt`
- `/usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt`
- `/usr/share/seclists/Discovery/DNS/namelist.txt`

**Sample output:**

```
Found: api.example.com.      (198.51.100.30)
Found: blog.example.com.     (198.51.100.31)
Found: shop.example.com.     (198.51.100.32)
Found: portal.example.com.   (198.51.100.33)
Found: admin.example.com.    (198.51.100.34)
```

---

## Step 3: Tune the Reverse-Lookup Range

By default, `fierce` looks 5 IPs above and below each discovered host for reverse DNS. Increase the window to find more siblings:

```bash
fierce --domain example.com --traverse 20
```

**Sample output:**

```
Found: www.example.com.   (198.51.100.50)

Nearby:
{
  "198.51.100.42": "old-www.example.com.",
  "198.51.100.45": "staging.example.com.",
  "198.51.100.48": "backup.example.com.",
  "198.51.100.55": "mail-bak.example.com.",
  "198.51.100.60": "vpn2.example.com."
}
```

This often reveals **forgotten / staging / backup hosts** that aren't named in DNS but live on the same IP block.

---

## Step 4: Other Useful Options

| Flag                       | Purpose                                                                  |
| -------------------------- | ------------------------------------------------------------------------ |
| `--domain <d>`             | Target domain                                                            |
| `--subdomains a,b,c`       | Inline list of subdomains to try (instead of a file)                     |
| `--subdomain-file <file>`  | Wordlist for subdomain brute-force                                       |
| `--dns-servers 8.8.8.8`    | Use a specific resolver (bypass local DNS cache)                         |
| `--traverse N`             | Number of IPs above/below each hit to reverse-lookup (default 5)          |
| `--wide`                   | Scan the entire `/24` around each discovered host (very noisy)            |
| `--connect`                | Attempt an HTTP HEAD request against every discovered host                |
| `--delay N`                | Seconds between requests — useful to evade rate limits                    |
| `--tcp`                    | Force TCP queries (some networks block UDP DNS)                          |

**Example — wide scan with a custom resolver and HTTP probing:**

```bash
fierce --domain example.com \
       --dns-servers 1.1.1.1 \
       --wide \
       --connect
```

---

## Step 5: Verify Findings Manually

Cross-check `fierce` results with standard tools.

**Resolve a discovered subdomain:**

```bash
dig +short mail.example.com
```

**Reverse-lookup a "nearby" IP:**

```bash
dig -x 198.51.100.34 +short
```

**Manually attempt AXFR (the same operation `fierce` performs internally):**

```bash
dig AXFR zonetransfer.me @nsztm1.digi.ninja
```

---

## Quick Reference

```bash
# 1. Default scan (AXFR + built-in 2280-word brute force)
fierce --domain example.com

# 2. Brute force with a custom wordlist
fierce --domain example.com --subdomain-file /usr/share/wordlists/dnsmap.txt

# 3. Wider reverse-lookup window
fierce --domain example.com --traverse 50

# 4. Whole /24 around each hit + HTTP probing
fierce --domain example.com --wide --connect

# 5. Use a specific public resolver
fierce --domain example.com --dns-servers 8.8.8.8

# 6. Old-style Perl syntax (some Kali installs)
fierce -dns example.com
```

---

## Observations / Conclusion

Using `fierce` against the target, we successfully:

1. Identified authoritative name servers and the SOA.
2. Attempted (and on misconfigured zones, succeeded with) an **AXFR zone transfer**.
3. Fell back to **brute-forcing** subdomains using the default ~2280-word list.
4. Discovered **adjacent hosts via reverse DNS** that were never named directly in the zone (a unique strength of `fierce`).

This information enables an attacker to:

- Map the entire externally-facing footprint of the target.
- Discover forgotten infrastructure (`old-www`, `backup`, `staging`, `vpn-bak`) that is often less hardened than production.
- Build a focused target list for service enumeration and exploitation.

---

## Comparison with Other DNS Tools

| Tool        | Strength                                                          |
| ----------- | ----------------------------------------------------------------- |
| `dnsenum`   | All-in-one: NS, MX, AXFR, subdomain brute, WHOIS, nmap follow-up  |
| `dnsrecon`  | Most flexible: SRV, CT logs, zone-walking, multiple output formats |
| `fierce`    | **Reverse-DNS sweep around discovered hosts** — finds neighbors  |

In practice, professionals run all three because each finds hosts the others miss.

---

## Defensive Recommendations

- **Disable AXFR** on public name servers; allow only specific secondary NS IPs.
- Use **split-horizon DNS** — internal hostnames must never live in the public zone.
- Avoid placing siblings of public hosts on the same `/24` (the trick `fierce --traverse` exploits).
- Retire descriptive hostnames (`backup`, `old`, `staging`, `dev`) from public DNS.
- Apply **DNS rate limiting (RRL)** to slow brute-force enumeration.
- Monitor authoritative DNS logs for NXDOMAIN spikes — a strong signature of subdomain brute-forcing.
- Periodically run `fierce`/`dnsrecon`/`dnsenum` against your own zones as part of routine attack-surface auditing.

---

## Disclaimer

This practical is for **educational purposes** only and must be performed only on domains you own or have explicit written authorization to test. `zonetransfer.me` is provided by its owner specifically for DNS-recon learning. Unauthorized DNS reconnaissance may violate laws such as the Information Technology Act in India and the Computer Fraud and Abuse Act in the US.
