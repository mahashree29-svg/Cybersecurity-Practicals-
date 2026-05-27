# Practical 4 — Web-Jacking via SET

> **Format: concept + defence.** No working redirect page, hosting steps,
> or link-delivery workflow appear in this writeup. See
> [`../README.md`](../README.md) for why.

---

## 1. Objective

Understand SET's **Web-Jacking** attack pattern: the UX trick at its
core, where it sits relative to plain credential-harvesting and
clickjacking, and how modern browsers and identity controls largely
defang it.

---

## 2. Background

### The pattern in one sentence
The victim visits a page that displays a message along the lines of
**"This site has been moved — click here to continue"**, and the link
takes them to a cloned credential-harvester page on the attacker's
host.

That's it. There is no exploit, no payload, no MITM — just a UI
deception that buys a click. SET automates the page generation and the
listener; the technique itself is older than SET.

### Why it earned a separate name
It sits between three related techniques:

| Technique | What's deceptive |
|---|---|
| Plain credential harvester | The login page itself looks like the real one |
| Clickjacking | A real button is hidden under a transparent overlay so the click does something different than the user thinks |
| **Web-jacking** | A fake "the site moved" interstitial convinces the user to *initiate* the navigation to the attacker's page |

The defining property of web-jacking is that the user clicks
*deliberately* but on a misleading premise. There is no overlay trick
(unlike clickjacking) and no immediate fake login page (unlike a plain
harvester). It's a two-step deception: interstitial → cloned login.

### Why two steps instead of one
Two reasons attackers historically reached for it:
- **URL inspection misdirection.** Some users glance at the URL on
  *arrival* at a login page. A short interstitial breaks that habit
  because the URL bar changes between the interstitial and the landing
  page.
- **Trust laundering.** The interstitial often sits on a *compromised
  legitimate site*. The user trusts that domain, clicks the "moved"
  link, and lands on the attacker domain still in the afterglow of
  that trust.

---

## 3. Detection

### Page-side (defender controls the destination brand)
- **Brand monitoring** for cloned login pages — same DNSTwist /
  Certificate Transparency / URLScan workflow as Practical 2.
- **Referer analytics on the real login page.** A sudden burst of
  legitimate sign-ins with no referer or a typosquat referer is
  worth investigating.

### Hosting-side (defender controls the *interstitial* site, e.g. a
compromised CMS they own)
- **File integrity monitoring** on web roots — the interstitial is an
  injected file or modified template, which an FIM picks up immediately.
- **CMS plugin / theme audit** — most web-jacking injections ride on
  outdated WordPress / Joomla components.
- **Outbound link policy** in the CMS — banner / interstitial of any
  kind requires editorial review.

### Browser / endpoint side
- **Safe Browsing / SmartScreen** classifies known phishing landing
  pages and warns before navigation completes.
- **Browser referrer-policy and `noopener`** reduce information leakage
  but don't prevent the user click.
- **Password-manager origin checks** — autofill refuses to populate
  credentials on the wrong domain. Stops the harvest even if the
  interstitial succeeds.

---

## 4. Prevention

### Owned-site hygiene (so attackers can't host interstitials on your domain)
- **Patch the CMS and plugins.** Web-jacking injections almost always
  ride on known CVEs in outdated components.
- **Restrict file write permissions** on the web root.
- **WAF rules** for known web-shell upload patterns.
- **Subresource Integrity (SRI)** on third-party scripts; **CSP** with
  `script-src 'self'` and `report-uri` to alert on unexpected scripts.
- **Outbound-link allowlist** for embedded `<a href>` in user content.

### Identity layer (so a successful harvest doesn't matter)
Same controls as Practicals 1 and 2:
- **Phishing-resistant MFA** (passkeys / FIDO2).
- **Conditional Access** binding sign-in to managed devices.
- **Short-lived sessions** and Continuous Access Evaluation.

### Browser / user layer
- **HSTS preload** on owned web properties.
- **Password manager** with strict origin matching.
- **External-link banners** in mail and chat clients.
- **Phishing-report channel** with a fast triage SLA.

---

## 5. Modern Relevance

Web-jacking as a labelled technique is mostly of historical interest in
2026. The reasons it has faded:

- **Browser warnings on cross-origin navigation** from compromised
  sites (Safe Browsing) intercept many interstitials before the user
  clicks.
- **Password managers refusing to autofill on a different domain**
  break the second step regardless of how convincing the first step
  was.
- **HSTS + URL-bar UX changes** make the cross-domain hop more visible.
- **AiTM kits** (Practical 2) outperform web-jacking for sophisticated
  attackers because they capture session tokens directly, including
  MFA-completed ones.

It survives as a teaching example because it cleanly illustrates the
*two-step trust transfer* idea, which is useful for understanding more
modern variants like consent phishing and OAuth-grant attacks (where
the trust transfer happens at the identity-provider level instead of
the browser level).

---

## 6. References

- TrustedSec — Social-Engineer Toolkit
- OWASP — *Phishing Prevention Cheat Sheet*
- Google Safe Browsing — https://safebrowsing.google.com
- Microsoft Defender SmartScreen — overview docs
- W3C — Content Security Policy Level 3
- W3C — Subresource Integrity
- Verizon DBIR — social-engineering attack patterns
