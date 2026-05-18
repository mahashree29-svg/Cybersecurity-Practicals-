# Practical 5: Installing VPNBook in Parrot Security Linux

## 📖 Description

This practical demonstrates how to install and configure **VPNBook**, a free OpenVPN service, on **Parrot Security Linux**. A VPN (Virtual Private Network) is used to hide your real IP address and route your traffic through a remote server, providing anonymity and privacy while browsing the internet.

When performing security testing or ethical hacking activities, it is important to mask your identity to avoid exposing your real IP address. VPNBook is a free, open-source VPN solution that helps achieve this on a personal computer.

> ⚠️ **Disclaimer:** This practical is intended for **educational purposes only**. Use VPN services responsibly and in compliance with your local laws and regulations.

---

## 🎯 Objective

- Understand the concept and purpose of a VPN.
- Learn how to download and configure VPNBook on Linux.
- Use OpenVPN to establish a secure tunnel and surf the internet anonymously.

---

## 🛠️ Tools Required

| Tool | Description |
|------|-------------|
| Parrot Security Linux (or any Linux distro) | Operating System |
| OpenVPN | Client used to connect to VPN servers |
| VPNBook | Free OpenVPN service provider |
| Web Browser | To verify the new IP address |

---

## 📚 What is a VPN?

A **Virtual Private Network (VPN)** establishes a virtual point-to-point connection that allows users to send and receive data across public networks as if their devices were directly connected to a private network. This is achieved through encryption and tunneling protocols.

**VPNBook** is one such free, open-source VPN service that allows users to run a VPN on their personal computer without any registration.

---

## 🚀 Step-by-Step Procedure

### Step 1: Download the VPNBook OpenVPN Bundle

1. Open your browser and visit the official VPNBook website:
   👉 [https://www.vpnbook.com/freevpn](https://www.vpnbook.com/freevpn)

2. Under the **Free OpenVPN** section, download any of the available bundles.
   - **Recommended:** `Euro 1 OpenVPN Certificate Bundle` or `Euro 2 OpenVPN Certificate Bundle`.

3. Open a terminal and navigate to the `Downloads` directory:

   ```bash
   cd ~/Downloads
   ```

4. Extract the downloaded VPN bundle using the `unzip` command:

   ```bash
   unzip VPNBook.com-OpenVPN-Euro1.zip
   ```

---

### Step 2: List the Extracted `.ovpn` Files

After extraction, list the contents of the directory to view the available OpenVPN configuration files:

```bash
ls
```

You should see files similar to:

```
vpnbook-euro1-tcp80.ovpn
vpnbook-euro1-tcp443.ovpn
vpnbook-euro1-udp25000.ovpn
vpnbook-euro1-udp53.ovpn
```

---

### Step 3: Connect Using an `.ovpn` File

Select any `.ovpn` file (for example, `vpnbook-pl226-tcp443.ovpn`) and run the following command using `openvpn`:

```bash
sudo openvpn vpnbook-pl226-tcp443.ovpn
```

> 💡 If OpenVPN is not installed, install it first using:
> ```bash
> sudo apt update
> sudo apt install openvpn -y
> ```

---

### Step 4: Enter the VPNBook Credentials

When prompted, you must enter the username and password provided by VPNBook.

- These credentials change regularly and can be found on the **VPNBook website**:
  👉 [https://www.vpnbook.com/freevpn](https://www.vpnbook.com/freevpn)

Look for the section showing the current **Username** and **Password**.

---

### Step 5: Verify the VPN Connection

After entering the credentials, wait until the terminal displays:

```
Initialization Sequence Completed
```

This message indicates that the VPN tunnel has been successfully established.

You can now open any browser and surf the internet anonymously.

✅ To confirm your new IP address, visit:
👉 [https://www.whatismyipaddress.com](https://www.whatismyipaddress.com)

You should see an IP address different from your real one — typically from the country corresponding to the VPN server you selected.

---

## 📸 Sample Output

```
Mon May 18 10:35:42 2026 TCP/UDP: Preserving recently used remote address
Mon May 18 10:35:42 2026 Socket Buffers: R=[131072->131072] S=[16384->16384]
Mon May 18 10:35:42 2026 Attempting to establish TCP connection with [AF_INET]...
Mon May 18 10:35:43 2026 TCP connection established with [AF_INET]...
Mon May 18 10:35:44 2026 [vpnbook.com] Peer Connection Initiated
Mon May 18 10:35:46 2026 Initialization Sequence Completed
```

---

## 🧠 Key Learnings

- Understood how VPNs provide anonymity by masking the real IP address.
- Learned to install and configure VPNBook on Parrot Security Linux.
- Gained practical experience with the `openvpn` command-line client.
- Verified the change of IP using an online IP-checking tool.

---

## 🛑 Troubleshooting

| Issue | Solution |
|-------|----------|
| `openvpn: command not found` | Install OpenVPN: `sudo apt install openvpn` |
| `AUTH_FAILED` error | Check for updated credentials on the VPNBook website |
| Connection timeout | Try a different `.ovpn` file (UDP/TCP variant) |
| DNS leak | Configure DNS manually or use a tool like `resolvconf` |

---

## 📌 References

- [VPNBook Official Website](https://www.vpnbook.com/freevpn)
- [OpenVPN Documentation](https://openvpn.net/community-resources/)
- [Parrot Security OS](https://www.parrotsec.org/)

---

## 👨‍💻 Author

**Practical performed as part of the Ethical Hacking / Cybersecurity Lab.**

> Maintained as a reference for educational and learning purposes.

---

## 📝 License

This project is released under the [MIT License](LICENSE) — feel free to use and modify for educational purposes.
