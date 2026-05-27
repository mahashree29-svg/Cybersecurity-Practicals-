# Practical 3 — Image Sniffing with Driftnet

> Lab writeup. Performed against my own VMs on an isolated host-only network.
> See [`../README.md`](../README.md) for lab setup and scope.

---

## 1. Objective

Use Driftnet to extract images from traffic flowing through the attacker
host. Understand what categories of traffic Driftnet can and cannot recover
in 2026, and why.

---

## 2. Background

### What Driftnet does
Driftnet is a passive image sniffer written by Chris Lightfoot (2002). It
reads packets via libpcap, reassembles HTTP responses, and extracts JPEG,
GIF, and PNG payloads. It does not perform MITM by itself — it only
consumes the traffic it can see on the local NIC. To see another host's
traffic on a switched network, you have to combine it with ARP poisoning,
a SPAN/mirror port, or a tap.

### Why most modern traffic is invisible to it
- **HTTPS / TLS** — image bytes are inside an encrypted tunnel. Driftnet
  sees only ciphertext.
- **HTTP/2 and HTTP/3** — multiplexed, header-compressed (HPACK / QPACK),
  and effectively always over TLS in the wild.
- **Lazy loading + CDN sharding** — even on plaintext HTTP, image streams
  are fragmented across many short-lived connections.

So Driftnet today is mostly a teaching artefact: it demonstrates how
trivially plaintext payloads leak on a shared medium, which is the lesson
that motivated universal TLS.

---

## 3. Lab Procedure

### 3.1 With a SPAN / mirror port (preferred — passive)
On a managed lab switch or a virtual switch configured for promiscuous
mode, mirror the victim port to the attacker port. Then simply:
```bash
sudo driftnet -i eth1
```
A window opens; any unencrypted image traffic on the mirrored segment is
displayed. This is the cleanest setup because no L2 attack is involved.

### 3.2 With ARP poisoning (active, for completeness)
For this lab I mirror Practical 2's setup so Driftnet has traffic to chew on:
```bash
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
sudo ettercap -T -q -i eth1 \
              -M arp:remote /192.168.56.20// /192.168.56.1//
# In another terminal:
sudo driftnet -i eth1 -d /tmp/driftnet -a
```
Flags:
- `-i eth1` — interface
- `-d /tmp/driftnet` — temp dir for extracted images
- `-a` — adjunct mode: don't open a display, just save to disk

### 3.3 Generating lab traffic
The victim VM browses to a deliberately HTTP-only lab gallery I host on the
DVWA VM (a directory of CC-licensed test images served via Python's
`http.server`). Images appear in the Driftnet window / output directory
within seconds.

### 3.4 Cleanup
```bash
sudo pkill driftnet
sudo pkill ettercap
sudo iptables -t nat -F
echo 0 | sudo tee /proc/sys/net/ipv4/ip_forward
rm -rf /tmp/driftnet
```
Snapshot restore.

---

## 4. Detection

Driftnet itself is **passive** — it does not transmit and is invisible on
the wire. What's visible is the **MITM mechanism that feeds it**:

| Signal | Where to look |
|---|---|
| ARP cache anomalies | arpwatch, `arp -a` |
| Unusual span/mirror configuration | Switch config audits |
| Promiscuous NIC on a host | Endpoint EDR alerts |

If you suspect a passive listener is present and you can't catch it via L2
signals, you generally can't — that's the design of passive sniffing.
Defence is upstream: don't put plaintext on the wire in the first place.

---

## 5. Prevention

- **HTTPS everywhere** — the single control that retires Driftnet.
- **DAI / DHCP snooping / 802.1X** — prevents the ARP-poisoning feeder.
- **Switched fabric with no mirror access for users** — restricts who can
  see whose traffic.
- **Wi-Fi**: WPA3 (or at minimum WPA2 with PMF) to prevent client-to-client
  sniffing on the wireless segment.
- **Network segmentation** — limits what a sniffer can reach even if
  successfully placed.

---

## 6. Modern Relevance

Driftnet is included in the CEH curriculum because the *concept* it
demonstrates — that any plaintext byte on a shared medium can be picked
off by anyone with read access to that medium — is fundamental. The tool
itself is now largely a museum piece for in-browser traffic. It remains
useful on IoT segments, legacy industrial protocols, and any environment
still serving images over HTTP.

---

## 7. References

- Driftnet — https://github.com/deiv/driftnet
- RFC 8446 — TLS 1.3.
- Mozilla — HTTPS-Only Mode documentation.
- OWASP — Transport Layer Protection Cheat Sheet.
