# 🔐 CRYPTOGRAPHY BOOK
> The art of protecting information. Understand how the world is encrypted and how weak crypto breaks.
> Version 2.0 — BUFFED: more commands, CTF attacks and practice.

---

## Why is cryptography the heart of security?
Every browser padlock, every WhatsApp, every software signature — that's cryptography. It protects **confidentiality** (no one reads), **integrity** (no one modifies), and **authenticity** (knowing who sent it). Without crypto, there's no security.

---

## 0. ⚠️ Encoding ≠ Encryption ≠ Hashing (confusion #1)

Before everything, tell these three apart (many people confuse them):
- **Encoding (Base64, hex, URL):** NOT security. Just represents data differently. **Reversible by anyone**, no key. (`echo aG9sYQ== | base64 -d`)
- **Encryption (AES, RSA):** protects with a **key**. Reversible only with the key.
- **Hashing (SHA-256):** **irreversible** fingerprint. There's no "un-hashing".

> If something "decodes" without a key, it was NOT encrypted — it was encoded. In CTFs, the first step is identifying which one.

---

## 1. The three pillars
1. **Confidentiality** → encryption.
2. **Integrity** → hashing.
3. **Authenticity** → digital signatures.
(+ non-repudiation and availability)

---

## 2. Hashing — the irreversible fingerprint

Properties: irreversible, deterministic, avalanche effect (change 1 letter → whole hash changes), collision-resistant.
```bash
echo -n "hello" | md5sum
echo -n "hello" | sha256sum
echo -n "hellO" | sha256sum     # totally different (avalanche)
```

### Algorithms
| Algorithm | Status | Use |
|-----------|--------|-----|
| **MD5** | ❌ BROKEN | Checksums only. Has collisions. |
| **SHA-1** | ❌ BROKEN | Obsolete since 2017. |
| **SHA-256 / SHA-512** | ✅ Secure | The current standard. |
| **bcrypt / argon2 / scrypt** | ✅ For passwords | Slow on purpose (resist brute force). |

### ⭐ Why passwords are hashed (not encrypted)
Systems **never** store your password in plaintext, they store its hash. On login, they hash what you type and compare. If they steal the DB, they don't have your passwords... **if the hash is strong.**

### Salting — against rainbow tables
If two users have the same password → same hash → the attacker notices. The **salt** (unique random value) is added before hashing: `hash = SHA256(salt + password)`. So equal passwords give different hashes, and **rainbow tables** (precomputed tables) are nullified.

### Cracking hashes (john and hashcat)
You don't "reverse" it — you try millions of candidates:
```bash
# Identify the hash type first
hashid hash.txt          # or hash-identifier

# hashcat (common -m modes)
hashcat -m 0    hash.txt rockyou.txt   # MD5
hashcat -m 100  hash.txt rockyou.txt   # SHA1
hashcat -m 1400 hash.txt rockyou.txt   # SHA256
hashcat -m 1800 hash.txt rockyou.txt   # sha512crypt (Linux /etc/shadow)
hashcat -m 1000 hash.txt rockyou.txt   # NTLM (Windows)
hashcat -m 22000 hash.txt rockyou.txt  # WPA2 (WiFi)

# With rules (mutate words: add numbers, capitals...)
hashcat -m 0 hash.txt rockyou.txt -r /usr/share/hashcat/rules/best64.rule

# john
john --wordlist=rockyou.txt hash.txt
john --show hash.txt
```
That's why a password should be **long and unique** — each extra character multiplies the time exponentially.

---

## 3. Symmetric encryption — one key

The same key encrypts and decrypts. Fast and strong, but: how do you share the key securely?

- **AES-256:** the world standard, practically unbreakable. Disks, WiFi, HTTPS.
- **ChaCha20:** modern and fast (mobile).
- **DES/3DES:** ❌ obsolete.

### Modes (important)
- **ECB:** ❌ insecure (patterns show — the "ECB penguin").
- **CBC:** chains blocks. Common, requires IV.
- **GCM:** ✅ recommended — encrypts AND verifies integrity.

```bash
openssl enc -aes-256-cbc -salt -in secret.txt -out secret.enc
openssl enc -d -aes-256-cbc -in secret.enc -out secret.txt
```

---

## 4. Asymmetric encryption — two keys (the magic)

Each person has a **pair**: **public** (you share it, encrypts) and **private** (only you, decrypts). What's encrypted with the public is decrypted only with the private → you never share secrets.

- **RSA:** classic, 2048/4096-bit keys.
- **ECC (elliptic curve):** modern, short keys with the same security.
- **Diffie-Hellman:** agree on a shared secret key over a public channel (without transmitting it). Brilliant.

### Digital signatures (the reverse of encrypting)
You sign with your **private** key, anyone verifies with your **public** → proves YOU created it and it wasn't altered. That's how certificates, software verification, and cryptocurrencies work.

---

## 5. TLS/HTTPS — crypto in action

The padlock combines it all: asymmetric handshake to agree on a **symmetric** session key → certificate signed by a **CA** that proves identity → traffic encrypted with AES. That's why an attacker on your WiFi sees THAT you visit an HTTPS site but not WHAT you do. Invalid certificate = possible MITM.

```bash
# Inspect a website's certificate
openssl s_client -connect example.com:443 -servername example.com
```

---

## 6. PKI — the trust infrastructure

**CA (Certificate Authority):** trusted entities that sign certificates (Let's Encrypt, DigiCert). **Chain of trust:** your browser trusts root CAs → which sign others → which sign websites' certificates. Valid certificate = a trusted CA signed it.

---

## 7. GPG — practical crypto in your terminal

```bash
gpg --full-generate-key                              # generate your key pair
gpg --export -a "Your Name" > public.asc             # export your public
gpg --encrypt --recipient person@mail.com file       # encrypt for someone
gpg --decrypt file.gpg                               # decrypt (with your private)
gpg --sign document.txt                              # sign
gpg --verify document.txt.sig                        # verify signature
gpg -c file.txt                                      # symmetric encryption with passphrase
```
Useful for encrypting backups, sending sensitive data, signing your software.

---

## 8. Cryptographic attacks
- **Brute force:** infeasible against AES-256, easy against short passwords.
- **Dictionary / Rainbow tables:** against unsalted hashes.
- **Collisions:** breaking MD5/SHA-1 (two files, same hash).
- **Man-in-the-Middle:** intercepting the key exchange (that's why certs matter).
- **Padding oracle / side-channel:** attacks on poorly made implementations.
- **The human link:** they rarely break the algorithm — they steal the key, the password, or exploit bad implementation.

**Golden rule:** *never invent your own encryption algorithm.* Use the standards (AES, RSA, SHA-256). Homemade always has holes you can't see.

---

## 9. Crypto in CTFs (what shows up most)
- **Identify:** is it encoding (base64/hex), classic cipher (Caesar, Vigenère, XOR) or modern (RSA, AES)?
- **Caesar/ROT:** try all 25 shifts. **Vigenère:** frequency analysis.
- **XOR:** if you know part of the plaintext, you recover the key (`plaintext XOR ciphertext = key`).
- **Weak RSA:** factor `n` if small (factordb.com), low `e`, close primes.
- **Key tool:** **CyberChef** (Google's web) — "Magic" auto-detects and decodes.

---

## 10. The future: post-quantum cryptography
Quantum computers could break RSA and ECC (Shor's algorithm). **Quantum-resistant** algorithms (Kyber, Dilithium) are already being developed. Keep it on the radar.

---

## 🎓 To practice
- **CryptoHack** (cryptohack.org) → the BEST crypto challenge resource, free.
- **CyberChef** → decode/encrypt/transform visually. Keep it in favorites.
- **Crypto in CTFs** (picoCTF, etc.) → almost all have crypto challenges.
- **Experiment** with `openssl`, `gpg`, `hashcat` and Python's `cryptography` library (`book-python-hacking.md`).

> ⬛ *Well-made cryptography is math that protects freedom. Respect it, understand it, and never invent your own.*
