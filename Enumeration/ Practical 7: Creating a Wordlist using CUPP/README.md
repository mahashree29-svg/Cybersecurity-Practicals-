# Practical 7: Creating a Wordlist using CUPP (Common User Password Profiler)

## Objective

In this practical we use the **CUPP** tool to generate a targeted password wordlist based on personal information about a victim (name, nickname, date of birth, partner, pet, company, etc.). Such "profile-based" wordlists are far more effective than generic dictionaries when an attacker has done OSINT on the target.

---

## Tool Overview

**CUPP — Common User Password Profiler** is a Python script that builds custom wordlists from personal details. People very often base their passwords on names, dates, and meaningful words, so a wordlist generated from OSINT is one of the most effective inputs for tools like `hydra`, `john`, or `hashcat`.

**Capabilities:**

- Interactive profile builder (`-i`)
- Download well-known default-password lists (`-l`)
- Improve an existing wordlist with leet/character transformations (`-w`)
- Download character/charset alphabetical lists (`-a`)
- Update itself from the upstream repository (`-v`)

---

## Prerequisites

- A Linux system (Kali, Parrot, Ubuntu, Debian).
- **Python 3** installed.
- **Git** installed (to clone the repository).

Check both:

```bash
python3 --version
git --version
```

Install if missing:

```bash
sudo apt update
sudo apt install python3 git -y
```

> On Kali, CUPP is already packaged: `sudo apt install cupp -y`. The steps below show the universal GitHub-clone method that works on Parrot, Ubuntu, and any Linux.

---

## Step 1: Install CUPP

Clone the repository from GitHub and enter the directory:

```bash
git clone https://github.com/Mebus/cupp.git
cd cupp
ls
```

**Sample output:**

```
Cloning into 'cupp'...
remote: Enumerating objects: 412, done.
remote: Counting objects: 100% (28/28), done.
remote: Compressing objects: 100% (22/22), done.
remote: Total 412 (delta 10), reused 21 (delta 6), pack-reused 384
Receiving objects: 100% (412/412), 88.16 KiB | 1.10 MiB/s, done.
Resolving deltas: 100% (217/217), done.

CHANGELOG.md  cupp.cfg  cupp.py  Dockerfile  LICENSE  README.md
```

Make the script executable and check the help menu:

```bash
chmod +x cupp.py
python3 cupp.py -h
```

**Sample output:**

```
 ___________
 cupp.py!                 # Common
      \                   # User
       \   ,__,           # Passwords
        \  (oo)____       # Profiler
           (__)    )\
              ||--|| *    [ Muris Kurgas | j0rgan@remote-exploit.org ]
                          [ Mebus | https://github.com/Mebus/        ]

[+] Options:
   -h         show this help message and exit
   -i         Interactive questions for user password profiling
   -w FILE    Use this option to improve existing dictionary
   -l         Download huge wordlists from repository
   -a         Parse default usernames and passwords directly from Alecto DB
   -v         Version of the program
```

---

## Step 2: Generate a Wordlist Interactively (`-i`)

Run CUPP in interactive mode. It will prompt for personal details and combine them in many ways.

```bash
python3 cupp.py -i
```

**Sample interactive session:**

```
[+] Insert the information about the victim to make a dictionary
[+] If you don't know all the info, just hit enter when asked! ;)

> First Name: john
> Surname: doe
> Nickname: jonny
> Birthdate (DDMMYYYY): 15081995

> Partner's name: jane
> Partner's nickname: janie
> Partner's birthdate (DDMMYYYY): 22031996

> Child's name: tommy
> Child's nickname: tom
> Child's birthdate (DDMMYYYY): 05072020

> Pet's name: rex
> Company name: acmecorp

> Do you want to add some key words about the victim? Y/[N]: Y
> Please enter the words, separated by comma. [i.e. hacker,juice,black]: cricket,batman,marvel

> Do you want to add special chars at the end of words? Y/[N]: Y
> Do you want to add some random numbers at the end of words? Y/[N]: Y
> Leet mode? (i.e. leet = 1337) Y/[N]: Y

[+] Now making a dictionary...
[+] Sorting list and removing duplicates...
[+] Saving dictionary to john.txt, counting 24863 words.
[+] Now load your pistolero with john.txt and shoot! Good luck!
```

### What CUPP does with these inputs

CUPP takes each input and generates many variants:

- Concatenations: `johndoe`, `johnjane`, `johnrex`, `johnacmecorp`
- Capitalisations: `John`, `JOHN`, `jOhn`
- Date insertions: `john1995`, `john15081995`, `john95`, `1995john`
- Leet substitutions: `j0hn`, `j0hnd03`, `cr1ck3t`, `b4tm4n`
- Suffix special chars: `john!`, `john@`, `john123`, `john2024!`
- Combinations of all of the above

This produces tens of thousands of plausible passwords that are statistically far more likely to match the victim than a generic dictionary.

---

## Step 3: Locate the Generated Wordlist

CUPP saves the output file (named after the victim's first name) in the same `cupp/` directory.

```bash
ls -lh
```

**Sample output:**

```
-rw-r--r-- 1 kali kali 312K Nov 14 11:42 john.txt
-rw-r--r-- 1 kali kali 4.7K Nov 14 11:42 cupp.cfg
-rwxr-xr-x 1 kali kali  16K Nov 14 11:42 cupp.py
```

Peek at the contents:

```bash
head -30 john.txt
wc -l john.txt
```

**Sample output:**

```
john
John
JOHN
johnjohn
john1995
john95
john15
john1508
john15081995
john_doe
johndoe
johndoe1995
johnjane
johnjane1996
johnrex
john@acmecorp
j0hn
j0hnd03
j0hnd03!
jonny
jonny15
jonny95
jonny1995
jonny@123
batman1995
b4tm4n!
cricket@123
...

24863 john.txt
```

Move it to your wordlists folder for convenience:

```bash
mkdir -p ~/wordlists
mv john.txt ~/wordlists/john_doe_cupp.txt
ls ~/wordlists/
```

---

## Step 4: Other CUPP Modes

### 4.1 Improve an Existing Wordlist (`-w`)

Take an existing wordlist and expand it with leet/transform rules:

```bash
python3 cupp.py -w /usr/share/wordlists/rockyou.txt
```

CUPP will produce an enhanced file with leet variants and suffixes.

### 4.2 Download Common Default Wordlists (`-l`)

```bash
python3 cupp.py -l
```

**Sample output:**

```
Choose the section you want to download:

   1   Moby            14   french          27   places
   2   afrikaans       15   german          28   polish
   3   american        16   hindi           29   random
   4   aussie          17   italian         ...

   Files will be downloaded from http://www.cs.toronto.edu/~gpenn/csc2517/Wordlists/

Enter number: 3
[+] Downloading american.zip...
[+] Done.
```

These provide broad dictionaries to mix with the CUPP-generated targeted list.

### 4.3 Alecto Default Credentials (`-a`)

```bash
python3 cupp.py -a
```

Pulls common vendor/default username-password pairs — useful against routers, IoT, and unconfigured services.

---

## Step 5: Using the Wordlist for an Attack (Authorised Testing Only)

The generated wordlist plugs directly into common cracking/spraying tools.

**Online brute force with Hydra (against an SSH service you own):**

```bash
hydra -l john -P ~/wordlists/john_doe_cupp.txt ssh://192.168.56.102
```

**Offline hash cracking with John the Ripper:**

```bash
john --wordlist=~/wordlists/john_doe_cupp.txt hashes.txt
```

**Offline hash cracking with hashcat (NTLM example):**

```bash
hashcat -m 1000 -a 0 hashes.txt ~/wordlists/john_doe_cupp.txt
```

> Run these only against systems, accounts, and hashes you are authorised to test.

---

## Quick Reference

```bash
# Clone & install
git clone https://github.com/Mebus/cupp.git
cd cupp && chmod +x cupp.py

# 1. Interactive profile-based wordlist
python3 cupp.py -i

# 2. Enhance an existing wordlist
python3 cupp.py -w rockyou.txt

# 3. Download big default wordlists
python3 cupp.py -l

# 4. Default-credential lists
python3 cupp.py -a

# 5. Version / update
python3 cupp.py -v
```

---

## Observations / Conclusion

Using CUPP we generated a **personalised wordlist (~24,000 entries)** from a small set of facts about the target — name, partner, child, pet, DOB, company, hobbies. Compared with a generic dictionary, this list is dramatically more effective because:

- Most users base passwords on personal facts (names, birthdays, pets, sports teams).
- Capitalisation, leet substitutions, and trailing numbers/specials are predictable transformations that CUPP automates.
- The list size (~24k) is small enough for fast brute force against either online services (with caution and lockout-awareness) or offline hashes.

This practical also demonstrates how **OSINT directly fuels password attacks** — every social-media post that reveals a partner's name, pet, or birthday materially weakens password security.

---

## Defensive Recommendations

- Enforce **minimum length of 12+ characters** and discourage personally-identifying strings.
- Adopt **passphrases** (e.g. four unrelated random words) — they resist profile-based wordlists by definition.
- Deploy a **password manager** and randomly-generated unique passwords per service.
- Enable **Multi-Factor Authentication (MFA)** everywhere — defeats wordlist attacks even on weak passwords.
- Check passwords against **known-breach lists** at creation time (e.g. HaveIBeenPwned API).
- Enforce **account lockout** or progressive throttling to defeat online brute-force.
- Train users on **OSINT hygiene** — limit public exposure of details like birthdays, partners, pets, and employer names.

---

## Disclaimer

This practical is for **educational purposes** only. Generating wordlists is not illegal in itself, but using them to attempt access to systems, accounts, or hashes you do not own or have explicit written authorisation to test is illegal under laws such as the Information Technology Act in India and the Computer Fraud and Abuse Act in the US.
