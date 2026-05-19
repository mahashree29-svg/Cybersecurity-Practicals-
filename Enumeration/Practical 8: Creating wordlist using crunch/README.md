# Practical 8: Creating a Wordlist using `crunch`

## Objective

In this practical we use the **crunch** tool to generate custom wordlists based on a chosen character set, minimum/maximum length, and an optional pattern. Such wordlists are widely used as input for password-cracking and brute-force tools like `john`, `hashcat`, `hydra`, and `aircrack-ng`.

---

## Tool Overview

**crunch** is a Linux command-line wordlist generator that creates every possible combination of a given character set within a chosen length range. It can:

- Generate **all permutations** of a character set within a min/max length.
- Use a **pattern (`-t`)** with placeholders (`@`, `,`, `%`, `^`) for targeted generation.
- Read an external **charset file (`-f`)**.
- **Split output** into multiple files by size or line count.
- **Pipe directly** into cracking tools (saves disk space on huge lists).

---

## Prerequisites

- A Linux system (Kali, Parrot, Ubuntu, Debian).
- `crunch` installed (pre-installed on Kali/Parrot).

Check installation:

```bash
which crunch
crunch --help 2>&1 | head -5
```

Install if missing:

```bash
sudo apt update
sudo apt install crunch -y
```

---

## Basic Syntax

```
crunch <min> <max> [charset] [options]
```

| Argument        | Meaning                                                                   |
| --------------- | ------------------------------------------------------------------------- |
| `<min>`         | Minimum length of generated words                                         |
| `<max>`         | Maximum length of generated words                                         |
| `[charset]`     | Characters to use (default = lowercase `a-z`)                              |
| `-o <file>`     | Write output to a file                                                    |
| `-t <pattern>`  | Use a fixed pattern with placeholders                                     |
| `-f <file> <name>` | Load a named charset from `charset.lst`                                |
| `-b <size>`     | Split output into chunks of given size (e.g. `100mb`)                     |
| `-c <num>`      | Split output every N lines                                                |
| `-p`            | Permute words/characters instead of cartesian generation                  |
| `-s <string>`   | Start generation from this string                                         |
| `-z <type>`     | Compress the output (`gzip`, `bzip2`, `lzma`, `7z`)                       |

**Pattern placeholders for `-t`:**

| Symbol | Meaning                              |
| ------ | ------------------------------------ |
| `@`    | Lowercase letter                     |
| `,`    | Uppercase letter                     |
| `%`    | Digit                                |
| `^`    | Special character                    |

---

## Step 1: Basic Wordlist (Length 4, Default Charset a–z)

Generate every 4-letter combination from `a` to `z`:

```bash
crunch 4 4
```

**IMPORTANT** — before letting `crunch` run, it prints the **predicted file size and line count**. Always read this first.

**Sample preview:**

```
Crunch will now generate the following amount of data: 2284880 bytes
2 MB
0 GB
0 TB
0 PB
Crunch will now generate the following number of lines: 456976

aaaa
aaab
aaac
aaad
...
zzzz
```

`26⁴ = 456,976` lines, ~2 MB — manageable.

Now compare with what happens at length 8:

```bash
crunch 8 8
```

```
Crunch will now generate the following amount of data: 1879491808000 bytes
1879491 MB
1879 GB
1 TB
Crunch will now generate the following number of lines: 208827064576
```

**~1.8 TB.** Press `Ctrl+C` immediately — this would fill the disk. Always check the preview.

---

## Step 2: Save Output to a File (`-o`)

```bash
crunch 4 4 -o wordlist4.txt
```

**Sample output:**

```
Crunch will now generate the following amount of data: 2284880 bytes
...
crunch: 100% completed generating output
```

Verify:

```bash
ls -lh wordlist4.txt
wc -l wordlist4.txt
head -5 wordlist4.txt
tail -5 wordlist4.txt
```

```
-rw-r--r-- 1 kali kali 2.2M Nov 14 12:08 wordlist4.txt
456976 wordlist4.txt
aaaa
aaab
aaac
aaad
aaae
zzzv
zzzw
zzzx
zzzy
zzzz
```

---

## Step 3: Custom Character Set

Specify exactly which characters to use. Order does **not** matter, but the set defines the alphabet.

**Digits only, length 4 (PIN-style):**

```bash
crunch 4 4 0123456789 -o pins.txt
```

```
Crunch will now generate the following amount of data: 50000 bytes
Crunch will now generate the following number of lines: 10000
```

Produces every 4-digit PIN from `0000` to `9999`.

**Alphanumeric + symbols, length 6:**

```bash
crunch 6 6 abcdefghijklmnopqrstuvwxyz0123456789!@#$ -o complex6.txt
```

> Always run a quick mental check: `charsetSize ^ length × (length+1) bytes`. For 40 chars and length 6 this is ~2.86 GB — usually too big to store, so pipe it instead (see Step 6).

---

## Step 4: Named Charsets via `charset.lst` (`-f`)

`crunch` ships with predefined charsets in `/usr/share/crunch/charset.lst`.

View available names:

```bash
cat /usr/share/crunch/charset.lst | head -30
```

**Sample contents:**

```
hex-lower               = [0123456789abcdef]
hex-upper               = [0123456789ABCDEF]
numeric                 = [0123456789]
numeric-space           = [0123456789 ]
symbols14               = [!@#$%^&*()-_+=]
symbols-all             = [!@#$%^&*()-_+=~`[]{}|\:;"'<>,.?/]
ualpha                  = [ABCDEFGHIJKLMNOPQRSTUVWXYZ]
lalpha                  = [abcdefghijklmnopqrstuvwxyz]
mixalpha-numeric        = [abc...XYZ0123456789]
mixalpha-numeric-all    = [abc...XYZ0123456789!@#...]
```

Use one by name:

```bash
crunch 8 8 -f /usr/share/crunch/charset.lst mixalpha-numeric -o mix8.txt
```

> Watch the size warning: `62⁸ ≈ 218 trillion` lines — abort if not piping.

---

## Step 5: Targeted Patterns (`-t`)

Patterns dramatically cut size when you know part of the password.

**Pattern: `pass@@@@`** — fixed prefix `pass`, four lowercase letters:

```bash
crunch 8 8 -t pass@@@@ -o pass_suffix.txt
```

```
Crunch will now generate the following number of lines: 456976
```

Sample lines:

```
passaaaa
passaaab
passaaac
...
passzzzz
```

**Pattern: `,@@@@%%%%`** — `Uppercase, four lowercase, four digits` (e.g. `Admin1234`):

```bash
crunch 9 9 -t ,@@@@%%%% -o complex_pattern.txt
```

**Pattern: birthdate-suffix passwords `john%%%%`** (`john0000` … `john9999`):

```bash
crunch 8 8 -t john%%%% -o john_year.txt
wc -l john_year.txt        # 10000
```

**Pattern with special chars `@@@@^`** (4 lowercase + 1 special):

```bash
crunch 5 5 -t @@@@^ -o lower_plus_special.txt
```

---

## Step 6: Piping Directly into a Cracking Tool

Generating a multi-TB wordlist on disk is impractical. Pipe the output instead.

**To `aircrack-ng` for WPA cracking:**

```bash
crunch 8 8 0123456789 | aircrack-ng -w - -b AA:BB:CC:DD:EE:FF capture.cap
```

**To `hashcat` (stdin mode):**

```bash
crunch 6 8 abcdefghijklmnopqrstuvwxyz0123456789 | hashcat -m 0 hashes.txt
```

**To `john --stdin`:**

```bash
crunch 6 8 -t pass%%%% | john --stdin --format=raw-md5 hashes.txt
```

This avoids ever writing the wordlist to disk.

---

## Step 7: Splitting Large Wordlists

Split into chunks of a given size:

```bash
crunch 5 5 -o START -b 10mb
```

Produces `aaaaa-bdxlp.txt`, `bdxlq-dhcvg.txt`, etc., each ~10 MB. `-o START` tells crunch to auto-name files using the starting and ending strings of each chunk.

Split by line count:

```bash
crunch 4 4 -o START -c 100000
```

---

## Step 8: Resume / Start Position (`-s`)

If you stopped halfway and want to continue from a known position:

```bash
crunch 5 5 abcdefghijklmnopqrstuvwxyz -s mzzzz -o resume.txt
```

Generation begins at `mzzzz` and continues to `zzzzz`.

---

## Step 9: Permutations (`-p`)

`-p` permutes the supplied words/chars rather than doing full cartesian generation:

```bash
crunch 1 1 -p admin john root
```

```
adminjohnroot
adminrootjohn
johnadminroot
johnrootadmin
rootadminjohn
rootjohnadmin
```

Useful when you know the exact tokens, just not the order.

---

## Step 10: `man crunch` for Full Documentation

```bash
man crunch
```

Scroll through the manual — every option, every placeholder, and many worked examples are documented.

Press `q` to quit.

---

## Quick Reference

```bash
# 1. Basic generation (length 4, a-z)
crunch 4 4

# 2. Save to file
crunch 4 4 -o list.txt

# 3. Digits-only PINs
crunch 4 4 0123456789 -o pins.txt

# 4. Custom alphabet
crunch 6 6 abc123!@# -o mixed.txt

# 5. Named charset
crunch 8 8 -f /usr/share/crunch/charset.lst mixalpha-numeric -o mix8.txt

# 6. Patterns
crunch 8 8 -t pass%%%%             # pass0000..pass9999
crunch 9 9 -t ,@@@@%%%%            # Uppercase + 4 lower + 4 digits
crunch 5 5 -t @@@@^                # 4 lower + 1 special

# 7. Pipe to cracker (no disk usage)
crunch 8 8 0123456789 | aircrack-ng -w - -b AA:BB:CC:DD:EE:FF cap.cap

# 8. Split output
crunch 5 5 -o START -b 50mb

# 9. Resume
crunch 5 5 abc...z -s mzzzz -o more.txt

# 10. Permutations
crunch 1 1 -p admin user root
```

---

## Observations / Conclusion

Using `crunch` we generated wordlists ranging from a 2 MB exhaustive 4-letter list to targeted pattern-based lists like `pass%%%%`. Key takeaways:

1. **Always read the size preview** before letting `crunch` run — exhaustive lists grow exponentially.
2. **Patterns (`-t`) are far more efficient** than blind brute force when partial password structure is known.
3. **Piping** to `hashcat`, `john`, `aircrack-ng`, or `hydra` avoids enormous disk writes.
4. Combined with profile-based tools like CUPP and leak-derived lists like `rockyou.txt`, `crunch` rounds out a complete attacker dictionary toolkit.

---

## Comparison with Other Wordlist Tools

| Tool       | Strength                                                            |
| ---------- | ------------------------------------------------------------------- |
| `crunch`   | Mathematically exhaustive generation with character sets / patterns |
| `cupp`     | Personal-info / OSINT-driven targeted wordlists                     |
| `cewl`     | Scrapes a target's website for likely password words                 |
| `hashcat --stdout` with rules | Apply transformation rules to existing wordlists  |

In practice, a comprehensive attacker workflow combines all of these.

---

## Defensive Recommendations

- Enforce **password length ≥ 12** — each extra character multiplies the brute-force search space by the alphabet size.
- Encourage **passphrases** of random words — defeats character-level exhaustive lists like `crunch` output.
- Block known-bad and breach-corpus passwords at creation time.
- **MFA everywhere** — even a perfect wordlist match cannot bypass a second factor.
- Rate-limit and lock out repeated failed authentications.
- For WPA2/WPA3 Wi-Fi, use long random PSKs (or enterprise auth) — exhaustive numeric/short PSKs are well within reach of `crunch | aircrack-ng` pipelines.
- Monitor for unusual authentication patterns (high-rate attempts, distributed brute force).

---

## Disclaimer

This practical is for **educational purposes** only. Creating wordlists is not illegal in itself, but using them to access systems, accounts, hashes, or Wi-Fi networks you do not own or have explicit written authorisation to test is illegal under laws such as the Information Technology Act in India and the Computer Fraud and Abuse Act in the US.
