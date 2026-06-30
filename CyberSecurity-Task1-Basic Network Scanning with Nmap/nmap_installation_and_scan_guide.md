# Nmap Installation and Basic Scanning Guide

This document walks through installing **Nmap** (with the Zenmap GUI) on Windows, and then demonstrates three basic scan types run against `localhost (127.0.0.1)`.

> **Note:** To view the images below, place the screenshot files in the same folder as this markdown file, using the filenames referenced (`nmap_1.png` through `nmap_6.png`, `scan.png`, `sV.png`, `O.png`).

---

## Part 1: Installing Nmap

### Step 1 – Choose Components
<img width="669" height="492" alt="nmap 1" src="https://github.com/user-attachments/assets/13d82075-8729-4840-b48d-66085763186d" />


The Nmap Setup wizard's **Choose Components** screen. Here you select which features to install — by default this includes the **Nmap Core Files**, **Register Nmap Path**, **Network Performance** improvements, **Ncat**, **Nping**, and **Zenmap (GUI Frontend)**. The total space required is shown as 88.1 MB. Click **Next** to continue.

### Step 2 – Choose Install Location
<img width="558" height="431" alt="nmap 2" src="https://github.com/user-attachments/assets/5c6d8e17-671d-4be5-bba0-05466fe1518e" />


The **Choose Install Location** screen lets you pick the destination folder for Nmap. The default path is `C:\Program Files (x86)\Nmap`. It also displays the space required (88.1 MB) versus the space available (78.7 GB). Clicking **Install** begins the installation process.

### Step 3 – Installing
<img width="553" height="434" alt="nmap 3" src="https://github.com/user-attachments/assets/d449ca65-c7a5-4ea0-adbc-33255a4de88c" />


The **Installing** progress screen shows the live extraction status of Nmap's files (in this case, extracting `CHANGELOG` at 93%). A progress bar tracks overall completion, and a **Show details** button reveals a verbose log of the installation steps.

### Step 4 – Installation Complete
<img width="573" height="449" alt="nmap 4" src="https://github.com/user-attachments/assets/b1e26d07-f152-4224-ad9d-0ee6acbb8868" />


Once all files have been extracted and installed, the **Installation Complete** screen appears with a fully filled green progress bar, confirming that "Setup was completed successfully." Click **Next** to proceed.

### Step 5 – Create Shortcuts
<img width="552" height="433" alt="nmap 5" src="https://github.com/user-attachments/assets/ace4fa55-9ed6-4751-a9f3-f14862f292a3" />


The **Create Shortcuts** screen allows you to choose whether to create a **Start Menu Folder** and/or a **Desktop Icon** for quick access to Nmap/Zenmap after installation. Both options are checked by default.

### Step 6 – Setup Finished
<img width="561" height="436" alt="nmap 6" src="https://github.com/user-attachments/assets/a642c33f-1e81-4fbb-a8da-f5341b61e101" />

The final **Finished** screen confirms that "Nmap has been installed on your computer." Clicking **Finish** closes the installation wizard, completing the setup process.

---

## Part 2: Running Scans with Zenmap

With Nmap installed, the **Zenmap** GUI can be used to run scans against a target — in these examples, the local machine (`127.0.0.1`).

### Basic Scan
<img width="758" height="641" alt="scan" src="https://github.com/user-attachments/assets/221235cc-df96-4993-a8ed-af74ed22e9fc" />


This screenshot shows a basic scan using the command `nmap 127.0.0.1` (Zenmap's "Regular scan" profile). The output lists the open TCP ports found on the host:
- **135/tcp** – open – msrpc
- **445/tcp** – open – microsoft-ds
- **1583/tcp** – open – simbaexpress
- **3351/tcp** – open – btrieve
- **7070/tcp** – open – realserver

The scan completed in 0.83 seconds, confirming the host is up and reporting the 5 open ports out of 1000 scanned (995 are closed).

### Service/Version Detection Scan (-sV)
<img width="756" height="641" alt="sV" src="https://github.com/user-attachments/assets/d445a301-952c-453f-9125-29247fbd5261" />


This screenshot demonstrates the `-sV` flag (`nmap -sV 127.0.0.1`), which performs **service and version detection** on open ports. In addition to the port/state/service columns, Nmap now identifies the specific software and version running behind each port:
- **135/tcp** – Microsoft Windows RPC
- **445/tcp** – microsoft-ds
- **1583/tcp** – Pervasive SQL Server – Relational Engine (encrypted)
- **3351/tcp** – Pervasive SQL Server – Btrieve Engine
- **7070/tcp** – ssl/realserver?

It also reports general **Service Info**, identifying the OS as Windows. This scan took longer (12.87 seconds) than the basic scan, since version probing requires additional interaction with each open port.

### OS Detection Scan (-O)
<img width="759" height="639" alt="O" src="https://github.com/user-attachments/assets/c5205aa8-5b72-4e42-92b8-cddfa08f83a5" />


This screenshot shows the `-O` flag (`nmap -O 127.0.0.1`), which performs **operating system detection**. Along with the same list of open ports, Nmap now provides:
- **Device type:** general purpose
- **Running:** Microsoft Windows 11
- **OS CPE:** `cpe:/o:microsoft:windows_11`
- **OS details:** Microsoft Windows 11 24H2 – 25H2
- **Network Distance:** 0 hops (since the target is the local machine)

This scan completed in 2.49 seconds and used TCP/IP stack fingerprinting to determine the likely operating system of the target host.

---
**Port Analysis**
**Port 135/TCP**

**Detected Service**

Microsoft Windows RPC (Remote Procedure Call)

**Purpose**

Used by Windows for communication between software components and services.
Required for many Windows management tasks.

**Risk Level**

Medium

**Potential Threats**

Frequently targeted during network reconnaissance.
Can expose information about Windows services.
Historically associated with remote code execution vulnerabilities when exposed to untrusted networks.
Should not be accessible from the Internet.

**Recommendation**

Keep enabled if Windows requires it.
Block external access using Windows Defender Firewall.
Restrict access to trusted internal networks only.
Ensure Windows is fully patched.
Port 445/TCP

**Detected Service**

Microsoft-DS (Server Message Block - SMB)

**Purpose**

File and printer sharing.
Windows networking.
Active Directory communications.

**Risk Level**

High

**Potential Threats**

One of the most attacked Windows services.
Used by malware such as:
WannaCry
NotPetya
EternalBlue exploits
Can allow unauthorized file access if misconfigured.

**Recommendation**

Disable File and Printer Sharing if not needed.
Block TCP 445 from public networks.
Use Windows Firewall.
Enable SMB signing where appropriate.
Disable SMBv1.
Keep Windows updated.
Port 1583/TCP

**Detected Service**

Pervasive SQL Server – Relational Engine (encrypted)
Nmap identified it as psql

**Purpose**

Database engine used by Pervasive/Actian Zen database applications.

**Risk Level**

Medium to High

**Potential Threats**

Database services should not be publicly accessible.
Could expose:
customer records
application data
authentication information
May be vulnerable if outdated.

**Recommendation**

Verify the application actually requires this database.
Restrict access to localhost or trusted hosts.
Update the database software.
Stop or uninstall the service if no longer used.
Port 3351/TCP

**Detected Service**

Pervasive SQL Btrieve Engine

**Purpose**

Legacy database engine used by older business software.

**Risk Level**

Medium

**Potential Threats**

Legacy database engines may contain unpatched vulnerabilities.
Unauthorized users could access database files if improperly configured.

**Recommendation**

Verify whether any installed application still depends on it.
Restrict network access.
Remove the service if it is no longer required.
Port 7070/TCP

**Detected Service**

SSL/RealServer (uncertain detection)

**Nmap reports:**

ssl/realserver?

This means the service could not be identified with certainty.

**Purpose**
Possible uses include:

Web application
Media server
Custom application
Development server
Third-party software

**Risk Level**

Unknown (requires investigation)

**Potential Threats**

Any unknown listening service should be investigated.
Could expose a web interface.
Could have authentication weaknesses.
Could belong to software no longer in use.

**Recommendation**
Identify the owning process:

netstat -ano | find ":7070"

**Then determine the process:**

tasklist /FI "PID eq <PID>"

Or in PowerShell:

Get-Process -Id <PID>

**If the application is unnecessary:**

Stop the service.
Disable it.
Uninstall the application.
Overall Risk Assessment
Port	Service	Risk	Recommendation
135	Microsoft RPC	Medium	Keep patched and restrict access
445	SMB	High	Disable if unused; block externally
1583	Pervasive SQL	Medium–High	Restrict or remove if unused
3351	Btrieve Engine	Medium	Restrict or remove if unused
7070	Unknown SSL service	Unknown	Investigate immediately
## Summary

| Scan Type | Command | Purpose |
|---|---|---|
| Basic Scan | `nmap 127.0.0.1` | Lists open/closed ports and services |
| Version Detection | `nmap -sV 127.0.0.1` | Identifies software/version running on each open port |
| OS Detection | `nmap -O 127.0.0.1` | Fingerprints and identifies the target's operating system |
