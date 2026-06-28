# 🔬 DIGITAL FORENSICS & INCIDENT RESPONSE BOOK
> When something bad happens, someone has to reconstruct the truth. That's you.
> Version 2.0 — BUFFED: more commands, artifacts and practical cases.

---

## What is digital forensics?
The science of **collecting, preserving and analyzing digital evidence** to reconstruct what happened in an incident: a hack, fraud, data theft. It answers: *how did they get in? what did they do? what did they take? are they still inside?* It's digital detective work — one of the most respected and best-paid in security.

---

## 1. The sacred principles (break them and the evidence is useless)

1. **Don't alter the original:** work on **copies**, never the original.
2. **Chain of custody:** document who touched the evidence, when and why. No gaps.
3. **Verifiable integrity:** compute the **hash** on collection. If it changes, it was altered.
4. **Document EVERYTHING:** every step, command and finding. If it's not documented, it didn't happen.
5. **Reproducibility:** another analyst must reach the same result repeating your steps.

---

## 2. The order of volatility — what to save first

From most volatile (lost fast) to most permanent:

1. **CPU registers/cache** (milliseconds) — almost impossible.
2. **RAM** ⭐ — lost on shutdown. **CAPTURE IT BEFORE POWERING OFF.** Active malware, passwords, connections, encryption keys live here.
3. **Network state** — active connections, ARP, sockets.
4. **Running processes.**
5. **Disk** — permanent but erasable.
6. **Remote logs / backups.**

**Rookie mistake #1:** powering off/rebooting the compromised machine → you lose the RAM (the best clues). **Isolate from the network, DON'T power off** until you capture memory.

---

## 3. Memory (RAM) forensics with Volatility3

RAM has what was happening **live**, even fileless malware (that doesn't touch disk).

### Capture the RAM
```bash
# Linux: AVML (Microsoft) or LiME
sudo ./avml memory.lime
# Windows: WinPmem, DumpIt, FTK Imager
```

### Analyze — essential plugins (Windows)
```bash
vol -f mem.dmp windows.info                    # dump info
vol -f mem.dmp windows.pslist                  # processes
vol -f mem.dmp windows.pstree                  # tree (parent/child)
vol -f mem.dmp windows.psscan                  # HIDDEN processes (hiding from pslist)
vol -f mem.dmp windows.netscan                 # network connections (C2?)
vol -f mem.dmp windows.cmdline                 # what each process ran
vol -f mem.dmp windows.malfind                 # ⭐ injected code
vol -f mem.dmp windows.dlllist --pid 1234      # a process's DLLs
vol -f mem.dmp windows.handles --pid 1234      # files/registry it opens
vol -f mem.dmp windows.filescan                # files in memory
vol -f mem.dmp windows.dumpfiles --pid 1234    # dump a process to analyze
vol -f mem.dmp windows.hashdump                # password hashes
vol -f mem.dmp windows.registry.printkey -K "..."   # registry keys
```

### Linux
```bash
vol -f mem.lime linux.pslist
vol -f mem.lime linux.bash          # bash history in memory
vol -f mem.lime linux.check_syscall # detect rootkit hooks
```

**What you look for:** processes with weird names, no legitimate parent, connections to strange IPs, injected code (`malfind`), obfuscated/base64 powershell in `cmdline`.

---

## 4. Disk forensics

### Create a forensic image (bit-by-bit copy)
```bash
sudo dd if=/dev/sdb of=image.dd bs=4M status=progress
sha256sum image.dd > image.hash        # prove integrity
# Better: dcfldd or ewfacquire (E01 format with metadata)
```
**Never** analyze the original disk — analyze the image (mount it read-only).

### Analyze with Sleuth Kit / Autopsy
```bash
mmls image.dd                    # partition table
fls -r -m / image.dd > body.txt  # list files (incl. deleted)
icat image.dd 12345 > file       # extract a file by inode
mactime -b body.txt -d > timeline.csv   # file timeline
```
**Autopsy** (GUI) recovers deleted files, browser history, builds timelines and keyword search.

### File carving (recover by signature, no file table)
```bash
foremost -i image.dd -o recovered/
scalpel image.dd -o output/
binwalk -e firmware.bin           # extract from firmware/binaries
```

### MAC times (when each thing happened)
- **M**odified (content), **A**ccessed (read), **C**hanged (metadata)
- Cross-referencing reconstructs what the attacker did and when. Note: advanced ones fake them (*timestomping* — compare with `$MFT` or logs).

---

## 5. Windows forensics (key artifacts)

Windows leaves MANY traces. The gems:
- **Registry:** `SAM`, `SYSTEM`, `SOFTWARE`, `NTUSER.DAT` → users, programs, persistence (`Run` keys)
- **Prefetch** (`C:\Windows\Prefetch`) → which programs ran and when
- **Event Logs** (`.evtx`) → logins (event 4624/4625), process creation (4688)
- **Amcache / ShimCache** → executable history
- **$MFT** → metadata of ALL NTFS files
- **Jump Lists / LNK** → recently opened files
- **Browser artifacts** → history, cookies, downloads
- Tools: **Eric Zimmerman's tools**, **RegRipper**, **Autopsy**

---

## 6. Network forensics

```bash
# Capture live traffic
sudo tcpdump -i eth0 -w capture.pcap

# Analyze (Wireshark GUI, or tshark CLI)
tshark -r capture.pcap -Y "http.request"           # HTTP requests
tshark -r capture.pcap -Y "dns" -T fields -e dns.qry.name   # DNS queries
tshark -r capture.pcap --export-objects http,output/        # extract transferred files
```
Look for: periodic traffic to a strange IP (C2 beaconing), DNS to suspicious domains, large transfers (exfiltration), weird protocols.

---

## 7. Log analysis — the attack timeline

```bash
# Logins and authentication (Linux)
grep "Accepted\|Failed" /var/log/secure
last        # successful logins
lastb       # failed attempts (brute force)

# Time range
journalctl -S "2026-06-26 00:00" -U "2026-06-26 23:59"
journalctl -u sshd

# IPs that attacked SSH the most
grep "Failed password" /var/log/secure | grep -oE "[0-9.]{7,15}" | sort | uniq -c | sort -rn
```

**Correlation = the art:** combine failed logins + successful login + launched processes + connections, ordered in time. The story builds itself: *"03:14 IP X tried 500 passwords, 03:16 got in, 03:18 downloaded something, 03:20 connected to a C2."*

---

## 8. Incident Response — the process (NIST/SANS, 6 phases)

1. **Preparation:** plan, tools, backups, active logs, IDS watching — BEFORE the incident.
2. **Identification:** is there a real incident? scope? Confirm with evidence, not panic.
3. **Containment:** isolate from the network (without powering off — saves RAM); then long-term containment.
4. **Eradication:** remove the malware, close the entry vector, reset credentials.
5. **Recovery:** restore from clean backups, watching it doesn't return.
6. **Lessons learned:** document, improve defenses. The phase people skip and get hacked again for.

---

## 9. IOCs and YARA

**IOCs (Indicators of Compromise):** malware hashes, C2 IPs/domains, weird file names, suspicious registry/cron keys, anomalous user-agents. Search them in **VirusTotal** and threat intel → maybe already linked to a known group. Use them to hunt across ALL your machines (threat hunting).

### YARA — hunt malware by patterns
```yara
rule Malware_Example {
    meta:
        description = "Detects family X"
    strings:
        $a = "malicious_command"
        $b = { 6A 40 68 00 30 00 00 }   // specific bytes
        $c = "evil-c2.com" nocase
    condition:
        $a or $b or $c
}
```
```bash
yara my_rule.yar suspicious_file
yara -r my_rule.yar /home/             # recursive
vol -f mem.dmp yarascan.YaraScan --yara-file rule.yar   # YARA over RAM
```

---

## 10. Your quick triage kit

```bash
# Suspicious system — live triage
ps aux                    # processes
ss -tunap                 # network connections + processes
last && lastb             # logins
sudo rkhunter --check     # rootkits
sudo chkrootkit
clamscan -r --infected /  # known malware
sudo lynis audit system   # overall state
journalctl -p err -S today
find / -mtime -1 -type f 2>/dev/null      # files modified in last 24h
crontab -l; ls /etc/cron*                 # persistence
```

**REMnux** (malware analysis distro) has 200+ forensic tools ready.

---

## 🎓 To practice
- **CyberDefenders, Blue Team Labs Online** → real IR scenarios
- **DFIR.training, Volatility Labs** → memory forensics challenges
- **The DFIR Report** (thedfirreport.com) → step-by-step reconstructions of real attacks
- **Capture your own RAM** and analyze it with volatility — risk-free practice
- **Continue with:** `book-malware.md` and `book-cybersecurity.md`

> ⬛ *Every attacker leaves traces. Forensics is the art of reading them and telling the truth.*
