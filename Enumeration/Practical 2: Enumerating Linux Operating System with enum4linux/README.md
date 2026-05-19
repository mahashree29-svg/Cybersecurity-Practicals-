# Practical 2: Enumerating Linux Operating System with enum4linux

## Objective

In this practical we enumerate a Linux target machine to extract user details, NetBIOS information, share details, and password policy using the `enum4linux` tool.

---

## Tool Overview

`enum4linux` is a Linux/Unix enumeration tool written in Perl. It is a wrapper around Samba tools such as `smbclient`, `rpcclient`, `net`, and `nmblookup`. It is primarily used to enumerate information from Windows and Samba-based systems, but works well on Linux systems running Samba.

**Information that can be enumerated:**

- User accounts and usernames
- Group memberships
- Share names (SMB)
- Password policy (length, complexity, lockout)
- Operating system information
- NetBIOS / workgroup information
- RID cycling output (SID enumeration)

> **Note:** This tool works only in a LAN environment where SMB (TCP 139/445) is reachable on the target.

---

## Lab Setup

| Machine     | Operating System          | Example IP      |
| ----------- | ------------------------- | --------------- |
| Attacker    | Kali Linux                | 192.168.56.101  |
| Target      | Metasploitable 2 (Linux)  | 192.168.56.102  |

Verify connectivity before starting:

```bash
ping -c 3 192.168.56.102
```

Check if `enum4linux` is installed (it ships with Kali by default):

```bash
which enum4linux
enum4linux -h
```

---

## Step 1: Basic Enumeration (Default Scan)

The basic syntax of the tool is:

```bash
enum4linux <target-ip>
```

Running this against the Metasploitable target:

```bash
enum4linux 192.168.56.102
```

**Sample output (truncated):**

```
Starting enum4linux v0.8.9 ( http://labs.portcullis.co.uk/application/enum4linux/ )

 ==========================
|    Target Information    |
 ==========================
Target ........... 192.168.56.102
RID Range ........ 500-550,1000-1050
Username ......... ''
Password ......... ''
Known Usernames .. administrator, guest, krbtgt, domain admins, root, bin, none

 =====================================================
|    Enumerating Workgroup/Domain on 192.168.56.102    |
 =====================================================
[+] Got domain/workgroup name: WORKGROUP

 =================================================
|    Nbtstat Information for 192.168.56.102        |
 =================================================
Looking up status of 192.168.56.102
        METASPLOITABLE  <00> -         B <ACTIVE>  Workstation Service
        METASPLOITABLE  <03> -         B <ACTIVE>  Messenger Service
        METASPLOITABLE  <20> -         B <ACTIVE>  File Server Service
        ..__MSBROWSE__. <01> - <GROUP> B <ACTIVE>  Master Browser
        WORKGROUP       <00> - <GROUP> B <ACTIVE>  Domain/Workgroup Name
        WORKGROUP       <1d> -         B <ACTIVE>  Master Browser
        WORKGROUP       <1e> - <GROUP> B <ACTIVE>  Browser Service Elections

 ===========================================
|    Session Check on 192.168.56.102         |
 ===========================================
[+] Server 192.168.56.102 allows sessions using username '', password ''
```

A null session being allowed (last line above) confirms the target is vulnerable to anonymous enumeration.

---

## Step 2: Enumerate User Accounts (`-U`)

The `-U` option grabs the user list from the target.

```bash
enum4linux -U 192.168.56.102
```

**Sample output:**

```
 ===========================================
|    Users on 192.168.56.102                 |
 ===========================================
index: 0x1 RID: 0x3e8 acb: 0x00000010 Account: games       Name: games           Desc: (null)
index: 0x2 RID: 0x3ea acb: 0x00000010 Account: nobody      Name: nobody          Desc: (null)
index: 0x3 RID: 0x3ec acb: 0x00000010 Account: bind        Name: (null)          Desc: (null)
index: 0x4 RID: 0x3ee acb: 0x00000010 Account: proxy       Name: proxy           Desc: (null)
index: 0x5 RID: 0x3f0 acb: 0x00000010 Account: syslog      Name: (null)          Desc: (null)
index: 0x6 RID: 0x3f2 acb: 0x00000010 Account: user        Name: just a user,111, Desc: (null)
index: 0x7 RID: 0x3f4 acb: 0x00000010 Account: www-data    Name: www-data        Desc: (null)
index: 0x8 RID: 0x3f6 acb: 0x00000010 Account: root        Name: root            Desc: (null)
index: 0x9 RID: 0x3f8 acb: 0x00000010 Account: msfadmin    Name: msfadmin,,,     Desc: (null)
...

user:[games]    rid:[0x3e8]
user:[nobody]   rid:[0x3ea]
user:[root]     rid:[0x3f6]
user:[msfadmin] rid:[0x3f8]
user:[user]     rid:[0x3f2]
```

**What this reveals:**

- Valid usernames such as `root`, `msfadmin`, `user`, `www-data` — useful for password-attack inputs.
- Each user's Relative Identifier (RID).

---

## Step 3: Enumerate Shared Resources (`-S`)

The `-S` option lists SMB shares exposed by the target.

```bash
enum4linux -S 192.168.56.102
```

**Sample output:**

```
 ========================================
|    Share Enumeration on 192.168.56.102  |
 ========================================

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        tmp             Disk      oh noes!
        opt             Disk
        IPC$            IPC       IPC Service (metasploitable server (Samba 3.0.20-Debian))
        ADMIN$          IPC       IPC Service (metasploitable server (Samba 3.0.20-Debian))

[+] Attempting to map shares on 192.168.56.102
//192.168.56.102/print$ Mapping: DENIED, Listing: N/A
//192.168.56.102/tmp    Mapping: OK, Listing: OK
//192.168.56.102/opt    Mapping: DENIED, Listing: N/A
//192.168.56.102/IPC$   [E] Can't understand response:
```

**What this reveals:**

- Available shares (`tmp`, `opt`, `print$`, `IPC$`, `ADMIN$`).
- Whether each share allows anonymous mapping/listing — note `tmp` is fully accessible.

You can manually browse an open share with:

```bash
smbclient //192.168.56.102/tmp -N
```

---

## Step 4: Enumerate Password Policy (`-P`)

The `-P` option retrieves the password policy (minimum length, complexity, lockout settings).

```bash
enum4linux -P 192.168.56.102
```

**Sample output:**

```
 ============================================
|    Password Policy Information for 192.168.56.102    |
 ============================================

[+] Attaching to 192.168.56.102 using a NULL session
[+] Trying protocol 139/SMB...
[+] Found domain(s):
        [+] METASPLOITABLE
        [+] Builtin

[+] Password Info for Domain: METASPLOITABLE

        [+] Minimum password length: 5
        [+] Password history length: None
        [+] Maximum password age: Not Set
        [+] Password Complexity Flags: 000000

                [+] Domain Refuse Password Change: 0
                [+] Domain Password Store Cleartext: 0
                [+] Domain Password Lockout Admins: 0
                [+] Domain Password No Clear Change: 0
                [+] Domain Password No Anon Change: 0
                [+] Domain Password Complex: 0

        [+] Minimum password age: None
        [+] Reset Account Lockout Counter: 30 minutes
        [+] Locked Account Duration: 30 minutes
        [+] Account Lockout Threshold: None
        [+] Forced Log off Time: Not Set
```

**Interpretation:**

| Setting                       | Value      | Risk                                                  |
| ----------------------------- | ---------- | ----------------------------------------------------- |
| Minimum password length       | 5          | Weak — short passwords allowed                        |
| Password complexity           | Disabled   | Simple/dictionary passwords allowed                   |
| Account lockout threshold     | None       | Brute-force attacks not blocked                       |
| Password history              | None       | Users may reuse the same password                     |
| Maximum password age          | Not Set    | Passwords never expire                                |

---

## Additional Useful Options

| Option | Purpose                                              |
| ------ | ---------------------------------------------------- |
| `-G`   | Get group and member list                            |
| `-o`   | Get OS information                                   |
| `-i`   | Get printer information                              |
| `-r`   | RID cycling — enumerate users by SID range           |
| `-a`   | Do all of the above (most thorough scan)             |

**Full enumeration:**

```bash
enum4linux -a 192.168.56.102
```

---

## Observations / Conclusion

Using `enum4linux` against the Metasploitable target, we successfully extracted:

1. NetBIOS / workgroup name (`WORKGROUP`, hostname `METASPLOITABLE`)
2. A list of valid user accounts (`root`, `msfadmin`, `user`, `www-data`, etc.)
3. SMB shares with their access permissions (`tmp` was readable anonymously)
4. A weak password policy: 5-character minimum, no complexity, no lockout

This information allows an attacker to launch targeted attacks such as:

- Password brute-force / dictionary attacks against discovered usernames (e.g. `hydra`, `medusa`).
- Direct file access via accessible SMB shares.
- Pivoting using known service accounts.

---

## Defensive Recommendations

- Disable null/anonymous SMB sessions (`restrict anonymous = 2` in `smb.conf`).
- Enforce strong password policies (≥ 12 chars, complexity, history, expiry).
- Enable account lockout after a small number of failed attempts.
- Restrict SMB ports (139, 445) to trusted networks only via firewall.
- Disable unused Samba shares and the `IPC$` null share where possible.
- Keep Samba updated — the version in Metasploitable (3.0.20) has multiple known RCE vulnerabilities.

---

## Disclaimer

This practical is for **educational purposes** only and must be performed only on systems you own or have explicit written authorization to test. Unauthorized scanning or enumeration of systems is illegal under most jurisdictions (e.g. the Information Technology Act in India, the Computer Fraud and Abuse Act in the US).
