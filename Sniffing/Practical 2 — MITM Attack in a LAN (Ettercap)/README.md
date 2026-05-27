# Practical 2 — MITM Attack in a LAN (Ettercap)

> Lab writeup. Performed against my own VMs on an isolated host-only network.
> See [`../README.md`](../README.md) for lab setup and scope.

---

## 1. Objective

Demonstrate how Ettercap automates a full ARP-poisoning MITM on a LAN, places
the attacker on the wire between two targets, and exposes plaintext protocols
to inspection. Pair the offensive view with detection and prevention.

---

## 2. Background

### Ettercap vs. raw arpspoof
`arpspoof` (from dsniff) is a single-purpose tool: it sends forged ARP
replies. Ettercap is a framework — it does ARP poisoning, ICMP redirect,
DHCP spoofing, DNS spoofing (via the `dns_spoof` plugin), and live protocol
dissection (telnet, FTP, HTTP Basic, SMTP AUTH, IMAP, POP3, SNMP v1/v2c,
NTLM, etc.). For a learner, Ettercap is the "one tool that ties the lesson
together."

### What MITM actually buys an attacker
Once traffic flows through the attacker:
- **Read** — passive inspection of any unencrypted protocol.
- **Modify** — inject, rewrite, or drop frames (Ettercap filters / etterfilter).
- **Redirect** — DNS spoofing or HTTP redirects to attacker-controlled hosts.

Modern transport encryption (TLS 1.3, QUIC, DoH/DoT) defeats the **read**
capability for most traffic but leaves metadata (SNI, IP, timing) and any
legacy plaintext protocol exposed.

---

## 3. Lab Procedure

### 3.1 GUI workflow
```bash
sudo ettercap -G
```
1. **Sniff → Unified sniffing → eth1**
2. **Hosts → Scan for hosts** (populates the host list from the /24)
3. **Hosts → Hosts list** — add `192.168.56.20` to **Target 1**, `192.168.56.1` to **Target 2**.
4. **Mitm → ARP poisoning → Sniff remote connections**
5. **Start → Start sniffing**

Credentials and tokens from supported plaintext protocols stream into the
bottom pane in real time. For HTTP forms on the lab DVWA, the username and
password appear as Ettercap's built-in HTTP dissector parses the POST.

### 3.2 CLI equivalent
```bash
sudo ettercap -T -q -i eth1 \
              -M arp:remote /192.168.56.20// /192.168.56.1//
```
`-T` text UI, `-q` quiet, `-M arp:remote` bidirectional poisoning.

### 3.3 Optional: DNS spoofing plugin
Edit `/etc/ettercap/etter.dns` to add `lab.test A 192.168.56.10`, then add
`-P dns_spoof` to the command line. A victim DNS lookup for `lab.test`
returns the attacker IP. Useful for phishing-page demos against a lab-only
domain.

### 3.4 Cleanup
Stop Ettercap with `q` — it sends corrective ARPs on exit. Then:
```bash
sudo iptables -t nat -F
echo 0 | sudo tee /proc/sys/net/ipv4/ip_forward
```
Restore snapshots.

---

## 4. Detection

| Signal | Where to look |
|---|---|
| Duplicate MAC for known IPs | `arp -a`, arpwatch alerts |
| Sudden surge in ARP traffic | NIDS / netflow baselines |
| TTL changes on routed packets | Endpoint network telemetry |
| Plugin DNS spoofing → unexpected resolutions | DNS query logging, DoH/DoT pinning |

Ettercap itself is noisy — it ARPs frequently and answers every ARP request
on the segment. An idle network with arpwatch will light up quickly.

---

## 5. Prevention

- **Dynamic ARP Inspection + DHCP snooping** on switches.
- **Port security** with sticky MAC, max-1-per-port on user access.
- **Private VLANs** for high-density user segments.
- **802.1X** to authenticate devices at L2.
- **End-to-end encryption** (TLS 1.3 + HSTS; SSH; VPN) so even successful
  MITM yields little.
- **Encrypted DNS** (DoH / DoT) blunts the `dns_spoof` plugin.

---

## 6. Modern Relevance

The L2 trust problem ARP poisoning exploits is unchanged since 1982 — only
the controls around it have improved. On managed enterprise networks with
DAI and 802.1X, this attack fails at step one. On flat networks (cheap
unmanaged switches, hotel/coffee-shop Wi-Fi, IoT VLANs), it still works.
Reading useful content from the resulting MITM is now much harder thanks to
TLS adoption — most of what you see is metadata.

---

## 7. References

- Ettercap project — https://www.ettercap-project.org/
- Cisco — Dynamic ARP Inspection configuration guide.
- RFC 826 — Ethernet Address Resolution Protocol (the original sin).
- OWASP — MITM Attack reference.
