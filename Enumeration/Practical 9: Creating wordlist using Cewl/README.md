# Practical 9: Creating a Wordlist using cewl

## Objective

In this practical we use the **cewl** (Custom Word List generator) tool to build a wordlist directly from a target website. `cewl` crawls the pages of the site, extracts every unique word it finds, and writes them to a file. The resulting list is then used as input to password-cracking tools like `john`, `hashcat`, or `hydra`.

The intuition: people and organisations often reuse domain-specific vocabulary (product names, team names, jargon, slogans, employee names) in their passwords. A wordlist built from the target's own website is statistically much more effective against that target than a generic dictionary.

---

## Tool Overview

**cewl — Custom Word List generator** is a Ruby application written by Robin Wood (DigiNinja). It spiders a URL to a specified depth and collects words for use in password attacks.

**Capabilities:**

- Spider a site to depth `N` and extract words.
- Filter by minimum/maximum word length.
- Include numbers, emails, or grouped n-grams.
- Extract author / publisher / company metadata from documents (PDF, DOCX, etc.).
- Use HTTP Basic / Digest authentication, custom headers, or a proxy.
- Output sorted by frequency.

---

## Prerequisites

- A Linux system (Kali / Parrot / Ubuntu / Debian).
- `cewl` installed (pre-installed on Kali and Parrot).

Check installation:

```bash
which cewl
cewl --version
```

Install if missing:

```bash
sudo apt update
sudo apt install cewl -y
```

A target website you are authorised to crawl. The examples below use `https://example.com` and the intentionally testable `https://digi.ninja/` (run by the author of `cewl`).

---

## Step 1: View the Help Menu

```bash
cewl --help
```

**Sample output (truncated):**

```
cewl 5.5.2 (Grouping) Robin Wood (robin@digi.ninja) (https://digi.ninja/)

Usage: cewl [OPTIONS] ... <url>

    OPTIONS:
        -h, --help: Show help.
        -k, --keep: Keep the downloaded file.
        -d <x>,--depth <x>: Depth to spider to, default 2.
        -m, --min_word_length: Minimum word length, default 3.
        -o, --offsite: Let the spider visit other sites.
        --exclude <file>: A file containing a list of paths to exclude.
        --allowed <regex>: A regex pattern that path must match to be followed.
        -w, --write: Write the output to the file.
        -u, --ua <agent>: User agent to send.
        -n, --no-words: Don't output the wordlist.
        -g <x>, --groups <x>: Return groups of words as well.
        --lowercase: Lowercase all parsed words.
        --with-numbers: Accept words with numbers in as well as just letters.
        --convert-umlauts: Convert common ISO-8859-1 (Latin-1) umlauts.
        -a, --meta: Include meta data.
            --meta_file <file>: Output file for meta data.
        -e, --email: Include email addresses.
            --email_file <file>: Output file for email addresses.
        --meta-temp-dir <dir>: The directory used by exiftool when parsing files.
        -c, --count: Show the count for each word found.
        -v, --verbose: Verbose.
        --debug: Extra debug information.
        Authentication
        --auth_type: Digest or basic.
        --auth_user: Authentication username.
        --auth_pass: Authentication password.
        Proxy Support
        --proxy_host: Proxy host.
        --proxy_port: Proxy port, default 8080.
        --proxy_username: Username for proxy, if required.
        --proxy_password: Password for proxy, if required.
        Headers
        --header, -H: In format name:value - can pass multiple.
        <url>: The site to spider.
```

---

## Step 2: Generate a Wordlist from a Site

The classic command from the practical:

```bash
cewl -d 3 -m 8 -w wordlist.txt https://example.com
```

| Flag | Meaning                                                     |
| ---- | ----------------------------------------------------------- |
| `-d 3` | **Depth** — follow links up to 3 pages deep from the start URL |
| `-m 8` | **Minimum word length** — only keep words ≥ 8 characters     |
| `-w wordlist.txt` | **Write** the output to `wordlist.txt`            |
| `<url>` | Target URL to crawl                                        |

**Sample console output:**

```
CeWL 5.5.2 (Grouping) Robin Wood (robin@digi.ninja) (https://digi.ninja/)

Spidering https://example.com to a depth of 3...
Visiting: https://example.com/
Visiting: https://example.com/about
Visiting: https://example.com/products
Visiting: https://example.com/products/widget-2000
Visiting: https://example.com/team
Visiting: https://example.com/contact

Finished spidering.
Wordlist written to wordlist.txt
```

> **Tip:** start with a small depth (`-d 1` or `-d 2`) on first run to gauge how big the site is. Large depths on big sites can take a long time and produce huge lists.

---

## Step 3: Inspect the Generated Wordlist

```bash
ls -lh wordlist.txt
wc -l wordlist.txt
head -20 wordlist.txt
```

**Sample output:**

```
-rw-r--r-- 1 kali kali 48K Nov 14 12:45 wordlist.txt
1247 wordlist.txt

innovation
enterprise
solutions
analytics
dashboard
performance
deployment
infrastructure
sustainability
methodology
automation
integration
optimization
engineering
documentation
developers
strategies
operations
manufacturing
collaboration
```

Every entry is 8+ characters long, exactly as requested by `-m 8`. These words came from the live site content (`<p>`, `<h1>`, etc., not HTML tags).

---

## Step 4: Useful Variations

### 4.1 Include words containing digits

By default, `cewl` only collects pure alphabetic tokens. To include words like `widget2000` or `apollo11`:

```bash
cewl -d 2 -m 6 --with-numbers -w wordlist_nums.txt https://example.com
```

### 4.2 Show word frequency counts

Sort by how often each word appears — high-frequency words are stronger candidates:

```bash
cewl -d 2 -m 8 -c -w wordlist_freq.txt https://example.com
```

**Sample output (first lines of `wordlist_freq.txt`):**

```
solutions, 42
enterprise, 31
products, 28
platform, 24
analytics, 19
dashboard, 17
innovation, 15
```

### 4.3 Extract email addresses

```bash
cewl -d 2 -e --email_file emails.txt -n https://example.com
```

`-n` suppresses the regular wordlist output so `cewl` focuses on emails. Useful for OSINT / phishing target lists.

### 4.4 Extract document metadata

`cewl` can call `exiftool` on linked PDF/DOCX files and pull out authors, software versions, and creation dates.

```bash
cewl -d 2 -a --meta_file meta.txt https://example.com
```

**Sample `meta.txt` excerpt:**

```
Author: John Doe
Creator: Microsoft Word
Producer: Adobe PDF Library 15.0
Last Modified By: jdoe@example.com
Created: 2023:07:14 09:12:33+05:30
```

This often leaks employee names and software versions — both valuable for further reconnaissance.

### 4.5 Combine wordlists and emails in one run

```bash
cewl -d 3 -m 8 -e --email_file emails.txt -w wordlist.txt https://example.com
```

### 4.6 Spoof the user agent

Some sites block default tool fingerprints:

```bash
cewl -d 2 -m 8 \
     -u "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36" \
     -w wordlist.txt https://example.com
```

### 4.7 Use a proxy (e.g. Burp Suite for visibility)

```bash
cewl -d 2 -m 8 \
     --proxy_host 127.0.0.1 --proxy_port 8080 \
     -w wordlist.txt https://example.com
```

### 4.8 Authenticate to crawl a logged-in area

```bash
cewl -d 2 -m 8 \
     --auth_type basic --auth_user alice --auth_pass S3cret! \
     -w wordlist.txt https://internal.example.com
```

### 4.9 Word groups (n-grams)

Generate 2- and 3-word groupings — useful against passphrases:

```bash
cewl -d 2 -m 6 -g 3 -w groups.txt https://example.com
```

**Sample output:**

```
enterprise
solutions
strategies
enterprise solutions
performance analytics
cloud native deployment
enterprise grade infrastructure
```

---

## Step 5: Refine the Wordlist with Standard Tools

Once `cewl` produces a list, post-processing makes it more useful:

```bash
# Lowercase everything and de-duplicate
tr '[:upper:]' '[:lower:]' < wordlist.txt | sort -u > wordlist_clean.txt

# Combine with a dictionary
cat wordlist_clean.txt /usr/share/wordlists/rockyou.txt | sort -u > combined.txt

# Apply hashcat rule mutations (leet, suffix, capitalisation)
hashcat --stdout wordlist_clean.txt -r /usr/share/hashcat/rules/best64.rule > wordlist_mut.txt
wc -l wordlist_mut.txt
```

---

## Step 6: Use the Wordlist for an Authorised Attack

Plug the cewl-generated list into common cracking tools.

**Online brute force against an HTTP form (on a system you own):**

```bash
hydra -l admin -P wordlist.txt 192.168.56.102 \
      http-post-form "/login:user=^USER^&pass=^PASS^:F=Invalid"
```

**Offline NTLM hash cracking:**

```bash
hashcat -m 1000 -a 0 hashes.txt wordlist.txt
```

**Wi-Fi WPA2 cracking against your own capture:**

```bash
aircrack-ng -w wordlist.txt -b AA:BB:CC:DD:EE:FF capture.cap
```

> Use these only against assets you own or have written authorisation to test.

---

## Quick Reference

```bash
# 1. Help
cewl --help

# 2. Classic crawl (depth 3, min length 8)
cewl -d 3 -m 8 -w wordlist.txt https://example.com

# 3. Include words with digits + frequency counts
cewl -d 2 -m 6 --with-numbers -c -w list.txt https://example.com

# 4. Pull emails
cewl -d 2 -e --email_file emails.txt -n https://example.com

# 5. Pull document metadata (authors, software)
cewl -d 2 -a --meta_file meta.txt https://example.com

# 6. Word groups (2- and 3-grams)
cewl -d 2 -m 6 -g 3 -w groups.txt https://example.com

# 7. Through a proxy with a custom user-agent
cewl -d 2 -m 8 \
     -u "Mozilla/5.0 ..." \
     --proxy_host 127.0.0.1 --proxy_port 8080 \
     -w wordlist.txt https://example.com

# 8. Authenticated crawl
cewl -d 2 -m 8 --auth_type basic --auth_user alice --auth_pass S3cret! \
     -w internal.txt https://internal.example.com
```

---

## Observations / Conclusion

Using `cewl` against a target website we successfully generated a custom wordlist composed of vocabulary actually used on the site. Compared to generic dictionaries:

1. **Higher hit rate** — domain-specific jargon, product/team names, and slogans frequently appear in passwords.
2. **Cheap to generate** — a single command produces hundreds-to-thousands of likely candidates.
3. **OSINT bonus** — metadata extraction (`-a`) leaks employee names and software versions; email extraction (`-e`) yields user lists.

Combined with **CUPP** (personal-info wordlists) and **crunch** (mathematical permutations), `cewl` rounds out the attacker's wordlist toolkit.

---

## Comparison with Other Wordlist Tools

| Tool      | Best for                                                                  |
| --------- | ------------------------------------------------------------------------- |
| `cewl`    | Site-specific vocabulary scraped from the target's own content            |
| `cupp`    | Personal-info-driven lists (names, DOB, pet, partner, etc.)                |
| `crunch`  | Mathematically exhaustive lists from charsets / patterns                  |
| `wordlists` (Kali) | Pre-built leak/dictionary lists (rockyou, SecLists, etc.)         |

A realistic workflow chains them: scrape with `cewl` → personalise with `cupp` → mutate with `hashcat` rules → fall back to `rockyou` for the long tail.

---

## Defensive Recommendations

- **Don't allow domain-specific or company-name passwords.** Enforce a deny-list seeded with words scraped from your own site.
- Encourage **passphrases** of 4+ random words — defeats single-token wordlists.
- Strip sensitive **document metadata** (author, software) before publishing PDFs/DOCXs.
- Hide / obfuscate **employee emails** on public pages, or use forms instead of `mailto:` links.
- Enforce **MFA** — even a perfectly matched wordlist guess cannot defeat a second factor.
- Rate-limit and lock out repeated failed authentications.
- Periodically run `cewl` against your own site and audit the result for words that should be banned in passwords.

---

## Disclaimer

This practical is for **educational purposes** only. `cewl` itself only fetches public web content (similar to a search-engine crawler), but using the generated wordlist to access systems or accounts you do not own or have explicit written authorisation to test is illegal under laws such as the Information Technology Act in India and the Computer Fraud and Abuse Act in the US. Always respect `robots.txt` and the target site's terms of service.
