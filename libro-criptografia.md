# 🔐 LIBRO DE CRIPTOGRAFÍA
> El arte de proteger información. Entiende cómo se cifra el mundo y cómo se rompe el cifrado débil.
> Versión 2.0 — BUFFEADO: más comandos, ataques de CTF y práctica.

---

## ¿Por qué la criptografía es el corazón de la seguridad?
Cada candado del navegador, cada WhatsApp, cada firma de software — es criptografía. Protege la **confidencialidad** (que nadie lea), la **integridad** (que nadie modifique) y la **autenticidad** (saber quién lo mandó). Sin cripto, no hay seguridad.

---

## 0. ⚠️ Codificación ≠ Cifrado ≠ Hashing (la confusión #1)

Antes de todo, distingue estas tres (mucha gente las confunde):
- **Codificación (Base64, hex, URL):** NO es seguridad. Solo representa datos de otra forma. **Reversible por cualquiera**, sin clave. (`echo aG9sYQ== | base64 -d`)
- **Cifrado (AES, RSA):** protege con una **clave**. Reversible solo con la clave.
- **Hashing (SHA-256):** huella **irreversible**. No hay "des-hashear".

> Si algo se "decodifica" sin clave, NO estaba cifrado — estaba codificado. En CTFs, lo primero es identificar cuál es.

---

## 1. Los tres pilares
1. **Confidencialidad** → cifrado.
2. **Integridad** → hashing.
3. **Autenticidad** → firmas digitales.
(+ no repudiación y disponibilidad)

---

## 2. Hashing — la huella irreversible

Propiedades: irreversible, determinista, efecto avalancha (cambias 1 letra → cambia todo), resistente a colisiones.
```bash
echo -n "hola" | md5sum
echo -n "hola" | sha256sum
echo -n "holA" | sha256sum     # totalmente distinto (avalancha)
```

### Algoritmos
| Algoritmo | Estado | Uso |
|-----------|--------|-----|
| **MD5** | ❌ ROTO | Solo checksums. Tiene colisiones. |
| **SHA-1** | ❌ ROTO | Obsoleto desde 2017. |
| **SHA-256 / SHA-512** | ✅ Seguro | El estándar actual. |
| **bcrypt / argon2 / scrypt** | ✅ Para contraseñas | Lentos a propósito (resisten fuerza bruta). |

### ⭐ Por qué las contraseñas se hashean (no se cifran)
Los sistemas **nunca** guardan tu contraseña en texto, guardan su hash. Al iniciar sesión, hashean lo que escribes y comparan. Si roban la BD, no tienen tus contraseñas... **si el hash es fuerte.**

### Salting (sal) — contra rainbow tables
Si dos usuarios tienen la misma contraseña → mismo hash → el atacante lo nota. La **sal** (valor aleatorio único) se añade antes de hashear: `hash = SHA256(salt + contraseña)`. Así contraseñas iguales dan hashes distintos, y se anulan las **rainbow tables** (tablas precalculadas).

### Crackear hashes (john y hashcat)
No se "revierte" — se prueban millones de candidatos:
```bash
# Identificar el tipo de hash primero
hashid hash.txt          # o hash-identifier

# hashcat (modos -m comunes)
hashcat -m 0    hash.txt rockyou.txt   # MD5
hashcat -m 100  hash.txt rockyou.txt   # SHA1
hashcat -m 1400 hash.txt rockyou.txt   # SHA256
hashcat -m 1800 hash.txt rockyou.txt   # sha512crypt (Linux /etc/shadow)
hashcat -m 1000 hash.txt rockyou.txt   # NTLM (Windows)
hashcat -m 22000 hash.txt rockyou.txt  # WPA2 (WiFi)

# Con reglas (muta las palabras: añade números, mayúsculas...)
hashcat -m 0 hash.txt rockyou.txt -r /usr/share/hashcat/rules/best64.rule

# john
john --wordlist=rockyou.txt hash.txt
john --show hash.txt
```
Por eso una contraseña debe ser **larga y única** — cada carácter extra multiplica el tiempo exponencialmente.

---

## 3. Cifrado simétrico — una sola llave

La misma llave cifra y descifra. Rápido y fuerte, pero: ¿cómo compartes la llave de forma segura?

- **AES-256:** el estándar mundial, prácticamente irrompible. Discos, WiFi, HTTPS.
- **ChaCha20:** moderno y rápido (móviles).
- **DES/3DES:** ❌ obsoletos.

### Modos (importante)
- **ECB:** ❌ inseguro (se notan patrones — el "pingüino ECB").
- **CBC:** encadena bloques. Común, requiere IV.
- **GCM:** ✅ recomendado — cifra Y verifica integridad.

```bash
openssl enc -aes-256-cbc -salt -in secreto.txt -out secreto.enc
openssl enc -d -aes-256-cbc -in secreto.enc -out secreto.txt
```

---

## 4. Cifrado asimétrico — dos llaves (la magia)

Cada persona tiene un **par**: **pública** (la repartes, cifra) y **privada** (solo tú, descifra). Lo que se cifra con la pública, solo se descifra con la privada → nunca compartes secretos.

- **RSA:** clásico, llaves 2048/4096 bits.
- **ECC (curva elíptica):** moderno, llaves cortas con la misma seguridad.
- **Diffie-Hellman:** acuerdan una llave secreta compartida por un canal público (sin transmitirla). Brillante.

### Firmas digitales (al revés que cifrar)
Firmas con tu **privada**, cualquiera verifica con tu **pública** → prueba que TÚ lo creaste y que no se alteró. Así funcionan los certificados, la verificación de software y las criptomonedas.

---

## 5. TLS/HTTPS — cripto en acción

El candado combina todo: handshake asimétrico para acordar una llave **simétrica** de sesión → certificado firmado por una **CA** que prueba la identidad → tráfico cifrado con AES. Por eso un atacante en tu WiFi ve QUE visitas un HTTPS pero no QUÉ haces. Certificado inválido = posible MITM.

```bash
# Inspeccionar el certificado de una web
openssl s_client -connect ejemplo.com:443 -servername ejemplo.com
```

---

## 6. PKI — la infraestructura de confianza

**CA (Certificate Authority):** entidades de confianza que firman certificados (Let's Encrypt, DigiCert). **Cadena de confianza:** tu navegador confía en CAs raíz → que firman a otras → que firman certificados de webs. Certificado válido = una CA confiable lo firmó.

---

## 7. GPG — cripto práctica en tu terminal

```bash
gpg --full-generate-key                              # genera tu par de llaves
gpg --export -a "Tu Nombre" > publica.asc            # exporta tu pública
gpg --encrypt --recipient persona@mail.com archivo   # cifra para alguien
gpg --decrypt archivo.gpg                            # descifra (con tu privada)
gpg --sign documento.txt                             # firma
gpg --verify documento.txt.sig                       # verifica firma
gpg -c archivo.txt                                   # cifrado simétrico con passphrase
```
Útil para cifrar backups, mandar datos sensibles, firmar tu software.

---

## 8. Ataques criptográficos
- **Fuerza bruta:** inviable contra AES-256, fácil contra contraseñas cortas.
- **Diccionario / Rainbow tables:** contra hashes sin sal.
- **Colisiones:** romper MD5/SHA-1 (dos archivos, mismo hash).
- **Man-in-the-Middle:** interceptar el intercambio de llaves (por eso importan los certs).
- **Padding oracle / side-channel:** ataques a implementaciones mal hechas.
- **El eslabón humano:** casi nunca rompen el algoritmo — roban la llave, la contraseña, o explotan mala implementación.

**Regla de oro:** *nunca inventes tu propio algoritmo de cifrado.* Usa los estándares (AES, RSA, SHA-256). Lo casero siempre tiene agujeros que no ves.

---

## 9. Cripto en CTFs (lo que más se ve)
- **Identificar:** ¿es codificación (base64/hex), cifrado clásico (César, Vigenère, XOR) o moderno (RSA, AES)?
- **César/ROT:** prueba los 25 desplazamientos. **Vigenère:** análisis de frecuencia.
- **XOR:** si conoces parte del texto, recuperas la clave (`texto XOR cifrado = clave`).
- **RSA débil:** factorizar `n` si es pequeño (factordb.com), `e` bajo, primos cercanos.
- **Herramienta clave:** **CyberChef** (la web de Google) — "Magic" detecta y decodifica automáticamente.

---

## 10. El futuro: criptografía post-cuántica
Las computadoras cuánticas podrían romper RSA y ECC (algoritmo de Shor). Ya se desarrollan algoritmos **resistentes a lo cuántico** (Kyber, Dilithium). Tenlo en el radar.

---

## 🎓 Para practicar
- **CryptoHack** (cryptohack.org) → el MEJOR recurso de retos de cripto, gratis.
- **CyberChef** → decodifica/cifra/transforma visualmente. Tenla en favoritos.
- **Cripto en CTFs** (picoCTF, etc.) → casi todos tienen retos de cripto.
- **Experimenta** con `openssl`, `gpg`, `hashcat` y la librería `cryptography` de Python (`libro-python-hacking.md`).

> ⬛ *La criptografía bien hecha es matemática que protege la libertad. Respétala, entiéndela, y nunca inventes la tuya.*
