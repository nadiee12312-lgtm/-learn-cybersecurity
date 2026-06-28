# 🌐 LIBRO DE REDES
> La base de TODA la ciberseguridad. Si no entiendes redes, las herramientas son magia que no controlas.
> Versión 2.0 — BUFFEADO: nmap a fondo, Wireshark y más comandos.

---

## ¿Por qué empezar por redes?
Hackear es, en el fondo, **hacer que las máquinas se comuniquen de formas no previstas**. Cada ataque viaja por la red: un escaneo, un exploit, un robo de datos, un C2. Sin entender cómo fluye la información, solo copias comandos. Este libro te da los cimientos.

---

## 1. El modelo de capas — cómo viaja un dato

### TCP/IP (el real, 4 capas)
| Capa | Qué hace | Ejemplos |
|------|----------|----------|
| **Aplicación** | Lo que usas | HTTP, DNS, FTP, SSH, SMTP |
| **Transporte** | Cómo se entrega | TCP, UDP |
| **Internet** | Cómo se enruta | IP, ICMP |
| **Enlace** | Físico y red local | Ethernet, MAC, ARP |

### OSI (teórico, 7 capas)
Físico → Enlace → Red → Transporte → Sesión → Presentación → Aplicación.
"Ataque de capa 7" = aplicación (HTTP); "capa 3/4" = IP/TCP. Lo oirás siempre.

**Encapsulación:** tus datos se meten en un segmento TCP → en un paquete IP → en una trama Ethernet. Al llegar, se desempaqueta al revés.

---

## 2. Direcciones IP

- **IPv4:** `192.168.1.50` (se están acabando). **IPv6:** `2001:db8::7334` (el futuro).
- **Públicas:** únicas en internet (te la da el ISP). **Privadas** (internas):
  - `10.0.0.0/8` · `172.16.0.0/12` · `192.168.0.0/16` (típica de casa)
- **NAT:** el router traduce tus IPs privadas a su única IP pública.

### Subnetting / CIDR
`/24` = primeros 24 bits son red, 8 para hosts = **254 máquinas**. `/16` = 65,534. `/24` = máscara `255.255.255.0`. Por eso `nmap 192.168.1.0/24` escanea las 254 máquinas de tu red.

---

## 3. Puertos (las puertas de cada máquina, 0-65535)

| Puerto | Servicio | | Puerto | Servicio |
|--------|----------|---|--------|----------|
| 21 | FTP | | 443 | HTTPS |
| 22 | SSH | | 445 | SMB (Windows) |
| 23 | Telnet ❌ | | 1433 | MSSQL |
| 25 | SMTP | | 3306 | MySQL |
| 53 | DNS | | 3389 | RDP |
| 80 | HTTP | | 5432 | PostgreSQL |
| 110/143 | POP3/IMAP | | 6379 | Redis |
| 161 | SNMP | | 8080 | HTTP-alt |

**Estados (nmap):** Open (servicio escuchando), Closed (responde sin servicio), Filtered (firewall bloquea).

---

## 4. TCP vs UDP

**TCP** (fiable, ordenado) — el **3-way handshake** (clave en escaneos): SYN → SYN-ACK → ACK. Lo usa web, SSH, correo.
**UDP** (dispara y olvida, rápido) — streaming, juegos, DNS, VoIP.

Un escaneo SYN (`nmap -sS`) manda solo el SYN sin completar la conexión → más sigiloso. UDP (`nmap -sU`) es lento porque no confirma.

---

## 5. nmap a fondo (tu herramienta #1 de red)

```bash
# Descubrimiento
nmap -sn 192.168.1.0/24            # ping scan: qué hosts están vivos
nmap 192.168.1.0/24                # escaneo básico (top 1000 puertos)

# Tipos de escaneo
nmap -sS objetivo                  # SYN (sigiloso, requiere sudo)
nmap -sT objetivo                  # connect (sin sudo)
nmap -sU objetivo                  # UDP (lento)
nmap -p- objetivo                  # TODOS los 65535 puertos

# Detección
nmap -sV objetivo                  # versiones de servicios
nmap -O objetivo                   # sistema operativo
nmap -A objetivo                   # todo (versión + OS + scripts + traceroute)

# Scripts (NSE) — esto es un curso entero
nmap --script vuln objetivo        # busca vulnerabilidades conocidas
nmap --script smb-enum-shares objetivo
nmap -sC objetivo                  # scripts por defecto

# Velocidad y sigilo
nmap -T4 objetivo                  # más rápido (-T0 lento/sigiloso a -T5 agresivo)
nmap -Pn objetivo                  # saltar ping (host que no responde a ping)
nmap -oN salida.txt objetivo       # guardar resultados
```

---

## 6. DNS — el directorio de internet

Traduce nombres a IPs. Consulta: tu PC pregunta al resolver → servidores raíz → `.com` → servidor del dominio.

### Registros (con `dig`)
A (IPv4), AAAA (IPv6), MX (correo), NS (servidores DNS), TXT (SPF/DKIM), CNAME (alias), PTR (inverso).
```bash
dig dominio.com A
dig dominio.com MX
dig -x 8.8.8.8            # inverso (¿de quién es esta IP?)
dig dominio.com ANY
dig @8.8.8.8 dominio.com  # preguntar a un DNS específico
host dominio.com
nslookup dominio.com
```
**Para seguridad:** DNS revela subdominios, infraestructura, correo. Primera parada del recon. **DNS spoofing** te manda a una IP falsa; **DNS tunneling** esconde datos robados en consultas DNS.

---

## 7. La red local — ARP, MAC, DHCP

- **MAC:** identificador físico único de cada tarjeta (`a4:5e:60:c3:1f:9b`). Se puede falsear (*MAC spoofing*).
- **ARP:** traduce IP local ↔ MAC. No verifica nada → un atacante responde "¡yo soy esa IP!" y desvía tu tráfico = **ARP spoofing**, base del **MITM** en red local (bettercap, ettercap).
- **DHCP:** te asigna una IP automáticamente al conectarte. El router es el servidor DHCP de casa.

```bash
ip neigh        # tabla ARP (qué IPs↔MACs conoce tu máquina)
arp -a
```

---

## 8. HTTP/HTTPS

**Petición:** método (GET/POST/PUT/DELETE) + ruta + headers (User-Agent, Cookie, Host) + cuerpo.
**Códigos:** 2xx éxito · 3xx redirección · 4xx error cliente (403/404) · 5xx error servidor (500).
**HTTPS** = HTTP dentro de TLS. Un atacante en tu WiFi lee HTTP pero no HTTPS.
**Cookies de sesión:** tras login, el servidor te da un token; robarlo = robar la sesión (*session hijacking*).

---

## 9. Wireshark — ver el tráfico real

```bash
sudo tcpdump -i wlan0 -n              # captura cruda en terminal
sudo tcpdump -i wlan0 -w captura.pcap # guardar a archivo
wireshark                             # análisis visual
```

### Filtros de Wireshark (esenciales)
```
http                          → solo HTTP
http.request.method == "POST" → solo POST (¡aquí van las contraseñas en HTTP!)
ip.addr == 192.168.1.50       → tráfico de/hacia una IP
tcp.port == 443               → un puerto
dns                           → consultas DNS
tcp.flags.syn == 1            → paquetes SYN (escaneos)
frame contains "password"     → paquetes con esa palabra
```
**Truco:** clic derecho en un paquete → "Follow → TCP Stream" → ves toda la conversación.

---

## 10. Comandos de red esenciales

```bash
ip a                 # tus IPs e interfaces
ip route             # tu gateway
cat /etc/resolv.conf # tus DNS
ss -tunap            # conexiones activas + procesos (¿algo raro?)
ping 8.8.8.8         # ¿hay internet? (ICMP)
traceroute google.com # qué saltos da tu tráfico
curl -sI http://sitio # headers HTTP
nc -lvnp 4444        # escuchar en un puerto (recibir conexiones)
```

---

## 11. Defensa de red
- **Firewall:** filtra tráfico por puerto/IP (`firewalld`, `ufw`, `iptables`).
- **VLAN:** divide una red física en lógicas (aísla dispositivos).
- **DMZ:** zona para servidores públicos, separada de la red interna.
- **IDS/IPS:** detecta (Suricata, Snort) o bloquea tráfico malicioso.
- **VPN:** túnel cifrado entre redes (WireGuard, Tailscale, OpenVPN).
- **Segmentación:** separar en zonas → si cae una, no caen todas.

---

## 🎓 Para practicar
- **Mapea tu propia red:** `nmap -sV 192.168.1.0/24` y entiende cada dispositivo.
- **Espía tu tráfico:** Wireshark + navega a una web HTTP (ves todo) y una HTTPS (cifrada).
- **Subnetting:** practica en subnetting.org hasta que `/24`, `/16` te salgan solos.
- **Sigue con:** `libro-linux.md` y `libro-web-hacking.md`.

> ⬛ *Las redes son el campo de batalla. Conócelo mejor que el enemigo.*
