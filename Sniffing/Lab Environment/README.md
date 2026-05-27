# CEH Practicals — Sniffing

Personal study notes for the Certified Ethical Hacker track.

> **Scope and ethics.** Every practical in this repository is performed inside
> an isolated virtual lab on hardware I own. No third-party networks, devices,
> or accounts are touched. These notes exist to document understanding of
> attack mechanics **and the corresponding defences** — the blue-team half is
> intentionally given equal weight.
>
> Running these techniques against systems you do not own or have written
> authorization to test is illegal in most jurisdictions
> (India: IT Act 2000 §§43, 66; US: CFAA; UK: Computer Misuse Act 1990; etc.).

---

## Lab Environment

All practicals share the same virtual lab unless stated otherwise.

| Role     | OS                      | IP (host-only)   | Notes                                  |
|----------|-------------------------|------------------|----------------------------------------|
| Attacker | Kali Linux 2025.x       | 192.168.56.10    | Tools and capture host                 |
| Victim   | Ubuntu 24.04 Desktop    | 192.168.56.20    | Simulated user                         |
| Gateway  | pfSense / VBox NAT      | 192.168.56.1     | L3 boundary                            |
| Webapp   | DVWA / Juice Shop VM    | 192.168.56.30    | Deliberately vulnerable target service |

VirtualBox host-only adapter, no bridge to the physical LAN.
Snapshots taken before each exercise so the lab is resettable in seconds.

---

## Index

| #  | Practical                                  | Folder                                  |
|----|--------------------------------------------|-----------------------------------------|
| 1  | Sniffing passwords in a LAN (sslstrip)     | [Practical-1-Sniff-Passwords-LAN](./Practical-1-Sniff-Passwords-LAN) |
| 2  | MITM attack in a LAN (Ettercap)            | [Practical-2-MITM-Attack-LAN](./Practical-2-MITM-Attack-LAN) |
| 3  | Image sniffing with Driftnet               | [Practical-3-Driftnet-Image-Sniffing](./Practical-3-Driftnet-Image-Sniffing) |
| 4  | Network traffic monitoring with darkstat   | [Practical-4-Darkstat-Network-Monitoring](./Practical-4-Darkstat-Network-Monitoring) |
| 5  | MITM with the MITMf framework              | [Practical-5-MITMf-Framework](./Practical-5-MITMf-Framework) |

---

## Conventions

Each practical README follows the same structure so they read as study notes,
not as attack recipes:

1. **Objective** — what the exercise demonstrates.
2. **Background** — why each step works at the protocol level.
3. **Lab procedure** — commands run against my own VMs.
4. **Detection** — how a defender spots this.
5. **Prevention** — controls that block or limit it.
6. **Modern relevance** — how well the technique still works in 2026.
7. **References.**
