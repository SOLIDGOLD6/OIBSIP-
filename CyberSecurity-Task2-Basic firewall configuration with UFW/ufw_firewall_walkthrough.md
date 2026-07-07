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

## 2. Walkthrough of the Screenshots

### Image 1 — Installing WSL
```
Downloading: Windows Subsystem for Linux 2.7.10
```
This is the very first step: installing **WSL (Windows Subsystem for Linux)** itself via `wsl.exe` on Windows. WSL lets you run a real Ubuntu Linux environment directly inside Windows, which is why all the following commands are run from an Ubuntu shell (`/mnt/c/Users/rikho`) even though the machine is Windows.

### Image 2 — Installing UFW and setting the first rules
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
4. **`sudo ufw deny http`** / **`sudo ufw allow http`** / **`sudo ufw deny http`** — these three commands are the user experimenting with the HTTP (port 80) rule, flip-flopping between deny and allow while getting a feel for how UFW rules work. The final state after this sequence is **deny http**, but this gets revisited again in the next image.

### Image 3 — Refining the rules and checking status
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

### Image 4 — Turning the rules into a reusable script
```bash
nano ufw_configuration.sh
chmod +x ufw_configuration.sh
./ufw_configuration.sh
```
1. **`nano ufw_configuration.sh`** — opens the nano text editor to write a script that automates all the UFW commands used so far (so they don't have to be typed manually every time).
2. **`chmod +x ufw_configuration.sh`** — makes the script executable.
3. **`./ufw_configuration.sh`** — runs the script. The output shows UFW backing up the old rule files (`user.rules`, `before.rules`, etc.) before applying the new ones, then applying:
   - Default incoming policy → `deny`
   - Default outgoing policy → `allow`
   - The allow/deny rules from before
   
   However, one line fails:
   ```
   ./ufw_configuration.sh: line 13: duso: command not found
   ```
   This is a **typo bug**: line 13 of the script was meant to say `sudo ufw deny from 192.168.1.100`, but it was mistyped as `duso ufw deny from 192.168.1.100`. Since `duso` isn't a real command, that line silently failed. It didn't break the script — bash just reports the error and moves on to the next line.

### Image 5 — Final confirmation
```bash
sudo ufw status verbose
```
This shows the same final rule set as Image 3 (22 allow, 80 deny, 443 allow, plus IPv6 versions). The rule against `192.168.1.100` had **already been applied manually back in Image 3**, so even though the script's line 13 failed due to the typo, the firewall still shows the correct end state — it just wasn't re-applied by the script itself.

### Image 6 — The full script contents
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
duso ufw deny from 192.168.1.100      # <-- typo, should be "sudo"

sudo ufw --force enable

sudo ufw status verbose
```
This is the complete automation script. It:
1. Updates package lists and (re)installs UFW.
2. **Resets** UFW to a clean slate (`--force reset`) so old rules don't linger.
3. Sets the **default policies**.
4. Applies the SSH/HTTP/HTTPS/IP-block rules.
5. Enables UFW (`--force` skips the "are you sure" prompt, useful for scripting).
6. Prints the final status for verification.

**Fix needed:** change `duso ufw deny from 192.168.1.100` to `sudo ufw deny from 192.168.1.100` so the script is fully self-contained and doesn't rely on that rule already existing from a manual command.

---

## 3. What Each Rule Achieves — and Why

| Rule | What it does | Why this rule was chosen |
|---|---|---|
| **Default: deny incoming** | Blocks *any* inbound connection unless a rule explicitly allows it. | This is the "secure by default" foundation of the whole setup. Rather than trying to block every bad thing individually, you block everything and only open what's needed. It drastically shrinks the attack surface. |
| **Default: allow outgoing** | Lets the machine freely initiate connections out to the internet (e.g., downloading updates, browsing). | Outbound traffic is generally low-risk and needed for normal operation (`apt update`, web browsing, etc.), so it's left open rather than adding friction for legitimate use. |
| **`allow ssh` (port 22)** | Permits incoming SSH connections. | SSH is how you remotely administer the machine. Without this rule, enabling the firewall would lock out remote access — a very common and painful mistake. This rule is applied *before* the deny-by-default policy takes effect in practice, ensuring the admin doesn't get locked out. |
| **`deny http` (port 80)** | Blocks incoming unencrypted web traffic. | Port 80 is plain-text HTTP — traffic isn't encrypted, so credentials or data sent over it can be intercepted. Since HTTPS (443) is allowed instead, there's no need to expose the insecure port 80 service. |
| **`allow https` (port 443)** | Permits incoming encrypted web traffic. | If the machine is meant to serve a website, HTTPS is the secure, modern standard (TLS-encrypted). Allowing only 443 (and not 80) nudges/forces all web traffic to use encryption. |
| **`deny from 192.168.1.100`** | Blocks all traffic (any port) from that specific IP address. | This targets a known-bad or untrusted host on the local network — for example, a machine that's been flagged as compromised, running scans, or simply one the admin doesn't want to have any access at all, regardless of which port it tries. Blocking by IP rather than by port is used because the concern is about *who* is connecting, not *which service* they're hitting. |

---

## 4. Key Takeaways

- **Deny-by-default + explicit allow-listing** is the core firewall philosophy demonstrated here: block everything, then open only what's necessary (SSH for admin access, HTTPS for secure web traffic).
- **Order matters when testing manually** — the back-and-forth `deny http` / `allow http` / `deny http` in Image 2 shows the user testing behavior interactively before settling on a final rule.
- **Scripting firewall rules** (Image 6) makes the configuration repeatable and version-controllable, but a single typo (`duso` instead of `sudo`) can silently cause a rule to be skipped — always test scripts and check `ufw status verbose` after running them to confirm the actual state matches the intended one.
- **`ufw status verbose`** is the command to trust — it shows what rules are *actually* active, not just what you intended to set.
