# Practical 2 — SET Credential Harvester (WAN Delivery)

> **Format: concept + defence.** No cloned pages, hosting steps, tunneling
> walkthroughs, or link-delivery workflow appear in this writeup. See
> [`../README.md`](../README.md) for why.

---

## 1. Objective

Understand how the credential-harvester pattern changes when delivery
moves from a single LAN segment to the public internet — what new
infrastructure is involved, what extra defences come into play, and what
makes WAN phishing the dominant credential-theft vector today.

---

## 2. Background

### What changes from LAN to WAN
The core mechanism (cloned login page, recording listener) is the same.
What changes is the **delivery and reachability** layer:

| Concern | LAN variant | WAN variant |
|---|---|---|
| Reachability | Listener bound on local IP | Listener must be reachable from the internet (public host, port forward, or reverse tunnel like ngrok / Cloudflared) |
| Delivery | DNS spoofing or LAN chat | Email, SMS, social DM, malicious ad, search-result poisoning |
| Hostname | Existing internal name | Lookalike domain (typo-squat, IDN homograph, subdomain abuse) |
| Trust signal | "It's on the corp network" | Email sender / brand impersonation |
| Defender stack | NAC, DAI, internal DNS | DMARC, SPF, DKIM, mail filtering, URL detonation, brand monitoring |

WAN delivery is the dominant form of phishing in the wild because the
attacker doesn't need any access to the victim's network — only the
ability to send a message and host a page. Both are trivially available.

### The infrastructure side (described, not provided)
A realistic WAN phishing kit needs four things: a registered lookalike
domain, a TLS certificate (Let's Encrypt makes this free and instant), a
public-facing host, and a delivery channel that survives basic spam
filtering. Each one of those is a control point that defenders target —
which is why the modern defence stack looks the way it does.

### Why lookalike domains work
- **Typosquatting**: `paypa1.com`, `mlcrosoft.com`.
- **Subdomain abuse**: `login-microsoft.cdn-host.com` — the rightmost
  registrable label is the attacker's, but users read left-to-right.
- **IDN homographs**: Cyrillic `а` vs Latin `a`. Largely mitigated by
  modern browsers' punycode display rules, but still occasionally
  effective.
- **Brand-confusion TLDs**: `.support`, `.security`, `.login`.

### Why TLS no longer signals safety
A green padlock on a lookalike domain proves the connection is
encrypted, not that the site is genuine. Free certificate authorities
democratised TLS — which was the right outcome for the web — but it
also retired "look for the padlock" as user-facing security advice.

---

## 3. Detection

### Inbound mail / message side
- **DMARC enforcement** (`p=reject`) on sending domains, with aggregate
  and forensic reports monitored.
- **SPF + DKIM** correctly published.
- **Mail-filter URL detonation** (Defender for Office, Proofpoint TAP,
  Mimecast) clicks links in a sandbox and inspects landing pages.
- **Anti-spoofing rules** for display-name impersonation of internal
  staff.

### Brand / domain side
- **Domain monitoring** (DNSTwist, URLScan, Phishtank feeds) for
  lookalike registrations and certificate-transparency hits on owned
  brand names.
- **CT log monitoring** — every cert issued for a typosquat domain
  appears in CT logs within minutes.
- **Takedown workflow** with registrars, hosting providers, and browser
  Safe-Browsing teams.

### Identity / outcome side
- **Sign-in risk policies** at the IdP — impossible-travel, unfamiliar
  location, anonymizer / Tor exit, malware-infected IP.
- **Anomaly detection on OAuth consent grants** (the modern variant —
  consent-phishing — bypasses passwords entirely).

### User-reporting side
- **Report-phish button** in mail and chat clients, routed to a
  fast-triage queue.
- **No-blame reporting culture** so users surface mistakes immediately.

---

## 4. Prevention

### Make harvested credentials worthless
- **Phishing-resistant MFA** (FIDO2 / WebAuthn / passkeys) — the single
  highest-impact control. A relayed password without a matching
  authenticator binding cannot complete a sign-in.
- **Conditional Access** binding sign-in to managed devices.
- **Token-binding / DPoP** where supported.
- **Short-lived sessions** and continuous access evaluation (CAE) so a
  stolen token has a short useful life.

### Make the delivery harder
- **DMARC p=reject** on all owned domains.
- **Defensive registrations** of obvious typosquats.
- **Brand-protection service** for takedowns at scale.
- **External-sender banners** in mail and chat clients.

### Make the landing page less convincing
- **Single sign-on for everything** — when users almost never see a
  standalone password prompt, a page that shows one becomes suspect.
- **Password managers** that refuse to autofill on a different domain
  — this is one of the strongest *passive* anti-phishing controls and
  scales without user training.

### Train the humans
- **Phishing simulations** with consent and a no-blame outcome.
- **Just-in-time prompts** when a user clicks an external link in mail.

---

## 5. Modern Relevance

WAN-delivered credential phishing remains the most common initial-access
vector in incident reports year after year (Verizon DBIR, M-Trends,
CrowdStrike Global Threat Report). What has shifted:

- **Adversary-in-the-middle (AiTM) kits** (Evilginx, Modlishka, Muraena)
  proxy the real site in real time and relay MFA prompts — defeating
  push and TOTP MFA, but **not** WebAuthn/passkeys.
- **Consent phishing** asks the victim to OAuth-grant an attacker app
  instead of typing a password — bypasses passwords entirely. Mitigated
  by admin consent policies and verified-publisher requirements.
- **Browser-in-the-browser (BitB)** spoofs a popup login window with
  HTML/CSS. Mitigated by password managers (which check the *real*
  origin, not the rendered one).

The defensive direction of travel is clear: move authentication off
passwords entirely (passkeys), and stop expecting users to detect
deception that gets more convincing every year.

---

## 6. References

- TrustedSec — Social-Engineer Toolkit
- Verizon DBIR (latest edition) — phishing as initial-access vector
- CISA — *Implementing Phishing-Resistant MFA*
- Microsoft — *Defending against AiTM phishing* (Threat Intelligence blog)
- FIDO Alliance — Passkeys overview
- RFC 7489 — DMARC
- RFC 6376 — DKIM
- RFC 7208 — SPF
- Google Safe Browsing — https://safebrowsing.google.com
