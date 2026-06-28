# 🐧 LINUX FOR SECURITY
> Linux is the tool and the battlefield. Master it and you master 80% of cybersecurity.
> Version 2.0 — BUFFED: privilege escalation, enumeration and more.

---

## Why is Linux the hackers' OS?
- **Transparent:** everything is a file, everything is inspectable and controllable.
- **It's the server OS:** most of what you'll attack/defend runs Linux.
- **The tools live here:** nmap, metasploit, all born on Linux.
- **The terminal is power:** what's 10 clicks elsewhere is one automatable command here.

---

## 1. The philosophy: "everything is a file"
**Absolutely everything** is represented as a file: documents, devices (`/dev/sda` is your disk), processes, configuration. You read, write and control it all with the same tools. The most powerful idea in the system.

---

## 2. The directory tree

```
/           → the root, the origin of everything
├── /bin    → essential commands
├── /boot   → boot and kernel
├── /dev    → devices
├── /etc    → ⭐ system CONFIGURATION
├── /home   → user folders
├── /opt    → optional software
├── /proc   → live process info (virtual)
├── /root   → the superuser's folder
├── /tmp    → temporary
├── /usr    → programs and libraries
└── /var    → ⭐ variable data: LOGS (/var/log)
```

**What an analyst watches:**
- `/etc/passwd` → users · `/etc/shadow` → password hashes (root only)
- `/var/log/` → traces of EVERYTHING · `/home/*/.ssh/` → SSH keys
- `/etc/crontab`, `/etc/cron.*` → scheduled tasks (malware persistence)

---

## 3. Survival commands

```bash
pwd; ls -la; cd /path; tree                  # move and view
find / -name "*.conf" 2>/dev/null            # find files
which nmap; locate file                       # locate
cat / less / head -n20 / tail -f file         # read (tail -f = live)
grep "error" log                              # search inside
cp / mv / rm / mkdir / touch / ln -s          # manipulate (rm = no trash, careful!)
```

---

## 4. Permissions — the heart of Linux security

`ls -l` shows: `-rwxr-xr--`
- 1st char: type (`-` file, `d` folder, `l` link)
- **rwx** owner · **r-x** group · **r--** others (r=read 4, w=write 2, x=execute 1)

```bash
chmod 755 file        # rwx / r-x / r-x
chmod 600 id_rsa      # owner only (private keys)
chmod +x script.sh    # make executable
chown user file       # change owner
```

### ⭐ SUID/SGID — where security breaks
A **SUID** binary runs with its **owner's** permissions, not the runner's. If it's root-owned → you run it but it acts as root. **The base of many privilege escalations.**
```bash
find / -perm -4000 -type f 2>/dev/null    # find SUID binaries (1st thing an attacker does)
```
If you find a weird SUID, check **GTFOBins** (gtfobins.github.io) → it tells you how to abuse it to become root.

---

## 5. Users and privileges

```bash
whoami; id                # who I am, my groups
sudo command; sudo -l     # run as root / see what I can run with sudo
su - user                 # switch user
cat /etc/passwd           # all users
```
**root (UID 0)** = system god. **Least privilege:** never work as root for everything. A bad `sudoers` config (`sudo -l`) is a direct escalation.

---

## 6. ⭐ Privilege escalation (privesc) — from user to root

What you do after entering a system with a limited user. **Enumerate first:**

```bash
# Quick manual enumeration
id; sudo -l                              # what can I run as root?
find / -perm -4000 -type f 2>/dev/null   # SUID binaries
cat /etc/crontab; ls -la /etc/cron.*     # scheduled tasks (editable?)
uname -a                                 # kernel version (known exploit?)
ps aux | grep root                       # root processes (exploitable?)
cat /etc/passwd                          # users
ls -la /home/*/.ssh/                     # SSH keys
env; cat ~/.bash_history                 # variables, history
```

**Common privesc paths:**
- **Misconfigured sudo:** `sudo -l` shows a binary you can run as root → GTFOBins tells you how to abuse it
- **Exploitable SUID:** a SUID binary in GTFOBins
- **Editable cron tasks:** a script root runs and you can modify
- **Old kernel:** a public exploit (DirtyCow, DirtyPipe, PwnKit...)
- **Credentials in files:** passwords in `.bash_history`, configs, `.env`

**Tools that automate enumeration:**
```bash
./linpeas.sh        # LinPEAS — scans EVERYTHING and highlights privesc paths ⭐
./linenum.sh        # LinEnum
```
> Practice privesc ONLY on CTF machines (HackTheBox, TryHackMe) or your lab.

---

## 7. Processes

```bash
ps aux                  # all processes
ps aux --sort=-%cpu     # by CPU usage
top / htop              # live monitor
kill -9 PID             # force kill
command &; nohup command &   # background
```
**Security:** on a compromise, `ps aux` looking for weird processes (strange names, high CPU, network). Malware runs as a process — that's where you catch it.

---

## 8. Network from the terminal

```bash
ip a; ip route          # IPs and gateway
ss -tunap               # connections + processes (C2?)
curl http://site; wget url
ssh user@ip; scp file user@ip:/path
nc -lvnp 4444           # listen (receive shells)
```

---

## 9. Logs — where the traces live

```bash
journalctl -xe                 # detailed events
journalctl -p err -S today     # today's errors
journalctl -u sshd -f          # a service, live
last; lastb                    # successful / failed logins
grep "Failed" /var/log/secure  # attack attempts (Fedora)
grep "Failed" /var/log/auth.log # (Debian/Ubuntu)
```
**An attacker wipes logs** (anti-forensics). That's why pros send logs to a central **SIEM** (Wazuh, Graylog) the attacker can't touch.

---

## 10. Bash scripting — automate your power

```bash
#!/bin/bash
TARGET=$1
[ -z "$TARGET" ] && { echo "Usage: $0 <ip>"; exit 1; }

echo "[*] Scanning $TARGET..."
nmap -sV "$TARGET" -oN "scan_$TARGET.txt"
if grep -q "80/tcp\|443/tcp" "scan_$TARGET.txt"; then
    echo "[+] Web found, launching nikto..."
    nikto -h "$TARGET"
fi
```
**Bash keys:** `$1` arguments · `$(cmd)` capture output · `|` pipe · `>` overwrite `>>` append · `&&` if success `||` if fail · `for x in list; do ... done`.

---

## 11. Hardening — locking down a Linux

1. **Update:** `sudo apt update && apt upgrade` (or `dnf update`)
2. **Firewall:** `ufw` / `firewalld` — close unnecessary ports
3. **fail2ban:** auto-blocks brute force
4. **Secure SSH:** no root login, keys only (no passwords), low `MaxAuthTries`
5. **Minimal software** = less attack surface
6. **Audit:** `sudo lynis audit system` (score + recommendations)
7. **Rootkits:** `sudo rkhunter --check`, `chkrootkit`
8. **No `chmod 777`**, least privilege always
9. **3-2-1 backups** = the only real defense against ransomware

```bash
sudo lynis audit system                       # security score
find / -perm -4000 -type f 2>/dev/null         # SUID
ss -tulpn                                      # what's listening
last -10                                       # recent logins
```

---

## 🎓 To practice
- **OverTheWire: Bandit** → the BEST wargame to master the terminal, free. Start TODAY.
- **GTFOBins** (gtfobins.github.io) → how normal binaries escalate privileges.
- **HackTheBox / TryHackMe** → practice real privesc on legal machines.
- **Linux Journey** (linuxjourney.com) → free interactive course.
- **Continue with:** `book-networking.md` and `book-web-hacking.md`.

> ⬛ *The terminal isn't hard, it's honest. It does exactly what you tell it. Learn to tell it well.*
