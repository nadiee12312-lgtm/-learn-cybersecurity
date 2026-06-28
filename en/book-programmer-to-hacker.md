# 💻➡️🦹 FROM PROGRAMMER TO HACKER
> You already know how to code. That's a HUGE advantage. Here you turn it into a security superpower.
> For the one coming from code. Version 2.0

---

## 🎯 Your secret advantage

If you already program, you have a brutal advantage over 90% of people starting in security. Why?

- **You understand how software is built** → so you understand how it breaks.
- **You're not afraid of code** → you can read exploits, write your own tools, automate everything.
- **You think in logic** → security is pure logic applied in reverse.

Most hackers had to learn to program AFTER. You already have it. You just need to **flip your mind**: from "building things that work" to "finding how things fail".

---

## 🔄 The mental shift: from builder to breaker

As a programmer, you think about the **happy path**: "the user types their name, I greet them".

A hacker thinks about the **broken path**: "what if instead of my name I type code? What if I send 10,000 characters? What if I leave the field empty? What if I put quotes?"

> **The golden rule of hacking, in programmer terms:** *all user input is dangerous until proven otherwise.* You, as a dev, have trusted input. As a hacker, you NEVER trust it.

Every `input`, every parameter, every form field, every HTTP header — they used to be "data" to you. Now they're **doors that someone didn't validate properly.**

---

## 🕳️ Vulnerabilities, explained for devs

These are the classic flaws. As a programmer, you'll understand them instantly:

### SQL Injection
You wrote code like this, right?
```python
query = f"SELECT * FROM users WHERE name = '{user_input}'"
```
**That code is vulnerable.** If `user_input` is `' OR '1'='1`, the query becomes `... WHERE name = '' OR '1'='1'` → always true → the attacker gets in. You created the bug by trusting input. A hacker exploits it.
**The lesson you already know (or will learn):** use parameterized queries. ALWAYS.

### XSS (Cross-Site Scripting)
You showed what the user typed without cleaning it:
```javascript
element.innerHTML = userComment;
```
If the comment is `<script>stealCookies()</script>`, it runs in other people's browsers. You injected foreign code without meaning to.
**The lesson:** escape/sanitize all output.

### Others you'll grasp quickly
- **IDOR:** your API uses `/user/123` and doesn't verify that 123 is YOURS. You change to `/user/124` and see other people's data. An authorization flaw you commit as a dev without realizing.
- **Buffer overflow:** sending more data than a variable holds, overwriting memory. Your C/C++ background gives you an edge here.
- **Race conditions:** two operations that collide in time. If you've done concurrency, you already know them.

**Your superpower:** while others memorize these flaws, you **understand** them because you've written the code that causes them. That makes you learn 10x faster.

---

## 🐍 Write your own tools

This is where you really shine. While others only USE tools, you **create** them. Python is ideal (but any language works). Examples you can write TODAY:

### A port scanner (your own mini-nmap)
```python
import socket
from concurrent.futures import ThreadPoolExecutor

def scan(host, port):
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.settimeout(1)
    if s.connect_ex((host, port)) == 0:
        print(f"[+] Port {port} OPEN")
    s.close()

def scan_host(host):
    with ThreadPoolExecutor(max_workers=100) as ex:
        for port in range(1, 1025):
            ex.submit(scan, host, port)

# scan_host("127.0.0.1")   # ONLY your own machine
```
In 15 lines you made a multithreaded scanner. That's what a non-programmer can't do.

### A subdomain finder, a fuzzer, a hash cracker...
Everything the famous tools do, you can recreate to **understand it from the inside.** And understanding from the inside = mastering.

---

## 🌐 Web hacking for programmers

If you've built web apps, you have a giant advantage: **you know how they work inside.** You know the frontend, the backend, the database, the APIs, the cookies, the tokens.

That means when you attack a web app, you're not guessing — **you know where the weak points are** because you've programmed them:
- You know frontend (JavaScript) validations can be bypassed → you attack the backend directly.
- You know how sessions and JWTs work → you look for how to steal or forge them.
- You know how REST APIs are built → you test every endpoint, change every parameter.

**Your key tool:** **Burp Suite** (free Community version). It's a proxy that intercepts and modifies all traffic between the browser and the web. For a programmer it's like having an HTTP traffic debugger. You'll love it.

**Your practice ground:** **DVWA** and **PortSwigger Web Security Academy** (free, the best). Built for you to break web apps legally.

---

## ⚙️ Automate EVERYTHING (your other advantage)

Those who don't program do things by hand. You automate them. In security that's gold:

- Automate reconnaissance (scan, find subdomains, gather info) with scripts.
- Chain tools (one's output feeds the next).
- Process thousands of results with code instead of by eye.
- Build your own "framework" of custom tools.

While someone else spends hours clicking, you run a script and go for coffee. That's the difference programming gives you.

---

## 📚 Python libraries you'll love

As a dev, you'll appreciate these:
- **`requests`** — to talk to web apps and APIs.
- **`scapy`** — create and manipulate network packets by hand (super powerful).
- **`beautifulsoup4`** — extract data from pages (scraping).
- **`pwntools`** — the framework for exploiting and CTFs (binary exploitation).
- **`paramiko`** — SSH from Python.
- **`impacket`** — Windows network protocols.

---

## 🎮 How to practice (leveraging that you code)

1. **Rewrite tools that already exist.** Make your own scanner, fuzzer, subdomain finder. Understanding inside = mastering.
2. **CTFs (Capture The Flag):** legal and fun hacking competitions. **picoCTF** is perfect to start, and there are many programming/exploiting ones you'll love.
3. **pwntools + binary exploitation:** if you like low-level challenges, this world is addictive for programmers.
4. **Bug bounty (later):** platforms like HackerOne **PAY you** to find flaws in real web apps, legally. Your programmer skill gives you a real edge here.

---

## 🧭 Your accelerated path (because you already code)

You don't need the slow road. This is your shortcut:

1. **Quick fundamentals:** networking and Linux (read `book-networking.md` and `book-linux-from-scratch.md` — you'll fly because you already think in logic).
2. **Jump to your thing:** write your first port scanner TODAY (the one above). You'll feel the click.
3. **Web hacking:** install Burp, get into PortSwigger Academy. Your dev background makes it easy.
4. **CTFs:** picoCTF to practice exploiting and reversing.
5. **Create, create, create:** your best way to learn is building tools. You can do it, others can't.

---

## 🔍 Your unique superpower: secure code review

Almost no one else can do this, but you CAN: **read source code looking for vulnerabilities.** Companies pay very well for this.

When you review code with a security eye, you look for dangerous patterns:

```python
# 🚩 RED FLAGS that jump out to a dev-hacker:

eval(user_input)                    # arbitrary code execution
os.system(f"ping {user_input}")     # command injection
query = f"SELECT * WHERE id={id}"   # SQL injection
pickle.loads(user_data)             # insecure deserialization
element.innerHTML = userData        # XSS
open(user_path)                     # path traversal
password == stored_password         # non-constant comparison (timing)
DEBUG = True                        # in production = info leak
api_key = "sk-1234..."              # hardcoded secret
```

**Tools that automate this (SAST):**
```bash
semgrep --config=auto .      # finds vulnerable patterns in your code ⭐
bandit -r project/           # security analysis for Python
gitleaks detect              # finds secrets/API keys in the repo
trufflehog git <repo>        # secrets in git history
```

**Your edge:** while a pentester without code guesses from outside (black box), you read the code and see the bug directly (white box). That makes you invaluable in **bug bounty**, **audits**, and **DevSecOps**.

---

## 💬 The truth

Many hackers would give anything for your programming background. Most had to learn it the hard way, later, suffering. You already have it.

You just need to **flip your mind**: from building to breaking, from trusting input to distrusting it, from "how do I make it work?" to "how do I make it fail?".

Once you make that shift, you'll advance faster than almost everyone. **Code is your advantage. Use it.**

---

> ⬛ *Tools make users. Code makes hackers. You already write code — just learn to read it in reverse, looking for where it breaks. That's the whole difference.*
>
> **Continue with:** `book-python-hacking.md`, `book-web-hacking.md` and `book-cybersecurity.md` to go deeper.
