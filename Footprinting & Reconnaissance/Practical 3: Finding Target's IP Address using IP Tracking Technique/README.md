# Practical 3: Finding Target's IP Address using IP Tracking Technique

## 📖 Description

This practical demonstrates how to convert a normal URL (any web link) into a **trackable link** using the **Grabify IP Logger**. When the target clicks the link, it redirects them to a harmless page (e.g., a YouTube video), while simultaneously logging their public information.

The information captured typically includes:

- 🌐 **Public IP address**
- 📱 **Device type** (mobile / desktop)
- 💻 **Operating System**
- 🌍 **Approximate geographic location** (based on ISP)
- 🔍 **Browser & User-Agent**
- 📶 **Internet Service Provider (ISP)**

This technique demonstrates **client-side reconnaissance** and shows how attackers can use social engineering with seemingly harmless links to gather information.

> ⚠️ **Disclaimer:** This practical is for **educational purposes only**. Do **NOT** use this technique against anyone without their **explicit consent**. Unauthorized tracking may violate privacy laws (e.g., GDPR, IT Act 2000 in India).

---

## 🎯 Objective

- Understand how URL-based IP logging works.
- Generate a tracking link using Grabify.
- Analyze the data captured when a target clicks the link.

---

## 🛠️ Requirements

| Tool | Description |
|------|-------------|
| Web Browser | Firefox / Chrome / Brave |
| Internet Connection | Required |
| Target URL | Any safe URL (YouTube, Wikipedia, etc.) |

---

## 🚀 Step-by-Step Procedure

### Step 1: Visit Grabify IP Logger

Open your browser and go to:
👉 [https://grabify.link/](https://grabify.link/)

This is a free service that creates **tracking redirect links** that capture visitor information.

#### 🔹 How to Create a Tracking Link
1. In the input box, paste any **legitimate URL** (the destination the target will be redirected to).
   - Example: `https://www.youtube.com/watch?v=dQw4w9WgXcQ`
2. Click on the **"Create URL"** button.

> 💡 The target will see the YouTube video as the final landing page, while Grabify silently records their data in the background.

---

### Step 2: Receive the Tracking URL

After clicking **Create URL**:

- Grabify generates a **new shortened tracking link** under the **"New URL"** section.
- Example tracking link:
  ```
  https://grabify.link/AB1CD2
  ```
- It also provides a **Tracking Code** (e.g., `AB1CD2`) that you can use later to view the captured logs.

> 💡 You can further shorten or rename the link using services like **TinyURL** or **Bit.ly** to make it look less suspicious during demonstrations.

---

### Step 3: Capture Target Information

When the target clicks on the tracking link:

1. They are seamlessly **redirected** to the original destination (YouTube video).
2. Grabify logs the visit and displays the captured information on the **Tracking Page**.

#### 🔍 Information Displayed
- IP Address
- Country / City
- ISP
- Browser & User-Agent
- Device / Operating System
- Timestamp of click

> 💡 To view the logs later, go back to Grabify and enter the **Tracking Code** in the "Tracking Code" box on the homepage.

---

## 📋 Sample Captured Data

```
IP Address:     203.0.113.45
Country:        India
City:           Bengaluru
ISP:            Airtel Broadband
Device:         Windows 10 / Desktop
Browser:        Chrome 125.0.0
User-Agent:     Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36
Referrer:       https://t.co/xxxxx
Timestamp:      2026-05-18 14:32:11 UTC
```

---

## 🧠 Key Learnings

- Understood how **redirect-based tracking links** work.
- Recognized the dangers of clicking on unknown shortened URLs.
- Learned how attackers can perform **passive OSINT** through social engineering.

---

## 🛡️ How to Protect Yourself from IP Tracking Links

| Method | Description |
|--------|-------------|
| 🔍 **Inspect the URL** | Hover over links before clicking to see the real destination |
| 🌐 **Use a VPN** | Hides your real IP from tracking services |
| 🧅 **Use Tor Browser** | Routes traffic through multiple relays for anonymity |
| 🛠️ **Use URL Unshorteners** | Tools like [unshorten.it](https://unshorten.it) reveal the destination |
| 🚫 **Avoid suspicious links** | Especially from unknown sources on social media |

---

## 🛑 Alternative IP Logging Services

| Service | Notes |
|---------|-------|
| [IPLogger.org](https://iplogger.org) | Similar functionality, supports image logging |
| [Blasze](https://blasze.com) | Lightweight URL tracker |
| [Grabify](https://grabify.link) | Most feature-rich (this practical) |

---

## 📌 References

- [Grabify Official Site](https://grabify.link/)
- [OSINT Framework](https://osintframework.com/)
- [URL Shortener Privacy Concerns](https://www.eff.org/)
