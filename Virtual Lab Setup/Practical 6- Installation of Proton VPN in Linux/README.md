# Practical 6- Installation of Proton VPN in Linux

## 📖 Description

This practical demonstrates how to install and configure **ProtonVPN** on a Linux system for the purpose of **spoofing the IP address** and browsing the internet anonymously. ProtonVPN is a secure, privacy-focused VPN service developed by the team behind ProtonMail, and it offers a free tier suitable for basic anonymity needs.

> ⚠️ **Disclaimer:** This practical is intended for **educational purposes only**. Use VPN services responsibly and in compliance with your local laws and regulations.

---

## 🎯 Objective

- Understand how to set up and configure a commercial VPN service on Linux.
- Learn to use the ProtonVPN CLI (Command Line Interface) tool.
- Connect to a remote VPN server and verify the spoofed IP address.

---

## 🛠️ Tools Required

| Tool | Description |
|------|-------------|
| Linux OS (Parrot / Kali / Ubuntu) | Operating System |
| ProtonVPN Account | Free or Paid account on protonvpn.com |
| Python 3 & pip3 | Required to install the CLI tool |
| Web Browser | For account registration and verification |

---

## 🚀 Step-by-Step Procedure

### Step 1: Visit the ProtonVPN Website

1. Open your browser and visit:
   👉 [https://protonvpn.com](https://protonvpn.com)

2. Click on the **GET PROTONVPN NOW** button.

---

### Step 2: Select a Plan

Choose a plan that suits your requirement. For this practical, we will use the **Free** plan.

- Click on the **Free** option to proceed.

---

### Step 3: Create a ProtonVPN Account

You can register using:
- A permanent email address, **or**
- A temporary email service such as [YopMail](https://yopmail.com) *(note: some temporary email services may not be accepted)*.

Fill in the registration form with the required details.

---

### Step 4: Email Verification

ProtonVPN will send a verification code to the email address you provided during registration.

- Open your inbox (or YopMail inbox) and copy the verification code.

---

### Step 5: Verify Your Account

- Paste the verification code into the ProtonVPN registration page.
- Click the **Verify** button to complete the account creation.

---

### Step 6: Download the Linux Version

After successful registration, you will be redirected to the **Downloads** section.

- Select **GNU/Linux** to view the Linux installation instructions.

---

### Step 7: Read the Installation Instructions

The official ProtonVPN documentation page will display step-by-step instructions for installing the **ProtonVPN CLI** tool on Linux.

---

### Step 8: Install ProtonVPN CLI

Open a terminal and execute the following command to install the ProtonVPN command-line tool:

```bash
pip3 install protonvpn-cli
```

> 💡 If `pip3` is not installed, install it first using:
> ```bash
> sudo apt update
> sudo apt install python3-pip -y
> ```

---

### Step 9: Initialize ProtonVPN

Run the following command to begin configuring the VPN service:

```bash
sudo protonvpn init
```

---

### Step 10: Retrieve OpenVPN Credentials

The CLI will prompt for a **username** and **password**. These are **NOT** the same as your account login credentials.

To find them:
1. Log in to your ProtonVPN account on the website.
2. Navigate to the **OpenVPN / IKEv2 Username** section in account settings.
3. Copy the **OpenVPN username** and **password** shown there.

---

### Step 11: Enter the Credentials

Paste the OpenVPN username and password into the terminal when prompted.

```
Enter your ProtonVPN OpenVPN username: <your-openvpn-username>
Enter your ProtonVPN OpenVPN password: <your-openvpn-password>
```

---

### Step 12: Select Your Account Plan

Choose the plan tier that matches your registration:

```
1) Free
2) Basic
3) Plus
4) Visionary
```

For this practical, select option **1) Free**.

---

### Step 13: Select the Connection Protocol

Choose your preferred VPN protocol:

```
1) UDP (Recommended)
2) TCP
```

> 💡 **UDP** is faster and recommended for most use cases.
> **TCP** is more reliable for unstable networks.

---

### Step 14: Confirm the Configuration

The CLI will display a summary of all the information you have entered.

- If the details are correct, press **Y** to confirm.
- ProtonVPN configuration will be saved successfully.

---

### Step 15: Connect to a VPN Server

To start the VPN service and connect to a random free server, run:

```bash
sudo protonvpn connect -r
```

> 🔹 The `-r` flag means **random server selection**.

---

### Step 16: Verify the New IP Address

To confirm that the VPN tunnel is active, check your public IP using one of the following methods:

```bash
curl ifconfig.me
```

Or visit:
👉 [https://www.whatismyipaddress.com](https://www.whatismyipaddress.com)

You should see an IP address different from your real one — indicating that the VPN tunnel has been successfully established.

---

### Step 17: Useful ProtonVPN CLI Commands

| Command | Description |
|---------|-------------|
| `sudo protonvpn connect` | Connect to a server (interactive menu) |
| `sudo protonvpn connect -r` | Connect to a random server |
| `sudo protonvpn connect -f` | Connect to the fastest available server |
| `sudo protonvpn connect --p2p` | Connect to a P2P-supported server |
| `sudo protonvpn connect --sc` | Connect to a Secure Core server |
| `sudo protonvpn connect --cc <country>` | Connect to a server in a specific country (e.g. `US`, `IN`) |
| `sudo protonvpn disconnect` | Disconnect from the VPN |
| `sudo protonvpn status` | Show current VPN connection status |
| `sudo protonvpn reconnect` | Reconnect to the last used server |
| `sudo protonvpn configure` | Modify the VPN configuration |
| `sudo protonvpn refresh` | Refresh the server list |
| `sudo protonvpn examples` | Display example commands |

---

## 📸 Sample Output

```
$ sudo protonvpn connect -r
Connecting to NL-FREE#3 via UDP...
Successfully connected to NL-FREE#3.

$ curl ifconfig.me
185.159.157.XX
```

---

## 🧠 Key Learnings

- Learned how to register and configure a ProtonVPN account.
- Gained practical experience installing the ProtonVPN CLI on Linux.
- Understood the difference between account credentials and OpenVPN credentials.
- Used CLI commands to connect, disconnect, and manage VPN sessions.
- Verified IP spoofing using public IP-checking tools.

---

## 🛑 Troubleshooting

| Issue | Solution |
|-------|----------|
| `pip3: command not found` | Install pip: `sudo apt install python3-pip` |
| `Permission denied` | Run commands with `sudo` |
| Authentication failed | Re-check OpenVPN credentials from your ProtonVPN dashboard |
| `protonvpn: command not found` | Add `~/.local/bin` to PATH or reinstall using `sudo pip3 install protonvpn-cli` |
| Connection drops frequently | Switch protocol from UDP to TCP using `protonvpn configure` |

---

## 📌 References

- [ProtonVPN Official Website](https://protonvpn.com)
- [ProtonVPN CLI GitHub Repository](https://github.com/ProtonVPN/linux-cli)
- [ProtonVPN Linux Installation Guide](https://protonvpn.com/support/linux-vpn-tool/)

---

## 👨‍💻 Author

**Practical performed as part of the Ethical Hacking / Cybersecurity Lab.**

> Maintained as a reference for educational and learning purposes.

---

## 📝 License

This project is released under the [MIT License](LICENSE) — feel free to use and modify for educational purposes.
