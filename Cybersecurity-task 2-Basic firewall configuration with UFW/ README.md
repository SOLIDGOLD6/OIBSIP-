# Setting Up a Firewall with UFW on WSL (Ubuntu)

## 1. What Is a Firewall?

A **firewall** is a security system that monitors and controls incoming and outgoing network traffic based on a set of rules. Think of it as a checkpoint sitting between your computer and the rest of the network (or the internet): every packet of data trying to get in or out has to pass inspection first.

A firewall decides what to do with traffic based on things like:
- **Source/destination IP address** (where the traffic is coming from or going to)
- **Port number** (which "door" on the machine the traffic is using — e.g., port 22 for SSH, port 80 for HTTP, port 443 for HTTPS)
- **Protocol** (TCP, UDP, etc.)
- **Direction** (incoming vs. outgoing)

Its main purpose is to **reduce your attack surface**: by default, a good firewall blocks everything, and you explicitly open only the specific services you actually need. This way, random scanners and attackers on the internet can't reach services running on your machine that you never intended to expose.

**UFW** ("Uncomplicated Firewall") is a user-friendly front-end for Linux's `iptables`/`nftables` firewall system. It lets you manage firewall rules with simple commands like `ufw allow` and `ufw deny` instead of writing raw `iptables` syntax.

---

## 2. Installing WSL


<img width="678" height="253" alt="1" src="https://github.com/user-attachments/assets/4fd47ca4-6ea3-4b26-a0db-a222368d0c93" />

```
Downloading: Windows Subsystem for Linux 2.7.10
```
This is the very first step: installing **WSL (Windows Subsystem for Linux)** itself via `wsl.exe` on Windows. WSL lets you run a real Ubuntu Linux environment directly inside Windows, which is why all the following commands are run from an Ubuntu shell (`/mnt/c/Users/rikho`) even though the machine is Windows.

Installing UFW and setting the first rules
<img width="852" height="636" alt="2" src="https://github.com/user-attachments/assets/2e8c2e98-faea-4f7f-82af-cf0b5947fb64" />

```bash
sudo apt install ufw
sudo ufw enable
sudo ufw allow ssh
sudo ufw deny http
sudo ufw allow http
sudo ufw deny http
```
Step by step:
1. **`sudo apt install ufw`** — installs the UFW package from Ubuntu's repositories.
2. **`sudo ufw enable`** — turns the firewall on and enables it to start automatically on boot.
3. **`sudo ufw allow ssh`** — allows incoming traffic on port 22 (SSH), so you can still remotely log into the machine.
4. **`sudo ufw deny http`** / **`sudo ufw allow http`** / **`sudo ufw deny http`** — these three commands are an experimentation with the HTTP (port 80) rule, flip-flopping between deny and allow while getting a feel for how UFW rules work. The final state after this sequence is **deny http**, but this gets revisited again in the next image.

Refining the rules and checking status
<img width="888" height="447" alt="3" src="https://github.com/user-attachments/assets/36dde52a-85c0-4280-82e1-76d91cdfb7d9" />

```bash
sudo ufw deny http
sudo ufw allow https
sudo ufw deny from 192.168.1.100
sudo ufw status verbose
```
1. **`sudo ufw deny http`** — reconfirms port 80 (HTTP) traffic is blocked.
2. **`sudo ufw allow https`** — allows port 443 (HTTPS) traffic.
3. **`sudo ufw deny from 192.168.1.100`** — blocks **all** traffic from the specific IP address `192.168.1.100`, regardless of port.
4. **`sudo ufw status verbose`** — displays the active rule set in detail. The output confirms:

| To | Action | From |
|---|---|---|
| 22/tcp | ALLOW IN | Anywhere |
| 80/tcp | DENY IN | Anywhere |
| 443 | ALLOW IN | Anywhere |
| Anywhere | DENY IN | 192.168.1.100 |
| *(IPv6 equivalents)* | | |

It also shows the **defaults**: `deny (incoming)`, `allow (outgoing)` — meaning any port not explicitly listed is blocked for incoming connections, while the machine itself is still free to reach out to the internet.

Turning the rules into a reusable script
<img width="791" height="680" alt="4" src="https://github.com/user-attachments/assets/dd50eb7c-c9e0-4e7d-bbdf-b7d1fe4b03dd" />

```bash
nano ufw_configuration.sh
chmod +x ufw_configuration.sh
./ufw_configuration.sh
```
1. **`nano ufw_configuration.sh`** — opens the nano text editor to write a script that automates all the UFW commands used so far, so they don't have to be typed manually every time.

 The full, corrected script contents
 <img width="495" height="441" alt="6" src="https://github.com/user-attachments/assets/1b93bfb4-3e00-4764-91fa-b9e97284b905" />

```bash
#!/bin/bash
sudo apt update
sudo apt install -y ufw

sudo ufw --force reset

sudo ufw default deny incoming
sudo ufw default allow outgoing

sudo ufw allow ssh
sudo ufw deny http
sudo ufw allow https
sudo ufw deny from 192.168.1.100

sudo ufw --force enable

sudo ufw status verbose
```
This is the complete automation script, now free of the earlier typo (the line correctly reads `sudo ufw deny from 192.168.1.100`). It:
- Updates package lists and (re)installs UFW.
- **Resets** UFW to a clean slate (`--force reset`) so old rules don't linger or conflict with a fresh run.
- Sets the **default policies** (deny incoming, allow outgoing).
- Applies the SSH/HTTP/HTTPS/IP-block rules.
- Enables UFW (`--force` skips the "are you sure you want to proceed with operation" prompt, which is necessary for unattended/scripted execution).
- Prints the final status for verification.

Because every line now uses valid `sudo` syntax, the script is fully self-contained — it doesn't depend on any rule having been set manually beforehand, and running it on a brand-new machine would produce the exact same end state shown in Images 4 and 5.

2. **`chmod +x ufw_configuration.sh`** — makes the script executable.
3. **`./ufw_configuration.sh`** — runs the script. Because the script itself contains `sudo` commands, Linux prompts for authentication mid-run:
   ```
   [sudo] authenticate] Password:
   ```
   Once the password is entered, the script proceeds cleanly:
   - It runs `apt update`, which checks the four configured repositories (`security`, `main`, `updates`, `backports`) and reports **59 packages can be upgraded** (informational only — it doesn't upgrade them, since the script only asks for `ufw` specifically).
   - It confirms **`ufw is already the newest version`**, so nothing new needs to be installed.
   - UFW then **backs up the existing rule files** (`user.rules`, `before.rules`, `after.rules`, and their IPv6 equivalents) before applying changes — this is a safety measure so the previous configuration can be restored if needed.
   - **Default incoming policy changed to 'deny'** and **default outgoing policy changed to 'allow'** — these are applied fresh by the script.
   - All the individual allow/deny rules are applied (**"Rules updated"** appears for each one, plus its IPv6 counterpart).
   - Finally, **"Firewall is active and enabled on system startup"** confirms UFW is running and will persist across reboots.

   This run is clean — every line of the script executed successfully with no errors.

 Final confirmation
 <img width="668" height="556" alt="5" src="https://github.com/user-attachments/assets/f9a3aa5f-27ea-4c0b-ae12-b6b4d7a689c3" />

```bash
sudo ufw status verbose
```
This is the continuation of the script's output, ending with the final rule table — matching what was configured manually in Image 3:

| To | Action | From |
|---|---|---|
| 22/tcp | ALLOW IN | Anywhere |
| 80/tcp | DENY IN | Anywhere |
| 443 | ALLOW IN | Anywhere |
| Anywhere | DENY IN | 192.168.1.100 |
| *(IPv6 equivalents)* | | |

This confirms the script **fully and correctly reproduced** the intended firewall configuration end-to-end, including the IP-specific block — this time applied *by the script itself*, not left over from an earlier manual command.


---

## 3. What Each Rule Achieves — and Why

| Rule | What it does | Why this rule was chosen |
|---|---|---|
| **Default: deny incoming** | Blocks *any* inbound connection unless a rule explicitly allows it. | This is the "secure by default" foundation of the whole setup. Rather than trying to block every bad thing individually, you block everything and only open what's needed. It drastically shrinks the attack surface. |
| **Default: allow outgoing** | Lets the machine freely initiate connections out to the internet (e.g., downloading updates, browsing). | Outbound traffic is generally low-risk and needed for normal operation (`apt update`, web browsing, etc.), so it's left open rather than adding friction for legitimate use. |
| **`allow ssh` (port 22)** | Permits incoming SSH connections. | SSH is how you remotely administer the machine. Without this rule, enabling the firewall would lock out remote access — a very common and painful mistake. Placing this rule before enabling the firewall ensures the admin doesn't get locked out. |
| **`deny http` (port 80)** | Blocks incoming unencrypted web traffic. | Port 80 is plain-text HTTP — traffic isn't encrypted, so credentials or data sent over it can be intercepted. Since HTTPS (443) is allowed instead, there's no need to expose the insecure port 80 service. |
| **`allow https` (port 443)** | Permits incoming encrypted web traffic. | If the machine is meant to serve a website, HTTPS is the secure, modern standard (TLS-encrypted). Allowing only 443 (and not 80) nudges/forces all web traffic to use encryption. |
| **`deny from 192.168.1.100`** | Blocks all traffic (any port) from that specific IP address. | This targets a known-bad or untrusted host on the local network — for example, a machine that's been flagged as compromised, running scans, or simply one the admin doesn't want to have any access at all, regardless of which port it tries. Blocking by IP rather than by port is used because the concern is about *who* is connecting, not *which service* they're hitting. |

---

## 4. Key Takeaways

- **Deny-by-default + explicit allow-listing** is the core firewall philosophy demonstrated here: block everything, then open only what's necessary (SSH for admin access, HTTPS for secure web traffic).
- **Order matters when testing manually** — the back-and-forth `deny http` / `allow http` / `deny http` in Image 2 shows the user testing behavior interactively before settling on a final rule.
- **Scripting firewall rules** (Image 6) makes the configuration repeatable and version-controllable. The script includes a `--force reset` step so it can be run safely from a clean state, and it prompts for `sudo` authentication once at the start of execution.
- **`ufw status verbose`** is the command to trust — it shows what rules are *actually* active, not just what you intended to set. In the final run, the script's output and the status table match exactly, confirming the automation works correctly end-to-end.
