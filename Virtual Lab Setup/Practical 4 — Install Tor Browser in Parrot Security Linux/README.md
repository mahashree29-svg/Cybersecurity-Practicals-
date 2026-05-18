# Practical 4 — Install Tor Browser in Parrot Security Linux

This practical covers installing the **Tor Browser** in **Parrot Security Linux**. Tor (The Onion Router) is an anonymity-focused browser used to surf the internet privately and to access `.onion` sites on the deep web and dark web that are not reachable through normal browsers.

---

## ⚠️ Important Warnings — Read Before Proceeding

> 🛡️ **Anonymity is not guaranteed.** Tor protects your traffic in transit, but it does not protect you from:
> - Logging into accounts that identify you
> - Browser fingerprinting via plugins, resizing the window, or installing extensions
> - Downloading and opening files outside of Tor (especially documents that fetch remote resources)
> - Malware already present on your system
>
> Read the official [Tor Browser warnings and best practices](https://support.torproject.org/tbb/) before relying on it for anything sensitive.

> ⚖️ **Legal notice.** Use of Tor is legal in most countries, but accessing certain content on the dark web is not. This practical is for **educational and lawful research purposes only**. The author assumes no responsibility for misuse.

---

## 🧰 Tools Required

| Tool | Purpose |
|------|---------|
| Parrot Security Linux | Host operating system |
| Tor Browser (`.tar.xz`) | Anonymity-focused web browser |

**Download link:** [https://www.torproject.org/download/](https://www.torproject.org/download/)

> 💡 Always verify the PGP signature of the Tor Browser download before extracting. Instructions: [How to verify Tor Browser's signature](https://support.torproject.org/tbb/how-to-verify-signature/).

---

## 🚀 Installation Steps

### Step 1 — Download Tor Browser

1. Open a normal browser inside Parrot Security Linux.
2. Visit the official download page: <https://www.torproject.org/download/>
3. Download the Linux `.tar.xz` package. By default it is saved to `~/Downloads`.

### Step 2 — Extract the Archive

Open a terminal and switch into the Downloads directory:

```bash
cd ~/Downloads
```

Extract the downloaded archive (replace the filename with the version you downloaded):

```bash
tar -xvJf tor-browser-linux64-*.tar.xz
```

Move into the extracted folder:

```bash
cd tor-browser
```

### Step 3 — Launch Tor Browser

From inside the `tor-browser` directory, run the launcher:

```bash
./start-tor-browser.desktop
```

A **Tor Network Settings** window will appear. Click **Connect** to establish a connection to the Tor network. Once the connection is complete, the Tor Browser window will open and you can begin browsing anonymously.

---


## 🧠 Best Practices While Using Tor

- **Do not maximize the browser window** — it makes you easier to fingerprint.
- **Do not install extra add-ons** — they break the standardized fingerprint.
- **Avoid logging into personal accounts** over Tor unless you understand the implications.
- **Keep Tor Browser updated** — patches are critical for anonymity and security.
- **Do not torrent over Tor** — it leaks your real IP and slows the network for others.
- **Use HTTPS** wherever possible (Tor Browser enforces this by default).

---

## 🩹 Troubleshooting

| Problem | Possible Fix |
|---------|--------------|
| `./start-tor-browser.desktop: Permission denied` | Run `chmod +x start-tor-browser.desktop` |
| Tor fails to connect | Your ISP may block Tor — configure a **bridge** in the connection settings (e.g., `obfs4`) |
| Stuck at "Establishing a Tor circuit" | Check system clock (`timedatectl`) — incorrect time often breaks the handshake |

---

## 📚 References

- [Tor Project — Official Website](https://www.torproject.org/)
- [Tor Browser User Manual](https://tb-manual.torproject.org/)
- [Tor Project — Support Portal](https://support.torproject.org/)
- [Parrot Security OS — Documentation](https://parrotsec.org/docs/)

---

## 📝 License

This repository documents an educational practical for learning purposes only. The author assumes no responsibility for misuse of the tools or techniques described.
