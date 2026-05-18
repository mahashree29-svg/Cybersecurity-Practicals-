# Practical 6: Gathering Information using Archive.org (Wayback Machine)

## 📖 Description

The **Internet Archive's Wayback Machine** ([archive.org](https://archive.org)) is a non-profit digital library that periodically captures and stores **snapshots of websites** across the internet. These historical snapshots provide a powerful resource for **passive reconnaissance** because they allow researchers and ethical hackers to:

- 🕰️ View **previous versions** of any website
- 🔍 Discover **deleted or hidden content** (admin pages, employee directories, exposed files)
- 📂 Find **outdated technologies, old subdomains, or removed disclaimers**
- 🔐 Identify **sensitive information** that was unintentionally exposed in the past but later removed

This is particularly useful when investigating targets that have **redesigned their website** or **removed sensitive information** that may still be archived.

> ⚠️ **Disclaimer:** This practical is intended for **educational and authorized testing purposes only**.

---

## 🎯 Objective

- Learn to use the Wayback Machine to access archived versions of websites.
- Discover information that has been removed from the live version of a target site.
- Compare historical versions of a website over time.

---

## 🛠️ Requirements

| Tool | Description |
|------|-------------|
| Web Browser | Firefox / Chrome / Brave |
| Internet Connection | Required |
| Wayback Machine | [https://archive.org](https://archive.org) |

---

## 🚀 Step-by-Step Procedure

### Step 1: Visit the Wayback Machine

Open your web browser and go to:
👉 [https://archive.org](https://archive.org)

The homepage contains the **Wayback Machine** search bar near the top of the page.

> 💡 You can also access it directly at: [https://web.archive.org](https://web.archive.org)

---

### Step 2: Enter the Target Website URL

In the search bar, type the domain name of the website you want to investigate. For example:

```
hackerschool.in
```

Press **Enter** to view the calendar timeline of archived snapshots.

---

### Step 3: Browse the Calendar by Year

The Wayback Machine displays a **horizontal timeline** of years from when the website was first archived up to the present.

- 📅 Select the **year** you want to investigate.
- A calendar view will appear with **colored circles** on dates that have snapshots.
  - 🔵 **Blue circles** → Successful archive
  - 🟢 **Green circles** → Successful redirect captures
  - 🔴 **Red circles** → Captured errors (404, 500, etc.)

---

### Step 4: View a Specific Snapshot

1. Hover over a highlighted date to see the **exact times** the site was captured.
2. Click on the desired timestamp to open the archived version of the site.

📌 **Example:**
> If you click on `June 25, 2014 → 11:32 AM`, you will see what the website looked like at that exact moment.

---

### Step 5: Explore the Archived Site

Once loaded, you can browse the archived website **just like a live site**. You can:

- 📄 Open old pages (about us, contact, team, etc.)
- 📂 Download archived PDFs and documents
- 🔍 Inspect old source code (right-click → View Source)
- 📧 Find old contact emails or phone numbers
- 🌐 Discover **subdomains** that no longer exist

---

## 📋 Tips for Effective Investigation

### 🔹 Use URL-Specific Searches

You can search for specific paths instead of the homepage:

```
https://web.archive.org/web/*/hackerschool.in/admin
https://web.archive.org/web/*/example.com/employees
```

### 🔹 Use the "Changes" Feature

The Wayback Machine has a `Changes` tab that highlights differences between two snapshots side-by-side. This is excellent for spotting **removed content**.

### 🔹 Use the API

You can query the Wayback Machine programmatically:

```bash
curl "http://archive.org/wayback/available?url=hackerschool.in&timestamp=2014"
```

### 🔹 Try the `Site Map` View

Click **Site Map** in the Wayback Machine UI to see a visualization of all archived pages.

---

## 📋 Useful Wayback Machine URL Patterns

| URL Pattern | Purpose |
|-------------|---------|
| `https://web.archive.org/web/*/example.com/*` | List all archived pages for the domain |
| `https://web.archive.org/web/2018/example.com` | Snapshot closest to 2018 |
| `https://web.archive.org/web/2*/example.com/admin*` | Find archived `/admin` URLs |
| `https://web.archive.org/__wb/sparkline?...` | Visualize crawl history |

---

## 🧠 Key Learnings

- Performed **passive OSINT** using publicly available archive data.
- Discovered how **historical website snapshots** can reveal sensitive information.
- Learned to **compare archived versions** with the live website.
- Understood the importance of properly removing data from search engines and archives.

---

## 🛡️ How Organizations Can Protect Their Data

| Method | Description |
|--------|-------------|
| Request Removal | Email `info@archive.org` to request removal of pages |
| `robots.txt` Exclusion | Add `User-agent: ia_archiver` `Disallow: /` to your robots.txt |
| Audit Old Content | Regularly audit archived versions for leaked information |
| Disclose & Rotate | If old credentials leaked, rotate them immediately |

---

## 📌 Useful Related Tools

| Tool | Purpose |
|------|---------|
| [Waybackurls](https://github.com/tomnomnom/waybackurls) | CLI tool to list all archived URLs of a domain |
| [Gau](https://github.com/lc/gau) | Fetches known URLs from multiple sources |
| [Archive.today](https://archive.ph) | Alternative archive service |
| [Cached View](https://cachedview.com) | Aggregates cached versions from multiple sources |

### Example: Using `waybackurls`

```bash
go install github.com/tomnomnom/waybackurls@latest
echo "hackerschool.in" | waybackurls > archived_urls.txt
```

---

## 📌 References

- [Internet Archive](https://archive.org)
- [Wayback Machine](https://web.archive.org)
- [Wayback Machine API Documentation](https://archive.org/help/wayback_api.php)
- [Waybackurls GitHub](https://github.com/tomnomnom/waybackurls)
