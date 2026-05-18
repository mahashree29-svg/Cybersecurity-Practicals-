# Practical 4: Footprinting Domain using Recon-ng Tool

## 📖 Description

**Recon-ng** is a powerful, full-featured **web reconnaissance framework** written in Python. It provides a modular environment similar to **Metasploit**, where each module performs a specific OSINT task — such as harvesting subdomains, emails, hostnames, contacts, or social media information.

It is one of the most flexible footprinting tools because of its:

- 🧩 **Modular architecture** (independent modules for each task)
- 🔌 **Marketplace** for installing community modules
- 💾 **Built-in workspace database** for storing findings
- 📊 **Report generation** in multiple formats (HTML, CSV, XML)

> ⚠️ **Disclaimer:** This practical is intended for **educational and authorized testing purposes only**.

---

## 🎯 Objective

- Understand the Recon-ng framework workflow.
- Install and use Recon-ng modules from the marketplace.
- Gather information about a target domain using a module.

---

## 🛠️ Prerequisites

| Tool | Description |
|------|-------------|
| Parrot Security OS / Kali Linux | Pre-installed with Recon-ng |
| `recon-ng` | Web reconnaissance framework |
| Internet connection | Required for module operation |

### 🔧 Installation (if not already installed)

```bash
sudo apt update
sudo apt install recon-ng -y
```

Or install from GitHub:

```bash
git clone https://github.com/lanmaster53/recon-ng.git
cd recon-ng
pip3 install -r REQUIREMENTS
```

---

## 🚀 Step-by-Step Procedure

### Step 1: Launch Recon-ng

Open a terminal and execute:

```bash
recon-ng
```

You will enter the interactive Recon-ng shell:

```
[recon-ng][default] >
```

---

### Step 2: Install Modules from the Marketplace

By default, Recon-ng comes **without any modules pre-installed**. To install all available modules:

```
[recon-ng][default] > marketplace install all
```

> 💡 You can also install a specific module:
> ```
> marketplace install recon/domains-hosts/hackertarget
> ```

---

### Step 3: Search for Available Modules

To list all available modules in the marketplace:

```
[recon-ng][default] > marketplace search
```

To search for a specific keyword (e.g., `hosts`):

```
[recon-ng][default] > marketplace search hosts
```

---

### Step 4: Load a Module

To load a module, use the `modules load` command:

```
[recon-ng][default] > modules load recon/domains-hosts/hackertarget
```

Once loaded, the prompt changes:

```
[recon-ng][default][hackertarget] >
```

---

### Step 5: View Module Options

```
[recon-ng][default][hackertarget] > options list
```

This lists all configurable parameters such as `SOURCE`, `TIMEOUT`, etc.

---

### Step 6: Set the Target Domain

Set the target domain as the SOURCE for the module:

```
[recon-ng][default][hackertarget] > options set SOURCE hackthissite.org
```

> 💡 You can also set the source as `default` to use values from the workspace database.

---

### Step 7: Run the Module

Execute the module to gather information:

```
[recon-ng][default][hackertarget] > run
```

Recon-ng will query the target and store discovered hosts/subdomains in its internal database.

---

### Step 8: View Collected Data

To view collected data, use the `show` command:

```
[recon-ng][default][hackertarget] > show hosts
```

You can also view other tables:

```
show contacts
show emails
show domains
show credentials
```

---

### Step 9: Generate a Report

Load the reporting module:

```
[recon-ng][default] > modules load reporting/html
[recon-ng][default][html] > options set CREATOR YourName
[recon-ng][default][html] > options set CUSTOMER TargetCompany
[recon-ng][default][html] > run
```

This generates a clean HTML report of all findings.

---

## 📋 Useful Recon-ng Commands

| Command | Description |
|---------|-------------|
| `marketplace search` | List all available modules |
| `marketplace install <module>` | Install a specific module |
| `modules load <module>` | Load a module |
| `options list` | View module options |
| `options set <key> <value>` | Configure a module option |
| `run` | Execute the loaded module |
| `show hosts` | View discovered hosts |
| `workspaces create <name>` | Create a new workspace |
| `back` | Exit current module |
| `exit` | Quit Recon-ng |

---

## 📋 Sample Workflow

```
$ recon-ng
[recon-ng][default] > workspaces create hackthissite
[recon-ng][default][hackthissite] > marketplace install recon/domains-hosts/hackertarget
[recon-ng][default][hackthissite] > modules load recon/domains-hosts/hackertarget
[recon-ng][default][hackthissite][hackertarget] > options set SOURCE hackthissite.org
[recon-ng][default][hackthissite][hackertarget] > run
[recon-ng][default][hackthissite][hackertarget] > show hosts
```

---

## 🧠 Key Learnings

- Used Recon-ng's **modular framework** for OSINT.
- Practiced installing and running **marketplace modules**.
- Stored and queried reconnaissance data via Recon-ng's built-in database.
- Generated automated HTML reports.

---

## 🛑 Troubleshooting

| Issue | Solution |
|-------|----------|
| `recon-ng: command not found` | Install: `sudo apt install recon-ng` |
| Module fails with API error | Add API key: `keys add <service>_api <KEY>` |
| `pip dependency error` | Run `pip3 install -r REQUIREMENTS` from source folder |
| Marketplace not loading | Check internet connection / firewall |

---

## 📌 References

- [Recon-ng GitHub](https://github.com/lanmaster53/recon-ng)
- [Recon-ng Wiki](https://github.com/lanmaster53/recon-ng/wiki)
- [Parrot Security OS](https://www.parrotsec.org/)
