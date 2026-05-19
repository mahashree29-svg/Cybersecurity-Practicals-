# Practical 3: Nmap Enumeration Commands

## Objective

In this practical we enumerate a target system using the Nmap Scripting Engine (NSE) — the collection of `.nse` scripts that ship with Nmap. We will use NSE scripts to enumerate SMB users, SMB shares, the operating system, and supported algorithms (SSH/SSL).

---

## Tool Overview

**Nmap** (Network Mapper) is an open-source tool for network discovery and security auditing. The **Nmap Scripting Engine (NSE)** extends Nmap's functionality through Lua scripts grouped into categories such as `auth`, `brute`, `discovery`, `vuln`, `safe`, and `intrusive`.

| Component   | Purpose                                                       |
| ----------- | ------------------------------------------------------------- |
| `nmap`      | Port scanning, service/version detection, OS fingerprinting   |
| `.nse`      | Lua scripts that perform targeted enumeration tasks           |
| `--script`  | Flag used to invoke one or more scripts in a scan             |

---

## Lab Setup

| Machine     | Operating System          | Example IP      |
| ----------- | ------------------------- | --------------- |
| Attacker    | Kali Linux                | 192.168.56.101  |
| Target      | Metasploitable 2 (Linux)  | 192.168.56.102  |

Verify connectivity:

```bash
ping -c 3 192.168.56.102
nmap --version
```

---

## Step 1: Locate Available NSE Scripts

NSE scripts are stored on disk and can be listed with `locate`.

```bash
locate *.nse
```

**Sample output (truncated):**

```
/usr/share/nmap/scripts/acarsd-info.nse
/usr/share/nmap/scripts/address-info.nse
/usr/share/nmap/scripts/afp-brute.nse
/usr/share/nmap/scripts/afp-ls.nse
/usr/share/nmap/scripts/afp-path-vuln.nse
/usr/share/nmap/scripts/afp-serverinfo.nse
...
/usr/share/nmap/scripts/smb-enum-domains.nse
/usr/share/nmap/scripts/smb-enum-groups.nse
/usr/share/nmap/scripts/smb-enum-processes.nse
/usr/share/nmap/scripts/smb-enum-services.nse
/usr/share/nmap/scripts/smb-enum-sessions.nse
/usr/share/nmap/scripts/smb-enum-shares.nse
/usr/share/nmap/scripts/smb-enum-users.nse
/usr/share/nmap/scripts/smb-os-discovery.nse
/usr/share/nmap/scripts/smb-protocols.nse
/usr/share/nmap/scripts/smb-security-mode.nse
/usr/share/nmap/scripts/smb-vuln-ms17-010.nse
...
/usr/share/nmap/scripts/ssh2-enum-algos.nse
/usr/share/nmap/scripts/ssh-auth-methods.nse
/usr/share/nmap/scripts/ssh-hostkey.nse
/usr/share/nmap/scripts/ssl-enum-ciphers.nse
```

If `locate` does not return results, update the database first:

```bash
sudo updatedb
```

You can also list scripts directly from the Nmap scripts directory:

```bash
ls /usr/share/nmap/scripts/ | grep smb
ls /usr/share/nmap/scripts/ | wc -l
```

---

## SMB User Enumeration with NSE

The `smb-enum-users` script enumerates user accounts on a target running SMB (TCP 139/445).

```bash
nmap --script smb-enum-users -p 139,445 192.168.56.102
```

**Sample output:**

```
Starting Nmap 7.94 ( https://nmap.org )
Nmap scan report for 192.168.56.102
Host is up (0.00042s latency).

PORT    STATE SERVICE
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds

Host script results:
| smb-enum-users:
|   METASPLOITABLE\backup (RID: 1068)
|     Full name:
|     Description:
|     Flags:       Normal user account
|   METASPLOITABLE\bin (RID: 1004)
|     Flags:       Normal user account
|   METASPLOITABLE\msfadmin (RID: 3000)
|     Full name:   msfadmin,,,
|     Flags:       Normal user account
|   METASPLOITABLE\root (RID: 1000)
|     Flags:       Normal user account
|   METASPLOITABLE\user (RID: 1010)
|     Full name:   just a user,111,
|     Flags:       Normal user account
|_  METASPLOITABLE\www-data (RID: 33)

Nmap done: 1 IP address (1 host up) scanned in 3.21 seconds
```

---

## Step 2: Share Enumeration with NSE

The `smb-enum-shares` script lists shared resources on the target. Combining it with `smb-enum-users` gives a richer view.

```bash
nmap --script smb-enum-shares -p 139,445 192.168.56.102
```

**Sample output:**

```
PORT    STATE SERVICE
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds

Host script results:
| smb-enum-shares:
|   account_used: <blank>
|   \\192.168.56.102\IPC$:
|     Type: STYPE_IPC_HIDDEN
|     Comment: IPC Service (metasploitable server (Samba 3.0.20-Debian))
|     Anonymous access: READ/WRITE
|   \\192.168.56.102\ADMIN$:
|     Type: STYPE_IPC
|     Comment: IPC Service (metasploitable server (Samba 3.0.20-Debian))
|     Anonymous access: <none>
|   \\192.168.56.102\opt:
|     Type: STYPE_DISKTREE
|     Comment:
|     Anonymous access: <none>
|   \\192.168.56.102\tmp:
|     Type: STYPE_DISKTREE
|     Comment: oh noes!
|     Anonymous access: READ/WRITE
|_  \\192.168.56.102\print$:
|     Type: STYPE_DISKTREE
|     Comment: Printer Drivers
|     Anonymous access: <none>
```

**Observation:** The `tmp` and `IPC$` shares allow anonymous READ/WRITE — a serious misconfiguration.

You can also run several SMB scripts together using a wildcard:

```bash
nmap --script "smb-enum-*" -p 139,445 192.168.56.102
```

---

## Step 3: OS Enumeration with NSE

The `smb-os-discovery` script identifies the OS, computer name, domain/workgroup, and SMB version.

```bash
nmap --script smb-os-discovery -p 139,445 192.168.56.102
```

**Sample output:**

```
PORT    STATE SERVICE
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds

Host script results:
| smb-os-discovery:
|   OS: Unix (Samba 3.0.20-Debian)
|   Computer name: metasploitable
|   NetBIOS computer name:
|   Domain name: localdomain
|   FQDN: metasploitable.localdomain
|_  System time: 2026-05-19T05:42:11+00:00
```

You can also combine this with Nmap's built-in OS fingerprinting (requires root):

```bash
sudo nmap -O --script smb-os-discovery 192.168.56.102
```

**Sample fingerprint output:**

```
Aggressive OS guesses: Linux 2.6.9 - 2.6.33 (96%), Linux 2.6.39 (94%)
No exact OS matches for host
Network Distance: 1 hop
```

---

## Step 4: Enumerating Algorithms with NSE

Algorithm enumeration helps identify weak or deprecated cryptographic algorithms on services like SSH and SSL/TLS.

### 4.1 SSH Algorithm Enumeration

```bash
nmap --script ssh2-enum-algos -p 22 192.168.56.102
```

**Sample output:**

```
PORT   STATE SERVICE
22/tcp open  ssh

| ssh2-enum-algos:
|   kex_algorithms: (4)
|       diffie-hellman-group-exchange-sha256
|       diffie-hellman-group-exchange-sha1
|       diffie-hellman-group14-sha1
|       diffie-hellman-group1-sha1
|   server_host_key_algorithms: (2)
|       ssh-rsa
|       ssh-dss
|   encryption_algorithms: (13)
|       aes128-cbc
|       3des-cbc
|       blowfish-cbc
|       cast128-cbc
|       arcfour128
|       arcfour256
|       arcfour
|       aes192-cbc
|       aes256-cbc
|       rijndael-cbc@lysator.liu.se
|       aes128-ctr
|       aes192-ctr
|       aes256-ctr
|   mac_algorithms: (6)
|       hmac-md5
|       hmac-sha1
|       hmac-ripemd160
|       hmac-ripemd160@openssh.com
|       hmac-sha1-96
|_      hmac-md5-96
```

**Weaknesses spotted:**

- `diffie-hellman-group1-sha1` (Logjam-vulnerable 1024-bit group)
- `ssh-dss` host key (weak DSA)
- CBC-mode ciphers and `arcfour` (RC4) — both deprecated
- `hmac-md5` MAC — broken

### 4.2 SSL/TLS Cipher Enumeration

```bash
nmap --script ssl-enum-ciphers -p 443 192.168.56.102
```

**Sample output:**

```
PORT    STATE SERVICE
443/tcp open  https

| ssl-enum-ciphers:
|   SSLv3:
|     ciphers:
|       TLS_RSA_WITH_RC4_128_SHA (rsa 1024) - C
|       TLS_RSA_WITH_3DES_EDE_CBC_SHA (rsa 1024) - C
|     warnings:
|       64-bit block cipher 3DES vulnerable to SWEET32 attack
|       Broken cipher RC4 is deprecated by RFC 7465
|       CBC-mode cipher in SSLv3 (CVE-2014-3566 POODLE)
|   TLSv1.0:
|     ciphers:
|       TLS_RSA_WITH_AES_128_CBC_SHA (rsa 1024) - A
|       TLS_RSA_WITH_AES_256_CBC_SHA (rsa 1024) - A
|     warnings:
|       Key exchange (rsa 1024) of lower strength than certificate key
|_  least strength: C
```

**Interpretation:** SSLv3 + RC4 + 3DES enabled — vulnerable to POODLE, SWEET32, and BEAST attacks.

### 4.3 SSH Host Keys

```bash
nmap --script ssh-hostkey -p 22 192.168.56.102
```

**Sample output:**

```
| ssh-hostkey:
|   1024 60:0f:cf:e1:c0:5f:6a:74:d6:90:24:fa:c4:d5:6c:cd (DSA)
|_  2048 56:56:24:0f:21:1d:de:a7:2b:ae:61:b1:24:3d:e8:f3 (RSA)
```

---

## Useful Combined Commands

**Run all SMB enumeration scripts at once:**

```bash
nmap --script "smb-enum-*,smb-os-discovery" -p 139,445 192.168.56.102
```

**Run all "safe" scripts on common ports:**

```bash
nmap -sV --script "default,safe" -p- 192.168.56.102
```

**Save output in multiple formats:**

```bash
nmap -sV --script smb-enum-shares,smb-enum-users -p 139,445 \
  -oA enum_smb 192.168.56.102
```

This produces three files: `enum_smb.nmap`, `enum_smb.gnmap`, `enum_smb.xml`.

---

## Observations / Conclusion

Using Nmap NSE scripts against the Metasploitable target, we successfully gathered:

1. A list of valid SMB users (`root`, `msfadmin`, `user`, `www-data`, etc.).
2. SMB share configurations — including anonymously writable shares (`tmp`, `IPC$`).
3. The target operating system (Unix with Samba 3.0.20-Debian) and hostname.
4. SSH and SSL/TLS algorithms — many of them weak or deprecated.

This information can be used to:

- Build a focused wordlist of valid usernames for password attacks.
- Drop or read files from misconfigured shares.
- Target known CVEs against the identified Samba version.
- Downgrade or break weakly configured SSH/TLS sessions.

---

## Defensive Recommendations

- Disable anonymous SMB sessions and remove unused shares.
- Patch Samba — version 3.0.20 has multiple critical RCE issues (e.g., CVE-2007-2447).
- On SSH: disable DSA host keys, CBC ciphers, RC4, and weak DH groups; use `ssh-ed25519` or strong RSA keys and modern KEX algorithms.
- On TLS: disable SSLv3 and TLS 1.0/1.1; allow only TLS 1.2+ with AEAD ciphers; replace 1024-bit certificates with 2048-bit or higher.  
- Restrict management ports (22, 139, 445) to trusted networks using a firewall.
- Monitor for repeated NSE-style probes via IDS/IPS rules.

---

## Disclaimer

This practical is for **educational purposes** only and must be performed only on systems you own or have explicit written authorization to test. Unauthorized scanning or enumeration is illegal under most jurisdictions (e.g., the Information Technology Act in India and the Computer Fraud and Abuse Act in the US).
