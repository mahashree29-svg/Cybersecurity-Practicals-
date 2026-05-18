# Practical 5: Google Dorks (Google Hacking)

## 📖 Description

**Google Dorking** (also known as **Google Hacking**) is the technique of using **advanced Google search operators** to find information that is **publicly accessible but not easily discoverable** through normal searches.

This technique helps in:

- 🔍 **Filtering** out irrelevant search results
- 📁 **Finding specific file types** (PDF, DOCX, XLSX, SQL backups, etc.)
- 🌐 **Discovering exposed login pages and admin panels**
- 🔓 **Identifying misconfigured servers and directories**
- 📂 **Finding sensitive documents** unintentionally indexed by Google

Google Dorks are widely used during the **information gathering** phase of penetration testing to uncover sensitive data that organizations may have unintentionally exposed.

> ⚠️ **Disclaimer:** This practical is for **educational and authorized testing purposes only**. Do **NOT** access, download, or use any sensitive information you may find. Doing so may be illegal under laws such as the IT Act 2000 (India), CFAA (USA), and GDPR (EU).

---

## 🎯 Objective

- Understand the syntax of advanced Google search operators.
- Use Google Dorks to filter and refine search results.
- Identify sensitive data leakage scenarios in real-world web applications.

---

## 🛠️ Requirements

| Requirement | Description |
|-------------|-------------|
| Web Browser | Firefox / Chrome / Brave |
| Google Search | [https://www.google.com](https://www.google.com) |

---

## 🚀 Common Google Dork Operators

### 🔹 Operator 1: `intitle:`

Filters pages by their **HTML title**.

```
intitle:"Index of/"
```

🔍 **Result:** Displays pages where directory listing is enabled and exposed (often with the title `Index of/`).

---

### 🔹 Operator 2: `inurl:`

Searches for pages that contain a specific keyword in the **URL**.

```
inurl:certifiedhacker
```

🔍 **Result:** Returns all indexed pages whose URLs contain `certifiedhacker`.

---

### 🔹 Operator 3: `filetype:`

Searches for files of a specific format.

```
filetype:docx hacker
filetype:pdf "confidential"
filetype:xlsx employee data
```

🔍 **Result:** Returns indexed documents matching the given file type and keyword.

---

### 🔹 Operator 4: `site:`

Limits results to a specific domain.

```
site:certifiedhacker.com
site:gov.in filetype:pdf "internal"
```

🔍 **Result:** Returns results only from the specified domain.

---

### 🔹 Operator 5: `allintitle:`

Returns pages where **all** specified keywords appear in the title.

```
allintitle: trojan definition
```

🔍 **Result:** Returns pages with both `trojan` AND `definition` in the title.

---

## 📋 Combined / Advanced Dorks

| Dork | Purpose |
|------|---------|
| `intitle:"index of" "parent directory"` | Find open directory listings |
| `inurl:admin filetype:php` | Find admin login pages |
| `intext:"DB_PASSWORD" filetype:env` | Find exposed `.env` config files |
| `intitle:"index of" "config.php"` | Find PHP config backups |
| `site:pastebin.com "password"` | Find leaked credentials on Pastebin |
| `intitle:"Login" inurl:admin` | Find login portals |
| `cache:example.com` | View Google's cached version of a site |
| `related:example.com` | Find sites related to the target |
| `link:example.com` | Find pages linking to the target |
| `"powered by" "phpmyadmin"` | Find phpMyAdmin panels |

---

## 📋 Additional Useful Operators

| Operator | Description |
|----------|-------------|
| `intext:` | Searches body text of pages |
| `allintext:` | All keywords must appear in body text |
| `allinurl:` | All keywords must appear in URL |
| `define:` | Returns dictionary definition |
| `OR` / `\|` | Either of two terms |
| `-` | Excludes a keyword (e.g., `hacker -movie`) |
| `"..."` | Searches for exact phrase |
| `*` | Wildcard (matches any word) |

---

## 🧠 Real-World Examples

### Find exposed log files
```
filetype:log inurl:"/var/log"
```

### Find indexed backup files
```
intitle:"index of" "backup"
```

### Find vulnerable webcams
```
inurl:"view/index.shtml"
```

### Find SQL dumps
```
filetype:sql "INSERT INTO" "password"
```

> 💡 The **Google Hacking Database (GHDB)** maintained on [Exploit-DB](https://www.exploit-db.com/google-hacking-database) has thousands of categorized dorks.

---

## 🛡️ How Organizations Can Protect Against Google Dorking

| Defense | Description |
|---------|-------------|
| `robots.txt` | Tell Google not to index sensitive paths |
| Authentication | Require login for sensitive resources |
| Directory listing OFF | Disable Apache/Nginx directory indexing |
| Search Console Removal | Request removal of indexed pages from Google |
| Periodic Audits | Use the same dorks to audit your own site |

---

## 📌 References

- [Google Advanced Operators Reference](http://www.googleguide.com/advanced_operators_reference.html)
- [Google Hacking Database (GHDB) – Exploit-DB](https://www.exploit-db.com/google-hacking-database)
- [Exploit-DB Official Site](https://www.exploit-db.com)
- [OSINT Framework](https://osintframework.com/)

---

## 🛑 Ethical Use Warning

Google Dorks reveal **only publicly indexed content** — they do not "hack" anything. However:

- ❌ Do **NOT** download or access exposed sensitive files belonging to others.
- ❌ Do **NOT** attempt to log into exposed admin panels.
- ✅ Use these techniques **only on your own assets** or with **explicit written authorization**.
