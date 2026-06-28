# 🛡️ CYBERSECURITY BOOK
> Complete guide: the world's tools, how criminals attack, and how to defend and trace them back.
> Version 2.0 — From zero to professional. The academy's master book.

---

## ⚠️ BEFORE YOU START — Read this

This book teaches **offensive and defensive** cybersecurity with one purpose: that you learn how attacks work **so you can defend against them**. That's called "thinking like an attacker to defend like a professional".

**The golden rules, no exceptions:**

1. **You only attack what is yours or what you have written permission to attack.** Your own lab, vulnerable machines built for practice (DVWA, HackTheBox, TryHackMe, VulnHub), or a client who hired you and signed a paper.
2. **Attacking someone else's system without permission is a crime**, even if you were "just testing". In the US (CFAA) and most countries there's real jail time for this.
3. **"Hacking back" against whoever attacked you is also illegal** almost everywhere. In this book you'll learn to **trace and report**, not to counterattack. That's the difference between a professional and a criminal.
4. Knowledge isn't the crime. **Unauthorized use is.**

A security professional uses exactly the same tools as a criminal. The only thing that separates them is **permission** and **intent**. You're going to be one of the good ones.

---

## 📖 How to think about cybersecurity

Before the tools, the mindset. All of cybersecurity revolves around two teams:

- **🔴 Red Team (offensive):** simulates the attacker. Finds holes before a criminal does.
- **🔵 Blue Team (defensive):** detects, responds and cleans up. Watches, alerts and kicks out the intruder.
- **🟣 Purple Team:** the mix — attacks and defends at once to improve both sides.

### The phases of an attack — The "Cyber Kill Chain"

Every attack, from the simplest to the most sophisticated, follows roughly these steps (Lockheed Martin model):

1. **Reconnaissance** — the attacker researches the target (OSINT, port scanning).
2. **Weaponization** — prepares the weapon: an exploit, a document with malware.
3. **Delivery** — sends it: phishing email, infected USB, fake link.
4. **Exploitation** — the weapon fires: a vulnerability is leveraged.
5. **Installation** — the malware installs and stays.
6. **Command and Control (C2)** — the attacker controls the machine remotely.
7. **Actions on objectives** — steals data, encrypts files (ransomware), spies.

If you break **any link** in this chain, the attack fails. That's why defending is breaking links.

### MITRE ATT&CK — the enemy's dictionary

**MITRE ATT&CK** (attack.mitre.org) is a free encyclopedia cataloging **all the real techniques** attackers use, with codes like `T1566` (Phishing) or `T1059` (command-line execution). Professional defenders use it to know what to look for. Memorize it as a reference: when you see an attack, you can classify it with ATT&CK.

---

# 🗡️ PART 1 — THE ESSENTIAL ARSENAL

Here, one by one, are the core tools of the trade: what they are, what they're for, what they teach, and a usage example. Organized by the attack/defense phase where they're used.

---

## 🔍 1.1 — Reconnaissance and OSINT

Reconnaissance is the **first phase of everything**. Without information, there's no attack or defense. These tools gather public data about the target.

### `whois`
- **What it is:** queries the public database of domains and IP ranges.
- **For what:** know who registered a domain, when, what company, what DNS servers and sometimes contact email/phone.
- **Teaches:** the internet has a public "civil registry". Almost every domain leaves a trace of its owner.
- **Example:** `whois example.com`
- **Defense:** companies use "WHOIS privacy" to hide this. If yours is exposed, it's info an attacker uses.

### `dig`
- **What it is:** queries DNS servers (the internet's "phone book" translating names to IPs).
- **For what:** see A (IP), MX (mail), NS (name servers), TXT (SPF/DKIM), CNAME (alias) records.
- **Teaches:** how DNS works inside, the base of all internet communication.
- **Example:** `dig example.com ANY` · `dig MX example.com`
- **Offensive trick:** DNS records reveal subdomains, mail providers and sometimes misconfigured internal servers.

### `theHarvester`
- **What it is:** automatic OSINT collector (in Kali/Parrot).
- **For what:** gathers emails, subdomains, employee names and IPs of a target from search engines and public sources.
- **Teaches:** how much "surface" a company exposes without realizing.
- **Example:** `theHarvester -d example.com -b all`

### `sherlock` / `maigret`
- **What they are:** search a username across hundreds (sherlock) or 2500+ (maigret) social networks and platforms.
- **For what:** see where a username exists (digital identity tracking).
- **Teaches:** reusing the same nick everywhere makes you easy to track.
- **Example:** `sherlock username` · `maigret username`

### `exiftool`
- **What it is:** reads and edits file **metadata** (photos, PDFs, documents).
- **For what:** a photo can hide the **exact GPS location**, camera/phone model, date and time. A PDF can carry the author's real name and software used.
- **Teaches:** files "talk" more than you think. Pure OSINT.
- **Example:** `exiftool photo.jpg`
- **Defense:** strip metadata before publishing: `exiftool -all= photo.jpg`

### `nmap` — the king of network reconnaissance
- **What it is:** the most famous network/port scanner in the world. Your most important tool.
- **For what:** discover what machines are on a network, what ports are open, what services run (and versions), even the OS.
- **Teaches:** how an attacker "sees" your network. Every open port is a potential door.
- **Key examples:**
  - `nmap -sV --open 192.168.1.0/24` → scans your whole local network, shows open ports and versions.
  - `nmap -sC -sV -p- target` → full scan with scripts and all ports.
  - `nmap -O target` → tries to detect the OS.
  - `nmap --script vuln target` → looks for known vulnerabilities.
- **The NSE (Nmap Scripting Engine):** nmap ships with hundreds of scripts (`/usr/share/nmap/scripts/`) detecting everything from vulnerabilities to default credentials. This alone is a whole course.

---

## 🔬 1.2 — Web vulnerability analysis

### `nikto`
- **What it is:** web server scanner.
- **For what:** checks a website for dangerous files, insecure configs, old software and thousands of known issues.
- **Teaches:** a misconfigured web server screams its weaknesses.
- **Example:** `nikto -h http://target.com`
- **Note:** it's "noisy" — any decent defense detects it instantly. Good for learning, bad for stealth.

### `sqlmap`
- **What it is:** automates detection and exploitation of **SQL injection** (SQLi), one of the most dangerous web vulnerabilities.
- **For what:** if a web doesn't properly filter form input, sqlmap can read (or delete!) the whole database: users, passwords, everything.
- **Teaches:** why you must NEVER trust user input, and why "parameterized queries" exist.
- **Example:** `sqlmap -u "http://target.com/item?id=1" --dbs --batch`
- **Defense:** validate and parameterize all input, use ORMs, least privilege in the DB.

---

## 💥 1.3 — Exploitation

### `msfconsole` (Metasploit Framework)
- **What it is:** the most powerful exploitation framework. A Swiss army knife with thousands of exploits, payloads and modules.
- **For what:** launch exploits against known vulnerabilities, generate test malware (payloads), establish control sessions (Meterpreter), do post-exploitation.
- **Teaches:** the whole attack cycle, start to finish, in a controlled environment. THE tool to learn exploitation.
- **Concepts to master:**
  - **Exploit:** the code that leverages the vulnerability.
  - **Payload:** what runs after exploiting (e.g. a reverse shell).
  - **Meterpreter:** an advanced payload giving total control: upload/download files, keylogging, webcam, etc.
- **Basic flow:**
  ```
  msfconsole
  search type:exploit service_name
  use exploit/...
  set RHOSTS target
  set PAYLOAD ...
  exploit
  ```
- **Practice ONLY on:** Metasploitable, DVWA, or HackTheBox/TryHackMe machines.

### `searchsploit` + ExploitDB
- **What it is:** offline search of public exploits (from exploit-db.com).
- **For what:** given a software and version, find if a public exploit already exists.
- **Example:** `searchsploit apache 2.4` · `searchsploit -m 12345` (copies the exploit).
- **Teaches:** old, unpatched software is gold for an attacker. **Updating is defense.**

---

## 🔑 1.4 — Passwords and cracking

Understanding how passwords break is understanding why they must be long and unique.

### Key concept: hashes
Systems don't store your password in plaintext, they store a **hash** (an irreversible mathematical fingerprint). On login, they hash what you type and compare. "Cracking" is trying millions of passwords, hashing each, and seeing which matches.

### `john` (John the Ripper)
- **What it is:** classic, versatile password cracker.
- **For what:** break password hashes (Linux, ZIP, PDF, etc.) by dictionary or brute force.
- **Example:** `john --wordlist=/opt/wordlists/rockyou.txt hashes.txt`
- **Teaches:** a password like "Maria123" falls in seconds.

### `hashcat`
- **What it is:** the fastest cracker in the world (uses GPU, also works on CPU, slower).
- **For what:** break almost any hash type at brute speed.
- **Example:** `hashcat -m 0 -a 0 hashes.txt /opt/wordlists/rockyou.txt` (mode 0 = MD5).
- **The "-m" is the hash type:** 0=MD5, 100=SHA1, 1000=NTLM (Windows), 1800=sha512crypt (Linux), 22000=WPA2 (WiFi).
- **Teaches:** the real power of modern brute force, and why 8 characters are no longer enough.

### `hydra`
- **What it is:** **online** login cracker (against live services).
- **For what:** try users/passwords against SSH, FTP, RDP, web forms, etc.
- **Example:** `hydra -l admin -P /opt/wordlists/rockyou.txt ssh://192.168.1.50`
- **Teaches:** why lockout-after-X-attempts and two-factor authentication (2FA) exist.
- **Defense:** fail2ban, 2FA, rate limits. **Never use this against services that aren't yours.**

### `rockyou.txt`
- **What it is:** the most famous wordlist: 14 million real passwords leaked in 2009.
- **For what:** the base dictionary for all cracking. It's at `/opt/wordlists/rockyou.txt`.
- **Teaches:** if your password is in rockyou, it's no longer a password.

---

## 🧬 1.5 — Reverse engineering and binary analysis

### `radare2` (r2) and `Ghidra`
- **What they are:** reverse engineering frameworks. radare2 (terminal) and Ghidra (the NSA's, with GUI and a **decompiler** that turns assembly back into readable C-like code, released free).
- **For what:** open a compiled program (an `.exe`, a Linux binary, malware) and understand what it does inside, without source code.
- **Teaches:** how a computer thinks at low level (assembly), how to analyze malware, how to find vulnerabilities.
- **Example:** `r2 -A binary` then `afl` (list functions), `pdf @main` (view main).
- **Note:** steep learning curve, but among the best-paid skills in security.

### `binwalk`
- **What it is:** analyzes binary files and firmware looking for hidden files inside.
- **For what:** extract content from router/IoT firmware images, find files hidden inside others.
- **Example:** `binwalk -e firmware.bin`
- **Teaches:** IoT device analysis and steganography.

---

## 🛡️ 1.6 — Defense, auditing and antimalware (your Blue Team side)

These are **defensive** tools — used to protect and review YOUR system.

### `clamscan` (ClamAV)
- Open-source antivirus. Scans files/folders for known malware. `clamscan -r --infected /home`

### `rkhunter` (Rootkit Hunter)
- Detects **rootkits** (malware hiding deep in the system). `sudo rkhunter --check`. Teaches how attackers hide and how to hunt them.

### `lynis`
- Security auditor and "hardening" tool for Linux/Unix. Reviews EVERYTHING and gives a "hardening index" with concrete recommendations. `sudo lynis audit system`. The best for learning real hardening.

### `yara`
- The "language" for writing rules that identify malware by patterns. Analysts write YARA rules describing a malware family, then scan files or memory for matches. The industry standard. `yara my_rule.yar suspicious_file`

### `volatility3` (`vol` command)
- The most important **RAM memory forensics** framework. Analyze a memory dump to see what processes ran, what network connections existed, what malware was active — even things that leave no disk trace. `vol -f memory.dmp windows.pslist`

### `suricata`
- A high-performance **IDS/IPS** (Intrusion Detection/Prevention System). Watches network traffic in real time and **alerts (or blocks)** when it sees something malicious — a scan, a known exploit, traffic to a C2. `sudo suricata -i wlan0`. The heart of the Blue Team.

### `tcpdump`
- Command-line packet capturer. See, raw, all traffic in and out of your machine. `sudo tcpdump -i wlan0 -n port 80`. The base of traffic analysis.

---

## 🌐 1.7 — Anonymity and networks

### `tor` and `proxychains`
- **Tor:** the most famous anonymity network. Routes your traffic through 3 random encrypted servers, hiding your real IP. Access to the .onion network. Teaches how internet anonymity works (and its limits — Tor doesn't make you invincible).
- **proxychains:** forces any program through a proxy (or Tor). `proxychains nmap -sT target`.

### `aircrack-ng`
- The **WiFi** auditing suite. Capture WiFi traffic, capture a WPA2 "handshake" and crack its password. Teaches why WPA2 with a weak password is insecure, and why WPA3 exists. Needs a WiFi adapter with **monitor mode** (like the Alfa AWUS036ACS). Only on YOUR networks.

### `nc` (netcat) and `socat`
- The "Swiss army knife" of network connections. Create raw TCP/UDP connections: transfer files, create (test) shells, listen on ports, debug services. `nc -lvnp 4444` (listen on port 4444 — typical for receiving a test reverse shell). Absolute fundamentals of how machines communicate. Essential in CTFs.

---

# 🌍 PART 2 — TOOLS TO KNOW (build out your map)

These complete your mental map of the profession. Most live in security distros (Kali, Parrot, BlackArch) or you install them when needed.

## Recon and scanning
- **`masscan`** — ultra-fast port scanner: scans ALL the internet in minutes. Where nmap is a scalpel, masscan is a machine gun.
- **`gobuster` / `ffuf` / `feroxbuster`** — brute-force hidden directories and files on webs (`/admin`, `/backup`, etc.). Indispensable in web pentesting.
- **`nuclei`** — modern template-based vulnerability scanner. The current standard.
- **`amass` / `subfinder`** — mass subdomain enumeration.
- **`recon-ng`** — Metasploit-style OSINT framework.
- **`Shodan`** — search engine for internet-connected devices (cameras, servers, IoT). "The hackers' Google". shodan.io.
- **`Maltego`** — graphical tool to map relationships between people, domains, emails (with a visual connection graph).

## Traffic and network analysis
- **`Wireshark`** — the GUI packet analyzer. The world standard. It's tcpdump but visual and with superpowers. **Learn it no matter what.**
- **`bettercap`** — the Swiss army knife for **Man-in-the-Middle** (MITM) attacks on networks.
- **`Responder`** — captures credentials on Windows networks by poisoning protocols (LLMNR/NBT-NS). Key in internal pentesting.

## Web pentesting
- **`Burp Suite`** — THE web hacking tool. A proxy that intercepts and modifies all traffic between your browser and the web. Essential for professional web pentesting. Free Community version available.
- **`OWASP ZAP`** — the free, open-source alternative to Burp. Excellent to start.

## Active Directory / Windows (enterprise networks)
- **`BloodHound`** — graphically maps attack paths in a Windows/Active Directory domain. Reveals how to go from a normal user to admin. Revolutionized enterprise pentesting.
- **`CrackMapExec` / `nxc`** — Swiss army knife for attacking Windows networks (SMB, WinRM, LDAP).
- **`evil-winrm` / `impacket`** — Windows protocol shells and tools.
- **`mimikatz`** — extracts passwords and tickets from Windows memory. Legendary. (Run on the target Windows.)

## Forensics and response
- **`Autopsy`** — GUI for disk forensics. Recovers deleted files, analyzes timelines.
- **`Wazuh`** — open-source SIEM/XDR platform: gathers logs from many machines, detects threats and alerts.
- **`Velociraptor`** — endpoint threat hunting and incident response.

## Steganography and CTF
- **`steghide` / `stegseek`** — hide and extract data inside images/audio. Heavily used in CTFs.
- **`CyberChef`** — "the cyber Swiss army knife" on the web: decode, decrypt and transform data. A Google app. **Keep it in favorites.**

---

# 🦹 PART 3 — HOW UNETHICAL HACKERS ATTACK

> *To defend against something, you first have to understand how it works.* This part explains the **techniques and technologies** of criminals — not for you to use, but for you to **recognize and stop them**.

## 3.1 — Social engineering: hacking the human
The #1 technique in the real world. **The human is the weakest link**, always. No matter how good your technical security is if someone convinces you to give them your password.

- **Phishing:** fake emails/messages imitating a bank, company or friend to steal credentials or install malware. (MITRE T1566.)
- **Spear phishing:** targeted, personalized phishing to a specific person (using prior OSINT).
- **Vishing / Smishing:** the same via voice call or SMS.
- **Pretexting:** inventing a believable story ("I'm from tech support") to gain trust.
- **Baiting:** leaving an infected USB "forgotten" so someone plugs it in out of curiosity.

**How you recognize it:** artificial urgency ("your account will be blocked NOW!"), weird senders, links that don't match the text, spelling errors, requests for sensitive data. **When in doubt, don't click and verify through another channel.**

## 3.2 — Types of malware
- **Virus:** attaches to a file, replicates when run.
- **Worm:** spreads by itself over the network (e.g. WannaCry).
- **Trojan:** disguises as a legit program. You install it thinking it's something else.
- **RAT (Remote Access Trojan):** a trojan giving the attacker **total remote control**: files, webcam, mic, keyboard.
- **Keylogger:** records every key you press (including passwords).
- **Ransomware:** encrypts all your files and demands crypto ransom. The most expensive threat in the world today.
- **Rootkit:** hides in the system's depths so even the antivirus doesn't see it (that's why rkhunter exists).
- **Spyware / Stalkerware:** secretly spies on your activity.
- **Botnet:** a network of thousands of infected machines ("zombies") controlled at once for mass attacks.

## 3.3 — Command & Control (C2)
Once inside, malware needs to "call home" to receive orders. That channel is **C2 (Command and Control)**. Frameworks like **Cobalt Strike** (legit, but heavily used by criminals), **Sliver** or **Mythic** manage armies of infected machines.

**How C2 hides:**
- **Domain fronting / DNS tunneling:** hides control traffic inside normal-looking traffic (DNS queries, HTTPS requests to famous sites).
- **Beaconing:** the malware "calls home" every X time intermittently to avoid suspicion.

**How it's detected:** periodic, regular traffic to a strange IP/domain. Here **Suricata** (an IDS) and log analysis shine.

## 3.4 — What the attacker does once inside
1. **Privilege escalation:** from normal user to admin/root (leveraging bugs or misconfigs).
2. **Persistence:** ensures they stay even if you reboot (scheduled tasks, services, registry keys). (MITRE T1547.)
3. **Lateral movement:** jumps from one machine to another in the network, seeking the valuable target. (Here they use BloodHound, CrackMapExec.)
4. **Defense evasion:** disables antivirus, deletes logs, obfuscates malware to avoid detection. (MITRE T1562.)
5. **Exfiltration:** steals the data, usually encrypted and bit by bit to avoid notice.
6. **Anti-forensics:** wipes their tracks — cleans logs, changes timestamps, uses encryption.

## 3.5 — Common network attacks
- **MITM (Man-in-the-Middle):** the attacker gets between your communication and reads/modifies it (with bettercap, for example). That's why HTTPS matters.
- **ARP Spoofing:** tricks your local network so traffic passes through the attacker's machine.
- **DNS Spoofing:** sends you to a fake site even if you type the correct address.
- **DoS / DDoS:** saturate a service with traffic until it falls (one or thousands of sources).
- **Evil Twin:** a fake WiFi point imitating the real one so you connect and get robbed.

## 3.6 — The attacker's mind: how they think and where to study them (legally)

### How an attacker really THINKS (the mindset)
An attacker doesn't think in "tools", they think in **questions**. This is their mental logic, step by step:

1. *"What's here?"* — Maps everything. Never assumes. Lists every port, service, subdomain, employee. (Obsessive recon.)
2. *"What's misconfigured or outdated?"* — Looks for the weak link, not the strong one. Doesn't attack the armored door; looks for the open window.
3. *"What's the path of least resistance?"* — Prefers tricking a human (phishing) over breaking encryption. Laziness is efficiency for them.
4. *"If I were the owner, what mistake would I have made?"* — Thinks like the defender to find their slips. Reused passwords, forgotten admin panels, public backups.
5. *"How do I stay without being seen?"* — Persistence and stealth. Getting in is easy; staying undetected is the art.
6. *"How do I erase my tracks?"* — Always thinks of the exit and leaving no trace.

**The key to the mindset:** an attacker sees systems as **puzzles with rules that can be bent**, not as things that "must" be used a certain way. A login form isn't for logging in: it's a data entry that maybe doesn't validate well. That way of looking at the world — "what if I use it backwards from how it's supposed to?" — is the heart of hacking.

### Where to study this LEGALLY (ethical hacker communities)
Here hackers share how they think, openly and with nothing illegal:

- **CTF and HackTheBox write-ups** — pure gold. A hacker documents step by step HOW they solved a machine and, more importantly, **WHY they thought each thing**. You read their reasoning live. Search "HackTheBox writeup" or **ctftime.org**.
- **0x00sec.org** — technical, ethical hacking forum; a serious community discussing malware, exploits and techniques with a learning focus.
- **Reddit:** r/netsec, r/hacking, r/AskNetsec, r/blueteamsec — daily discussions by professionals.
- **YouTube (watch a hacker think live):**
  - **LiveOverflow** — explains the mindset and the "why" behind exploits.
  - **IppSec** — solves HackTheBox machines narrating every thought.
  - **John Hammond** — malware analysis and CTFs.
  - **NahamSec** — bug bounty and real web hacking.
- **DEF CON and Black Hat conferences** — thousands of free talks on YouTube where the world's best explain their techniques.
- **Discords:** TryHackMe, HackTheBox and bug bounty ones — people learning and sharing all day.

### How real CRIMINALS think (without going where you shouldn't) — Threat Intelligence
To see the mind of **real** attackers (ransomware groups, APTs, scammers), you don't enter their forums: **you read the reports of those who hunt them.** Security companies publish detailed analyses of real criminal operations:

- **The DFIR Report** (thedfirreport.com) — reconstruct real attacks hour by hour. Like reading the criminal's mind with full context.
- **Mandiant, CrowdStrike, Cisco Talos, Unit 42 (Palo Alto)** — their blogs analyze real criminal groups.
- **Krebs on Security** (krebsonsecurity.com) — a journalist who investigates cybercrime from the inside.
- **MITRE ATT&CK** — catalogs the exact techniques of each known criminal group.
- **Malware Bazaar / abuse.ch** — real malware samples to study (with GREAT care, on an isolated machine like REMnux).

### ⚠️ Real criminal forums — why you should NOT enter
There are forums and markets (on the normal web and the dark web) selling malware, stolen data, hacked accesses and exploits. **I know it's tempting. Don't enter. This is serious:**

1. **It's illegal to browse and participate.** Just registering or buying something makes you an accomplice to a crime. Minors have ended up with charges for this.
2. **They're watched by police.** The FBI, Europol and national police constantly infiltrate and take down these forums (RaidForums, BreachForums, Genesis Market — all fell, and their users were arrested, including teenagers).
3. **You'll get scammed or infected.** That world is based on deceiving each other. 90% of what's sold there is a scam or malware designed to hack YOU.
4. **It closes your future.** Security companies (where you'll want to work) check your digital trail. A history on criminal forums closes the career you're building. Forever.

**The important truth:** you don't need to enter a criminal forum to learn to defend against criminals — just as a doctor doesn't need to get infected to cure. All the real, deep, useful knowledge is in the **legal** resources above. Criminals aren't smarter for being on those forums; they're **dumber for risking their freedom.** Play the long game.

---

# 🚨 PART 4 — HOW TO RESPOND TO AN ATTACK (Blue Team)

> You're being attacked, or think you are. What do you do? This is **Incident Response**. There's a professional method, not improvising.

## 4.1 — The incident response process (NIST model)
Professionals follow **6 phases**. Memorize them:

1. **Preparation:** BEFORE the attack. Have backups, active logs, a plan, ready tools. (This is where all your hardening with `lynis` comes in.)
2. **Identification / Detection:** is there really an incident? How serious? Confirm with evidence.
3. **Containment:** stop the bleeding. Isolate the affected machine (disconnect from the network), without powering it off yet (you'd lose the RAM, which is evidence).
4. **Eradication:** remove the cause — kill the malware, close the hole it entered through.
5. **Recovery:** restore from clean backups, return systems to production, watching it doesn't return.
6. **Lessons learned:** how did it get in? how to prevent it? Document everything. The phase people skip and get hacked again for.

## 4.2 — Your response kit with the tools above
If you suspect your machine was compromised, this is your flow:

```bash
# 1. Suspicious active network connections? (C2?)
ss -tunap                 # current connections with their process
sudo tcpdump -i wlan0 -n  # see raw traffic live

# 2. Weird processes running?
ps aux --sort=-%cpu | head
top

# 3. Hidden rootkits or backdoors?
sudo rkhunter --check
sudo chkrootkit

# 4. Malware in files?
clamscan -r --infected /home

# 5. Audit the system's security state
sudo lynis audit system

# 6. Who logged in and when?
last            # login history
lastb           # failed login attempts
who             # active sessions now

# 7. Check system logs for traces
sudo journalctl -p err -S today          # today's errors
sudo grep -i "failed\|invalid" /var/log/auth.log   # failed logins
```

## 4.3 — Forensic analysis (if you want to understand WHAT happened)
- **RAM:** capture memory and analyze it with **`volatility3`** (`vol`) — you'll see hidden processes and connections no longer on disk.
- **Pattern detection:** write or download **`yara`** rules to identify the malware family.
- **Network:** set up **`suricata`** as an IDS to watch in real time and alert on future attacks.

## 4.4 — Defenses you should set up
- **`fail2ban`** — auto-blocks IPs that fail many logins (anti-brute force). **Install it now**, it's super useful.
- **Firewall** — `firewalld` or `ufw`. Close everything you don't need.
- **`suricata`** — your network IDS.
- **`Wazuh`** — a central SIEM that gathers and correlates everything.
- **2FA everywhere** — the most cost-effective defense against credential theft.
- **3-2-1 backups** — 3 copies, 2 different media, 1 offsite. The ONLY real defense against ransomware.

---

# 🕵️ PART 5 — TRACE THE ATTACKER BACK (Attribution)

> You want to know **who** attacked you and from where. This is called **attribution** and **threat intelligence**.

## ⚠️ The legal rule first
**You CANNOT "hack back".** Accessing the attacker's machine, even if they attacked you first, is a crime in most of the world. What you CAN and should do:
- **Trace with public information** (OSINT, logs, threat intel).
- **Document** everything with evidence.
- **Report** to the authorities and the attacker's provider.

Attribution is **hard** even for pros: serious attackers use Tor, VPNs, hacked intermediate machines and servers in other countries. But clumsy attackers (most of them) leave traces.

## 5.1 — Step 1: Gather the attack evidence
Before tracing, you need attacker data. From your logs you extract:
- **The source IP(s)** of the attack (from `auth.log`, Suricata, tcpdump).
- **The exact timestamps.**
- **The method** used (which port, which exploit, which user-agent).
- **Any file** they left (malware, scripts).

```bash
# IPs that tried to enter via SSH the most
sudo grep "Failed password" /var/log/auth.log | grep -oE "[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+" | sort | uniq -c | sort -rn
```

## 5.2 — Step 2: Investigate the attacker's IP (OSINT — legal)
With the IP, use your tools and public services:

```bash
whois 1.2.3.4                    # range owner, ISP, country
dig -x 1.2.3.4                   # reverse DNS name (PTR)
nmap -sV 1.2.3.4                 # what runs on that IP? (carefully — info only)
```

Public threat intelligence services:
- **AbuseIPDB** (abuseipdb.com) — is that IP already reported for attacking others? Almost always yes.
- **VirusTotal** — reputation of IPs, domains and files.
- **Shodan** (shodan.io) — what services does that IP expose? Is it a hacked intermediate server?
- **ip-api.com / ipinfo.io** — approximate geolocation and ISP.
- **GreyNoise** — is it a mass internet scanner or a targeted attack on you?

## 5.3 — Step 3: Analyze what they left
If they left malware or scripts, there are golden clues:
- **`exiftool`** on any file → metadata, language, software used.
- **`Ghidra` / `radare2`** on the malware → strings, embedded C2 domains, code language, developer paths (sometimes they leave `C:\Users\TheirName\...`).
- **`yara`** → identifies which family/group the malware is (that's partial attribution already).
- Look for the **IOCs (Indicators of Compromise)**: IPs, domains, file hashes. Search them in VirusTotal and threat intel — maybe already linked to a known group.

## 5.4 — Step 4: Tell the story (correlation)
Put it all together: IP + geolocation + attack time + malware type + language + targets. With that you build an **attacker profile**. This is exactly what an OSINT framework with a relationship graph, timeline and correlation is built for.

## 5.5 — Honeypots: let the attacker come to you
The most elegant attribution technique: setting a **trap**.
- **What a honeypot is:** a deliberately fake, vulnerable system, designed for attackers to attack. It has nothing of value — its only job is to **log EVERYTHING** the attacker does.
- **For what:** you study the attacker's techniques live, capture their malware and IPs, and learn a lot, with no real risk.
- **Tools:** `Cowrie` (SSH/Telnet honeypot), `T-Pot` (full honeypot suite with dashboards), `Canarytokens` (trap files that alert you if someone opens them).
- **Cheap project:** a Raspberry Pi running Cowrie is a perfect honeypot to learn, for very little money.

## 5.6 — Step 5: Report (the right and legal way)
- **To the attacker's ISP:** `whois` gives you the abuse contact (`abuse@provider.com`). Report it with your logs.
- **To AbuseIPDB:** report the IP to protect others.
- **To the authorities:** if the attack was serious, report it to your country's cybercrime police.
- **Keep all evidence** intact (logs, files, captures) in case of legal proceedings.

---

# 🎓 CLOSING — Your learning path

To get the most out of this, this order:

1. **Master the fundamentals:** networking (TCP/IP, DNS, HTTP), Linux and a language (Python is ideal). Without this, tools are magic you don't understand.
2. **Practice in legal and free environments:**
   - **TryHackMe** — the best to start, guided step by step.
   - **HackTheBox** — harder, real machines to hack (legally).
   - **OverTheWire** — terminal wargames, great for Linux.
   - **PortSwigger Web Security Academy** — free and the best for web hacking.
   - **DVWA / Metasploitable / VulnHub** — vulnerable machines for your own lab.
3. **Learn both sides:** don't be only Red Team. The best understand attack AND defense.
4. **Certifications (for the future):** CompTIA Security+ (base), then eJPT, PNPT or OSCP (the ones worth gold in pentesting).
5. **Read and stay current:** MITRE ATT&CK, threat intel reports, and r/netsec.

## The phrase that sums it all up
> "With great power comes great responsibility."

If you're learning this young, that's real power. Use it to **build and protect**, not to destroy. The best hackers in the world — the ones who earn most, the respected ones — are those who chose the right side. Be one of them.

---

## 📚 Other books in the academy
```
book-first-day.md            — start here if you're at zero
book-linux-from-scratch.md   — Linux for those who never touched a terminal
book-programmer-to-hacker.md — if you already code, your shortcut
book-networking.md           — networking fundamentals
book-linux.md                — Linux for security
book-python-hacking.md       — build your own tools
book-web-hacking.md          — web hacking with real payloads
book-osint.md                — open-source investigation
book-cryptography.md         — encryption and how it breaks
book-forensics.md            — post-attack investigation
book-malware.md              — malicious code analysis
```

> ⬛ *Learn, protect, and stay on the right side.*
