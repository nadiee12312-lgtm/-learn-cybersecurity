# 🌐 NETWORKING BOOK
> The foundation of ALL cybersecurity. If you don't understand networks, tools are magic you can't control.
> Version 2.0 — BUFFED: nmap in depth, Wireshark and more commands.

---

## Why start with networking?
Hacking is, at its core, **making machines communicate in unintended ways**. Every attack travels over the network: a scan, an exploit, data theft, a C2. Without understanding how information flows, you just copy commands. This book gives you the foundations.

---

## 1. The layer model — how data travels

### TCP/IP (the real one, 4 layers)
| Layer | What it does | Examples |
|-------|--------------|----------|
| **Application** | What you use | HTTP, DNS, FTP, SSH, SMTP |
| **Transport** | How it's delivered | TCP, UDP |
| **Internet** | How it's routed | IP, ICMP |
| **Link** | Physical and local net | Ethernet, MAC, ARP |

### OSI (theoretical, 7 layers)
Physical → Link → Network → Transport → Session → Presentation → Application.
"Layer 7 attack" = application (HTTP); "layer 3/4" = IP/TCP. You'll hear it constantly.

**Encapsulation:** your data goes into a TCP segment → into an IP packet → into an Ethernet frame. On arrival, it's unpacked in reverse.

---

## 2. IP addresses

- **IPv4:** `192.168.1.50` (running out). **IPv6:** `2001:db8::7334` (the future).
- **Public:** unique on the internet (given by your ISP). **Private** (internal):
  - `10.0.0.0/8` · `172.16.0.0/12` · `192.168.0.0/16` (typical home)
- **NAT:** the router translates your private IPs to its single public IP.

### Subnetting / CIDR
`/24` = first 24 bits are network, 8 for hosts = **254 machines**. `/16` = 65,534. `/24` = mask `255.255.255.0`. That's why `nmap 192.168.1.0/24` scans the 254 machines on your network.

---

## 3. Ports (the doors of each machine, 0-65535)

| Port | Service | | Port | Service |
|------|---------|---|------|---------|
| 21 | FTP | | 443 | HTTPS |
| 22 | SSH | | 445 | SMB (Windows) |
| 23 | Telnet ❌ | | 1433 | MSSQL |
| 25 | SMTP | | 3306 | MySQL |
| 53 | DNS | | 3389 | RDP |
| 80 | HTTP | | 5432 | PostgreSQL |
| 110/143 | POP3/IMAP | | 6379 | Redis |
| 161 | SNMP | | 8080 | HTTP-alt |

**States (nmap):** Open (service listening), Closed (responds without service), Filtered (firewall blocks).

---

## 4. TCP vs UDP

**TCP** (reliable, ordered) — the **3-way handshake** (key in scans): SYN → SYN-ACK → ACK. Used by web, SSH, mail.
**UDP** (fire and forget, fast) — streaming, gaming, DNS, VoIP.

A SYN scan (`nmap -sS`) sends only the SYN without completing the connection → stealthier. UDP (`nmap -sU`) is slow because it doesn't confirm.

---

## 5. nmap in depth (your #1 network tool)

```bash
# Discovery
nmap -sn 192.168.1.0/24            # ping scan: which hosts are alive
nmap 192.168.1.0/24                # basic scan (top 1000 ports)

# Scan types
nmap -sS target                    # SYN (stealthy, needs sudo)
nmap -sT target                    # connect (no sudo)
nmap -sU target                    # UDP (slow)
nmap -p- target                    # ALL 65535 ports

# Detection
nmap -sV target                    # service versions
nmap -O target                     # operating system
nmap -A target                     # everything (version + OS + scripts + traceroute)

# Scripts (NSE) — this is a whole course
nmap --script vuln target          # finds known vulnerabilities
nmap --script smb-enum-shares target
nmap -sC target                    # default scripts

# Speed and stealth
nmap -T4 target                    # faster (-T0 slow/stealthy to -T5 aggressive)
nmap -Pn target                    # skip ping (host that doesn't reply to ping)
nmap -oN out.txt target            # save results
```

---

## 6. DNS — the internet's directory

Translates names to IPs. Query: your PC asks the resolver → root servers → `.com` → the domain's server.

### Records (with `dig`)
A (IPv4), AAAA (IPv6), MX (mail), NS (DNS servers), TXT (SPF/DKIM), CNAME (alias), PTR (reverse).
```bash
dig domain.com A
dig domain.com MX
dig -x 8.8.8.8            # reverse (whose IP is this?)
dig domain.com ANY
dig @8.8.8.8 domain.com   # ask a specific DNS
host domain.com
nslookup domain.com
```
**For security:** DNS reveals subdomains, infrastructure, mail. First stop of recon. **DNS spoofing** sends you to a fake IP; **DNS tunneling** hides stolen data inside queries.

---

## 7. The local network — ARP, MAC, DHCP

- **MAC:** unique physical identifier of each card (`a4:5e:60:c3:1f:9b`). Can be faked (*MAC spoofing*).
- **ARP:** translates local IP ↔ MAC. Verifies nothing → an attacker replies "I'm that IP!" and diverts your traffic = **ARP spoofing**, the base of **MITM** on the local network (bettercap, ettercap).
- **DHCP:** assigns you an IP automatically when you connect. The router is the home DHCP server.

```bash
ip neigh        # ARP table (which IPs↔MACs your machine knows)
arp -a
```

---

## 8. HTTP/HTTPS

**Request:** method (GET/POST/PUT/DELETE) + path + headers (User-Agent, Cookie, Host) + body.
**Codes:** 2xx success · 3xx redirect · 4xx client error (403/404) · 5xx server error (500).
**HTTPS** = HTTP inside TLS. An attacker on your WiFi reads HTTP but not HTTPS.
**Session cookies:** after login, the server gives you a token; stealing it = stealing the session (*session hijacking*).

---

## 9. Wireshark — see the real traffic

```bash
sudo tcpdump -i wlan0 -n              # raw capture in terminal
sudo tcpdump -i wlan0 -w capture.pcap # save to file
wireshark                             # visual analysis
```

### Wireshark filters (essential)
```
http                          → HTTP only
http.request.method == "POST" → POST only (passwords travel here in HTTP!)
ip.addr == 192.168.1.50       → traffic to/from an IP
tcp.port == 443               → a port
dns                           → DNS queries
tcp.flags.syn == 1            → SYN packets (scans)
frame contains "password"     → packets with that word
```
**Trick:** right-click a packet → "Follow → TCP Stream" → see the whole conversation.

---

## 10. Essential network commands

```bash
ip a                 # your IPs and interfaces
ip route             # your gateway
cat /etc/resolv.conf # your DNS
ss -tunap            # active connections + processes (anything weird?)
ping 8.8.8.8         # is there internet? (ICMP)
traceroute google.com # what hops your traffic takes
curl -sI http://site # HTTP headers
nc -lvnp 4444        # listen on a port (receive connections)
```

---

## 11. Network defense
- **Firewall:** filters traffic by port/IP (`firewalld`, `ufw`, `iptables`).
- **VLAN:** splits a physical network into logical ones (isolates devices).
- **DMZ:** zone for public servers, separated from the internal network.
- **IDS/IPS:** detects (Suricata, Snort) or blocks malicious traffic.
- **VPN:** encrypted tunnel between networks (WireGuard, Tailscale, OpenVPN).
- **Segmentation:** separate into zones → if one falls, not all fall.

---

## 🎓 To practice
- **Map your own network:** `nmap -sV 192.168.1.0/24` and understand each device.
- **Spy on your traffic:** Wireshark + browse an HTTP site (you see everything) and an HTTPS one (encrypted).
- **Subnetting:** practice on subnetting.org until `/24`, `/16` come naturally.
- **Continue with:** `book-linux.md` and `book-web-hacking.md`.

> ⬛ *Networks are the battlefield. Know it better than the enemy.*
