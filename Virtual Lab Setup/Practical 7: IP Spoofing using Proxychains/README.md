# Practical 7- IP Spoofing using Proxychains

## 📖 Description

This practical demonstrates how to configure and use the **Proxychains** tool to **spoof your IP address** while running any application or command-line tool on Linux. Proxychains routes your traffic through a chain of proxy servers (HTTP, SOCKS4, SOCKS5) — or through the **Tor network** — so that the destination server never sees your real IP address.

This technique is widely used during **penetration testing**, **OSINT investigations**, and **anonymous browsing** to avoid revealing the tester's true identity.

> ⚠️ **Disclaimer:** This practical is intended for **educational and authorized penetration-testing purposes only**. Do not use these techniques on systems or networks you do not own or have explicit permission to test.

---

## 🎯 Objective

- Understand the working of the Proxychains tool.
- Learn the difference between **dynamic**, **strict**, and **random** chaining modes.
- Configure Proxychains to route traffic through the **Tor network**.
- Run common tools like `firefox` and `nmap` anonymously through proxy chains.

---

## 🛠️ Tools Required

| Tool | Description |
|------|-------------|
| Parrot Security Linux / Kali Linux | Operating System (Proxychains comes pre-installed) |
| Proxychains | Proxy-chaining tool |
| Tor | Anonymity network used as a backend proxy |
| Firefox / Nmap | Sample applications used with Proxychains |

---

## 📚 What is Proxychains?

**Proxychains** is a UNIX tool that forces any TCP connection made by a given application to go through a chain of proxy servers — including SOCKS4, SOCKS5, HTTP/HTTPS proxies, or the Tor network.

It works by hooking network-related libcalls of dynamically linked programs, so the target application is unaware that its traffic is being tunneled through proxies.

---

## 🚀 Step-by-Step Procedure

### Step 1: Understanding the Proxychains Configuration

Proxychains is available by default on **Parrot Security Linux** and **Kali Linux**.

The proxy IP addresses are added at the end of the configuration file at `/etc/proxychains.conf` (or `/etc/proxychains4.conf` on newer systems) using the following syntax:

```
<protocol>  <ip_address>  <port>  [user]  [password]
```

**Example:**
```
https  184.34.234.45   2764
socks5 23.45.79.163    8434
```

Supported protocols:
- `http`
- `https`
- `socks4`
- `socks5`

---

### Step 2: Open and Edit the Configuration File

Open the configuration file using any text editor with `sudo` privileges:

```bash
sudo nano /etc/proxychains.conf
```

> 💡 You can also use `vim`, `gedit`, or `mousepad` instead of `nano`.

---

### Step 3: Choose a Chaining Option

Inside the configuration file, there are **three chaining modes** available. Only **one** can be active at a time — uncomment the one you want to use and comment out the others.

#### 🔹 1. `dynamic_chain`
- Routes traffic through **every proxy** in the list, in order.
- If any proxy is dead or unresponsive, it is **skipped silently** and the next one is used.
- ✅ **Recommended** — most reliable option.

#### 🔹 2. `strict_chain`
- Uses every proxy in the **exact order** listed.
- If any proxy in the chain fails, the **entire chain breaks** and no connection is established.

#### 🔹 3. `random_chain`
- Picks proxies **randomly** from the list.
- The IP chain changes every time you run Proxychains.

#### 🔹 4. `chain_len`
- Used along with `random_chain`.
- Defines **how many proxies** from the list should be used in the random chain.
- Uncomment the line and set a numeric value:

```
chain_len = 3
```

**Example configuration:**
```
# dynamic_chain     <-- commented out
# strict_chain
random_chain
chain_len = 3
```

---

### Step 4: Configure Proxychains to Use the Tor Network

To make Proxychains route traffic through the **Tor network**, add the following lines at the **end** of the configuration file under the `[ProxyList]` section:

```
socks4 127.0.0.1 9050
socks5 127.0.0.1 9050
```

> 💡 Port `9050` is the default SOCKS port used by the Tor service.

Save and close the file (`CTRL + O`, then `CTRL + X` in nano).

---

### Step 5: Check Your Original IP Address

Before using Proxychains, verify your **current public IP** to compare it later.

Open Firefox and visit:
👉 [https://www.whatismyipaddress.com](https://www.whatismyipaddress.com)

Note down the displayed IP address.

---

### Step 6: Start the Tor Service

Start the Tor service so that Proxychains can route traffic through it:

```bash
sudo service tor start
```

To verify that Tor is running:

```bash
sudo service tor status
```

> ⚠️ **Important:** Before running an application through Proxychains, **close all previous instances** of that application. For example, if you want to run Proxychains with Firefox, close all open Firefox windows first.

---

#### 🔸 Syntax to Run an Application via Proxychains

```bash
proxychains <application-name> [arguments]
```

#### 🔸 Examples

Run Firefox anonymously:
```bash
proxychains firefox
```

Run Nmap anonymously:
```bash
proxychains nmap -sT -Pn <target-ip>
```

> 💡 When using Nmap with Proxychains, use **`-sT` (TCP Connect Scan)** and **`-Pn`** since Proxychains only supports TCP traffic (no ICMP/UDP).

---

### Step 7: Verify the Spoofed IP Address

After Firefox opens through Proxychains:

1. Visit:
   👉 [https://www.whatismyipaddress.com](https://www.whatismyipaddress.com)

2. You should see a **completely different IP address**, often from a different country — confirming that your traffic is being routed through the Tor network or the proxy chain.

✅ **Success!** Your IP has been successfully spoofed.

---

## 📸 Sample Output

```
$ proxychains firefox
ProxyChains-3.1 (http://proxychains.sf.net)
|DNS-request| www.whatismyipaddress.com
|S-chain|-<>-127.0.0.1:9050-<><>-4.2.2.2:53-<><>-OK
|DNS-response| www.whatismyipaddress.com is 104.16.155.36
|S-chain|-<>-127.0.0.1:9050-<><>-104.16.155.36:443-<><>-OK
```

---

## 🧠 Key Learnings

- Understood how Proxychains routes traffic through multiple proxy hops.
- Learned the differences between `dynamic_chain`, `strict_chain`, and `random_chain`.
- Configured Proxychains to use the **Tor network** as a default proxy.
- Successfully spoofed the public IP address while using Firefox and Nmap.
- Reinforced the importance of operational security (OpSec) during pen-testing.

---

## 🛑 Troubleshooting

| Issue | Solution |
|-------|----------|
| `proxychains: command not found` | Install it: `sudo apt install proxychains -y` |
| `Cannot connect to SOCKS proxy 127.0.0.1:9050` | Start Tor: `sudo service tor start` |
| Firefox does not show a new IP | Close ALL existing Firefox windows first, then re-run via Proxychains |
| `Permission denied` editing config | Use `sudo` when opening `/etc/proxychains.conf` |
| Slow browsing speed | Normal behavior — Tor adds latency; reduce `chain_len` if using `random_chain` |
| DNS leaks | Ensure `proxy_dns` is **uncommented** in `/etc/proxychains.conf` |

---

## 🔐 Best Practices

- ✅ Always close existing application instances before running them via Proxychains.
- ✅ Use `dynamic_chain` for reliability during pen-tests.
- ✅ Keep `proxy_dns` enabled to avoid DNS leaks.
- ✅ Combine Proxychains with **Tor** for maximum anonymity.
- ❌ Do not use Proxychains for malicious or unauthorized activities.

---

## 📌 References

- [Proxychains-NG GitHub Repository](https://github.com/rofl0r/proxychains-ng)
- [Tor Project Official Website](https://www.torproject.org)
- [Parrot Security OS](https://www.parrotsec.org/)
- [WhatIsMyIPAddress](https://www.whatismyipaddress.com)

---

## 👨‍💻 Author

**Practical performed as part of the Ethical Hacking / Cybersecurity Lab.**

> Maintained as a reference for educational and learning purposes.

---

## 📝 License

This project is released under the [MIT License](LICENSE) — feel free to use and modify for educational purposes.
