# Practical 4 — Network Traffic Monitoring with darkstat

> Lab writeup. darkstat is a passive **monitoring** tool intended for use on
> networks you administer. Unlike Practicals 1–3 and 5, this is a defensive
> / observational exercise — no MITM is required and the tool is appropriate
> to run on a network you own or have authorization to monitor.

---

## 1. Objective

Use darkstat to passively observe traffic on a network segment, produce
per-host and per-port statistics, and read the resulting web report.
Understand where this fits in a defender's tool chain alongside heavier
options (ntopng, Zeek, Suricata, Elastic + Packetbeat).

---

## 2. Background

### What darkstat is
darkstat is a small, single-binary network statistics collector written by
Emil Mikulic. It captures via libpcap, accounts bytes per host / per port /
per protocol, and serves a built-in HTTP report on a configurable port.
Designed to run as a long-lived daemon on a router or monitoring host.

### Where it sits
| Tool | What it gives you | When to reach for it |
|---|---|---|
| **darkstat** | Per-host / per-port byte counts, simple graphs | Tiny home / lab visibility, embedded routers |
| **ntopng** | Rich flows, deep dissection, alerts | Mid-size networks needing flow analysis |
| **Zeek** | Protocol-aware logs, scripting | Threat hunting, SOC pipelines |
| **Suricata** | Signature + flow IDS/IPS | Active detection |

darkstat is the right starting tool when you want answers to *"who is
talking to whom on my network, and how much"* without standing up a full
SOC stack.

### What it does *not* do
- Decrypt traffic.
- Generate alerts on anomalies.
- Parse application-layer content beyond basic protocol identification.

---

## 3. Lab Procedure

Run on the **gateway VM** (pfSense / a Linux router VM) so it sees the
whole segment, or on the attacker VM with the interface in promiscuous
mode on a mirrored port — both are legitimate monitoring positions for a
network I own.

### 3.1 Install
```bash
sudo apt update
sudo apt install darkstat
```

### 3.2 Run
```bash
sudo darkstat -i eth1 -p 8080 --local 192.168.56.0/24 \
              --no-promisc-check \
              --chroot /var/lib/darkstat \
              --user nobody
```
Flags:
- `-i eth1` — capture interface
- `-p 8080` — HTTP port for the report UI
- `--local 192.168.56.0/24` — treat this range as "internal" for graphs
- `--chroot` / `--user` — drop privileges after binding pcap

### 3.3 View report
Browse to `http://192.168.56.1:8080/` from any lab host. Tabs:
- **Graphs** — bandwidth over time (last minute / hour / day / month).
- **Hosts** — per-IP byte totals, sortable.
- **Ports** — top destination / source ports across the capture window.

### 3.4 Generating lab traffic
On the victim VM, run a mix of:
- A few `curl` / `wget` downloads of varied sizes.
- `iperf3` against the attacker VM for a sustained flow.
- Normal browsing.

Within a minute the Hosts and Ports tables populate; the Graphs tab shows
the iperf burst clearly.

### 3.5 Stop
```bash
sudo pkill darkstat
```

---

## 4. Practical Uses

- **Baseline a quiet lab** — know what "normal" looks like before running
  any of the attack practicals; deviations become easy to spot.
- **Spot a noisy host** — identify the VM that's chatting to the internet
  unexpectedly.
- **Confirm an attack worked** in a controlled exercise — after running
  Practical 1 or 2, darkstat on a third VM (passive observer) shows the
  unexpected attacker↔victim flow.

---

## 5. Limitations and When to Move Up

- No alerting → pair with Suricata for detection.
- Counter-based, not flow-based → upgrade to ntopng for NetFlow / IPFIX.
- Single-host, no scaling → for multi-segment visibility look at
  Zeek + Kafka + Elastic.

---

## 6. References

- darkstat — https://unix4lyfe.org/darkstat/
- Zeek — https://zeek.org/
- Suricata — https://suricata.io/
- NIST SP 800-61 Rev. 2 — Computer Security Incident Handling Guide.
