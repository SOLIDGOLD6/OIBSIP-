# Nmap Installation and Basic Scanning Guide

This document walks through installing **Nmap** (with the Zenmap GUI) on Windows, and then demonstrates three basic scan types run against `localhost (127.0.0.1)`.

> **Note:** To view the images below, place the screenshot files in the same folder as this markdown file, using the filenames referenced (`nmap_1.png` through `nmap_6.png`, `scan.png`, `sV.png`, `O.png`).

---

## Part 1: Installing Nmap

### Step 1 – Choose Components
![Choose Components](nmap_1.png)

The Nmap Setup wizard's **Choose Components** screen. Here you select which features to install — by default this includes the **Nmap Core Files**, **Register Nmap Path**, **Network Performance** improvements, **Ncat**, **Nping**, and **Zenmap (GUI Frontend)**. The total space required is shown as 88.1 MB. Click **Next** to continue.

### Step 2 – Choose Install Location
![Choose Install Location](nmap_2.png)

The **Choose Install Location** screen lets you pick the destination folder for Nmap. The default path is `C:\Program Files (x86)\Nmap`. It also displays the space required (88.1 MB) versus the space available (78.7 GB). Clicking **Install** begins the installation process.

### Step 3 – Installing
![Installing](nmap_3.png)

The **Installing** progress screen shows the live extraction status of Nmap's files (in this case, extracting `CHANGELOG` at 93%). A progress bar tracks overall completion, and a **Show details** button reveals a verbose log of the installation steps.

### Step 4 – Installation Complete
![Installation Complete](nmap_4.png)

Once all files have been extracted and installed, the **Installation Complete** screen appears with a fully filled green progress bar, confirming that "Setup was completed successfully." Click **Next** to proceed.

### Step 5 – Create Shortcuts
![Create Shortcuts](nmap_5.png)

The **Create Shortcuts** screen allows you to choose whether to create a **Start Menu Folder** and/or a **Desktop Icon** for quick access to Nmap/Zenmap after installation. Both options are checked by default.

### Step 6 – Setup Finished
![Finished](nmap_6.png)

The final **Finished** screen confirms that "Nmap has been installed on your computer." Clicking **Finish** closes the installation wizard, completing the setup process.

---

## Part 2: Running Scans with Zenmap

With Nmap installed, the **Zenmap** GUI can be used to run scans against a target — in these examples, the local machine (`127.0.0.1`).

### Basic Scan
![Basic Scan](scan.png)

This screenshot shows a basic scan using the command `nmap 127.0.0.1` (Zenmap's "Regular scan" profile). The output lists the open TCP ports found on the host:
- **135/tcp** – open – msrpc
- **445/tcp** – open – microsoft-ds
- **1583/tcp** – open – simbaexpress
- **3351/tcp** – open – btrieve
- **7070/tcp** – open – realserver

The scan completed in 0.83 seconds, confirming the host is up and reporting the 5 open ports out of 1000 scanned (995 are closed).

### Service/Version Detection Scan (-sV)
![Service Version Detection](sV.png)

This screenshot demonstrates the `-sV` flag (`nmap -sV 127.0.0.1`), which performs **service and version detection** on open ports. In addition to the port/state/service columns, Nmap now identifies the specific software and version running behind each port:
- **135/tcp** – Microsoft Windows RPC
- **445/tcp** – microsoft-ds
- **1583/tcp** – Pervasive SQL Server – Relational Engine (encrypted)
- **3351/tcp** – Pervasive SQL Server – Btrieve Engine
- **7070/tcp** – ssl/realserver?

It also reports general **Service Info**, identifying the OS as Windows. This scan took longer (12.87 seconds) than the basic scan, since version probing requires additional interaction with each open port.

### OS Detection Scan (-O)
![OS Detection](O.png)

This screenshot shows the `-O` flag (`nmap -O 127.0.0.1`), which performs **operating system detection**. Along with the same list of open ports, Nmap now provides:
- **Device type:** general purpose
- **Running:** Microsoft Windows 11
- **OS CPE:** `cpe:/o:microsoft:windows_11`
- **OS details:** Microsoft Windows 11 24H2 – 25H2
- **Network Distance:** 0 hops (since the target is the local machine)

This scan completed in 2.49 seconds and used TCP/IP stack fingerprinting to determine the likely operating system of the target host.

---

## Summary

| Scan Type | Command | Purpose |
|---|---|---|
| Basic Scan | `nmap 127.0.0.1` | Lists open/closed ports and services |
| Version Detection | `nmap -sV 127.0.0.1` | Identifies software/version running on each open port |
| OS Detection | `nmap -O 127.0.0.1` | Fingerprints and identifies the target's operating system |
