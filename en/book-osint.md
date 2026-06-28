# 🔎 OSINT BOOK
> Open Source Intelligence. The art of knowing everything using only public information.
> Version 2.0 — BUFFED: real tools, dorks, frameworks and pro techniques.

---

## What is OSINT?
**OSINT (Open Source Intelligence)** is getting valuable information from **public and legal sources**: social media, search engines, public records, metadata, breaches. Nothing is hacked — it's **investigated**. It's the base of:
- The **reconnaissance phase** of every pentest.
- Police and journalistic investigations.
- Identity verification and due diligence.
- Knowing your own exposure (defensive).

**The core idea:** people and organizations leave crumbs of information everywhere. Separately they say nothing; **together, they tell the whole story.** OSINT is the art of gathering those crumbs.

---

## 1. The OSINT methodology (not random searching)

Professional OSINT follows a cycle:

1. **Define the objective:** what exactly do I want to know?
2. **Collect:** gather data from all possible sources.
3. **Process:** organize what's collected (graphs, spreadsheets).
4. **Analyze:** connect dots, find patterns.
5. **Intelligence:** turn loose data into useful conclusions.

**Pivoting (the key technique):** each piece of data is a springboard to the next. An email → username → profiles → photos → metadata → location. You jump from one to another building the full picture. **Always look for the next pivot.**

---

## 2. People OSINT — tools by data type

### 👤 Username (the goldmine — people reuse the same nick)
```bash
sherlock username               # hundreds of platforms
maigret username                # 2500+ sites + extracts profile data
```
- **WhatsMyName** (whatsmyname.app) — very complete web version
- **Namechk / Instant Username** — quick availability across many sites

### 📧 Email
```bash
holehe email@example.com        # which services it's registered on
```
- **Have I Been Pwned** (haveibeenpwned.com) → which breaches it appeared in?
- **Hunter.io** → company emails + pattern (name.surname@)
- **Epieos** (epieos.com) → powerful web tool for email and phone
- **GHunt** → everything about a Google account
- Predictable corporate pattern: `name.surname@company.com`

### 📱 Phone
- **PhoneInfoga** → carrier, line type, footprint
- **Epieos** → linked apps (does it have WhatsApp/Telegram?)
- Reverse lookup: paste the number in Google with quotes, Truecaller, etc.

### 🪪 Real name
- Google with quotes: `"Name Surname"`, `"Name Surname" city`
- Public records, voter rolls, LinkedIn
- **Pipl, That'sThem, FastPeopleSearch** (US)

### 🌐 Social media (each exposes something different)
- **LinkedIn** → work, colleagues, job history
- **Instagram** → locations, routine, inner circle
- **Twitter/X** → opinions, activity hours (time zone)
- **Facebook** → family, friends, events
- **Photo analysis:** background, clothes, reflections, plates, signs → context and location
- **Schedule analysis:** when they post → their time zone and daily routine

### 🖼️ Reverse image search
- **Yandex** → the BEST for faces (better than Google)
- **Google Images, TinEye, Bing Visual Search**
- **PimEyes** → facial recognition (paid, powerful)
- **Search4Faces** → faces on social networks

---

## 3. Google Dorking — search like a pro

Operators that turn Google into a passive hacking tool:

```
site:example.com                    → that domain only
filetype:pdf                        → one file type only
intitle:"index of"                  → open directories
inurl:admin                         → URLs with "admin"
intext:"password"                   → pages with that word
site:example.com -www               → subdomains
cache:example.com                   → Google's saved version
related:example.com                 → similar sites
"Name Surname" filetype:pdf         → documents about someone
```

### Powerful combos (Google Hacking)
```
site:example.com filetype:pdf "confidential"
intitle:"index of" "backup"
inurl:"/wp-content/" site:example.com               → WordPress
filetype:env "DB_PASSWORD"                          → leaked configs
intitle:"index of" "parent directory" "*.sql"       → exposed databases
inurl:viewerframe?mode=                             → open IP cameras
site:pastebin.com "company.com"                     → leaks in pastes
site:github.com "company.com" password              → secrets in code
```
The **Google Hacking Database (GHDB)** on exploit-db.com has **thousands** of ready dorks. The same operators work on **Bing, DuckDuckGo, Yandex**.

---

## 4. Domain & infrastructure OSINT

```bash
whois example.com                   # who registered it, dates, contact
dig example.com ANY                 # DNS records (A, MX, NS, TXT)
dig -x 1.2.3.4                      # reverse DNS

# Subdomains (each = attack surface)
subfinder -d example.com
amass enum -d example.com
# crt.sh (web) — SSL certificates reveal hidden subdomains

# Mass collection (emails, hosts, employees)
theHarvester -d example.com -b all
```

### Key web sources
- **crt.sh** → SSL certificates (hidden subdomains)
- **Shodan.io** → "the Google of devices": servers, cameras, IoT, exposed panels. Dorks: `org:"company"`, `port:3389`, `webcam`
- **Censys.io** → similar to Shodan, very technical
- **Wayback Machine** (web.archive.org) → old versions (info they deleted)
- **DNSDumpster / SecurityTrails** → visual map of DNS infrastructure
- **ViewDNS.info** → 30+ DNS/IP tools on one site
- **BuiltWith / Wappalyzer** → technologies a site uses

---

## 5. Images, geolocation and metadata

### Metadata (EXIF) — what photos hide
```bash
exiftool photo.jpg
```
A photo can reveal: **exact GPS coordinates**, camera/phone model, date/time, editing software, even serial number. Office/PDF documents reveal author, company, internal paths.
> **Defense:** strip metadata before publishing → `exiftool -all= photo.jpg`

### Geolocation by image (GEOINT)
When there's NO GPS, you deduce location from content:
- Signs, languages, plates, architecture, vegetation, light poles, power outlets
- **Shadows** → time and orientation (north/south)
- Cross-reference with **Google Earth / Street View / Mapillary**
- **SunCalc** → sun position by date/time/place
- The game **GeoGuessr** trains exactly this skill

---

## 6. Breach OSINT

When a company gets hacked, the data ends up public. Investigate (legally):
- **Have I Been Pwned** → is your email/phone leaked?
- **Dehashed, IntelligenceX, Snusbase** → search leaked data (paid)
- **Pastebin, Ghostbin** → data sometimes leaked there
- **Google dork:** `site:pastebin.com "domain.com"`

**For defense:** check your emails in HIBP. If they show up → change those passwords NOW + enable 2FA.

---

## 7. Frameworks that automate EVERYTHING

When you want to scale from single tools to automated investigation:
- **SpiderFoot** → automates 200+ OSINT modules, generates graphs. The most complete free one.
- **Maltego** → the industry standard; maps relationships visually (free Community version)
- **recon-ng** → Metasploit-style framework for recon
- **OSINT Framework** (osintframework.com) → huge directory of tools by category
- **OBSIDIAN** (github.com/nadiee12312-lgtm/Obsidian-framework) → local OSINT framework, free and open-source, with relationship graph and timeline

**The value is in CONNECTING**, not just collecting:
- **Graphs** (Maltego-style): nodes joined by relationships → reveal connections invisible in a list
- **Timelines:** order events → reveal patterns and routines
- **Correlation:** the same data (phone, username) on two sites → connects them

---

## 8. OPSEC — protect yourself while investigating

If you investigate, DON'T leave a trail yourself:
- **Don't use your personal account** (some networks notify who viewed — LinkedIn)
- **Sock puppet accounts:** fake identities dedicated to investigating, with no real data of yours, with their own email/phone
- **VPN or Tor** to hide your IP (`proxychains` to force tools through Tor)
- **VM or separate browser** only for investigations
- **Don't interact** with the target (no likes, no follows, no comments) — it gives you away
- **Beware of webhooks/links** the target controls (they can capture YOUR IP)

---

## 9. Ethics and legality (the line you DON'T cross)

OSINT uses **public** info, but that doesn't always make it legal or ethical:
- ✅ **Legal:** viewing public profiles, searching Google, reading public records
- ⚠️ **Gray area:** creating fake profiles, mass collection of personal data
- ❌ **Illegal:** harassing, stalking, doxxing, using data to harm, accessing the private

**Doxxing** (publishing someone's private data to harm them) is illegal and destroys lives. OSINT is used to **protect and investigate legitimately** — verifying a scam, knowing your own exposure, authorized investigations. That's the difference between an investigator and a criminal.

---

## 🎓 To practice
- **TraceLabs** → OSINT CTFs that help find missing people (OSINT for good)
- **Sherlock/Maigret on your own username** → see how exposed you are
- **GeoGuessr** → train geolocation
- **OSINT Dojo, Cipher387** → challenges and resources
- **Continue with:** `book-networking.md` (infrastructure) and `book-python-hacking.md` (automate your recon)

> ⬛ *Information wants to be free. OSINT is the art of listening to it. Use it to protect, not to harm.*
