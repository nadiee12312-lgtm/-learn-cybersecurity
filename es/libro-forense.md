# 🔬 LIBRO DE FORENSE DIGITAL E INCIDENT RESPONSE
> Cuando algo malo pasa, alguien tiene que reconstruir la verdad. Ese eres tú.
> Versión 2.0 — BUFFEADO: más comandos, artifacts y casos prácticos.

---

## ¿Qué es la forense digital?
La ciencia de **recolectar, preservar y analizar evidencia digital** para reconstruir qué pasó en un incidente: un hackeo, fraude, robo de datos. Responde: *¿cómo entraron? ¿qué hicieron? ¿qué se llevaron? ¿siguen dentro?* Es el trabajo de detective digital — de los más respetados y mejor pagados de la seguridad.

---

## 1. Los principios sagrados (rómpelos y la evidencia no sirve)

1. **No alterar el original:** trabajas sobre **copias**, nunca el original.
2. **Cadena de custodia:** documentar quién tocó la evidencia, cuándo y por qué. Sin huecos.
3. **Integridad verificable:** calcula el **hash** al recolectar. Si cambia, se alteró.
4. **Documentar TODO:** cada paso, comando y hallazgo. Si no está documentado, no pasó.
5. **Reproducibilidad:** otro analista debe llegar a lo mismo repitiendo tus pasos.

---

## 2. El orden de volatilidad — qué salvar primero

De lo más volátil (se pierde rápido) a lo más permanente:

1. **Registros/caché de CPU** (milisegundos) — casi imposible.
2. **RAM** ⭐ — se pierde al apagar. **CAPTÚRALA ANTES DE APAGAR.** Aquí vive el malware activo, contraseñas, conexiones, claves de cifrado.
3. **Estado de red** — conexiones activas, ARP, sockets.
4. **Procesos en ejecución.**
5. **Disco** — permanente pero borrable.
6. **Logs remotos / backups.**

**Error de novato #1:** apagar/reiniciar la máquina comprometida → pierdes la RAM (las mejores pistas). **Aísla de la red, NO apagues** hasta capturar la memoria.

---

## 3. Forense de memoria (RAM) con Volatility3

La RAM tiene lo que pasaba **en vivo**, incluso malware fileless (que no toca disco).

### Capturar la RAM
```bash
# Linux: AVML (Microsoft) o LiME
sudo ./avml memoria.lime
# Windows: WinPmem, DumpIt, FTK Imager
```

### Analizar — plugins esenciales (Windows)
```bash
vol -f mem.dmp windows.info                    # info del volcado
vol -f mem.dmp windows.pslist                  # procesos
vol -f mem.dmp windows.pstree                  # en árbol (padre/hijo)
vol -f mem.dmp windows.psscan                  # procesos OCULTOS (que se esconden de pslist)
vol -f mem.dmp windows.netscan                 # conexiones de red (¿C2?)
vol -f mem.dmp windows.cmdline                 # qué ejecutó cada proceso
vol -f mem.dmp windows.malfind                 # ⭐ código inyectado
vol -f mem.dmp windows.dlllist --pid 1234      # DLLs de un proceso
vol -f mem.dmp windows.handles --pid 1234      # archivos/registros que abre
vol -f mem.dmp windows.filescan                # archivos en memoria
vol -f mem.dmp windows.dumpfiles --pid 1234    # volcar un proceso para analizarlo
vol -f mem.dmp windows.hashdump                # hashes de contraseñas
vol -f mem.dmp windows.registry.printkey -K "..."   # claves de registro
```

### Linux
```bash
vol -f mem.lime linux.pslist
vol -f mem.lime linux.bash          # historial de bash en memoria
vol -f mem.lime linux.check_syscall # detectar hooks de rootkit
```

**Qué buscas:** procesos con nombres raros, sin padre legítimo, conexiones a IPs extrañas, código inyectado (`malfind`), powershell ofuscado/base64 en `cmdline`.

---

## 4. Forense de disco

### Crear una imagen forense (copia bit a bit)
```bash
sudo dd if=/dev/sdb of=imagen.dd bs=4M status=progress
sha256sum imagen.dd > imagen.hash        # probar integridad
# Mejor: dcfldd o ewfacquire (formato E01 con metadatos)
```
**Nunca** analices el disco original — analiza la imagen (móntala read-only).

### Analizar con Sleuth Kit / Autopsy
```bash
mmls imagen.dd                    # tabla de particiones
fls -r -m / imagen.dd > body.txt  # listar archivos (incl. borrados)
icat imagen.dd 12345 > archivo    # extraer un archivo por su inode
mactime -b body.txt -d > timeline.csv   # línea de tiempo de archivos
```
**Autopsy** (GUI) recupera archivos borrados, historial de navegador, hace timeline y búsqueda por keywords.

### File carving (recuperar por firma, sin tabla de archivos)
```bash
foremost -i imagen.dd -o recuperados/
scalpel imagen.dd -o salida/
binwalk -e firmware.bin           # extraer de firmware/binarios
```

### MAC times (cuándo pasó cada cosa)
- **M**odified (contenido), **A**ccessed (lectura), **C**hanged (metadata)
- Cruzarlos reconstruye qué hizo el atacante y cuándo. Ojo: los avanzados los falsifican (*timestomping* — compáralos con el `$MFT` o logs).

---

## 5. Forense de Windows (artifacts clave)

Windows deja MUCHAS huellas. Las joyas:
- **Registro:** `SAM`, `SYSTEM`, `SOFTWARE`, `NTUSER.DAT` → usuarios, programas, persistencia (claves `Run`)
- **Prefetch** (`C:\Windows\Prefetch`) → qué programas se ejecutaron y cuándo
- **Event Logs** (`.evtx`) → logins (event 4624/4625), creación de procesos (4688)
- **Amcache / ShimCache** → historial de ejecutables
- **$MFT** → metadatos de TODOS los archivos del NTFS
- **Jump Lists / LNK** → archivos abiertos recientemente
- **Browser artifacts** → historial, cookies, descargas
- Herramientas: **Eric Zimmerman's tools**, **RegRipper**, **Autopsy**

---

## 6. Network forensics

```bash
# Capturar tráfico en vivo
sudo tcpdump -i eth0 -w captura.pcap

# Analizar (Wireshark GUI, o tshark CLI)
tshark -r captura.pcap -Y "http.request"           # peticiones HTTP
tshark -r captura.pcap -Y "dns" -T fields -e dns.qry.name   # consultas DNS
tshark -r captura.pcap --export-objects http,salida/        # extraer archivos transferidos
```
Busca: tráfico periódico a una IP rara (C2 beaconing), DNS a dominios sospechosos, transferencias grandes (exfiltración), protocolos raros.

---

## 7. Análisis de logs — la línea de tiempo del ataque

```bash
# Logins y autenticación (Linux)
grep "Accepted\|Failed" /var/log/secure
last        # logins exitosos
lastb       # intentos fallidos (fuerza bruta)

# Rango de tiempo
journalctl -S "2026-06-26 00:00" -U "2026-06-26 23:59"
journalctl -u sshd

# IPs que más atacaron por SSH
grep "Failed password" /var/log/secure | grep -oE "[0-9.]{7,15}" | sort | uniq -c | sort -rn
```

**Correlación = el arte:** juntas logins fallidos + login exitoso + procesos lanzados + conexiones, ordenado en el tiempo. La historia se arma sola: *"03:14 la IP X probó 500 contraseñas, 03:16 entró, 03:18 descargó algo, 03:20 se conectó a un C2."*

---

## 8. Incident Response — el proceso (NIST/SANS, 6 fases)

1. **Preparación:** plan, herramientas, backups, logs activos, IDS vigilando — ANTES del incidente.
2. **Identificación:** ¿hay incidente real? ¿alcance? Confirmar con evidencia, no pánico.
3. **Contención:** aislar de la red (sin apagar — guarda la RAM); luego contención a largo plazo.
4. **Erradicación:** quitar el malware, cerrar la vía de entrada, resetear credenciales.
5. **Recuperación:** restaurar de backups limpios, vigilando que no regrese.
6. **Lecciones aprendidas:** documentar, mejorar defensas. La fase que la gente se salta y por la que la vuelven a hackear.

---

## 9. IOCs y YARA

**IOCs (Indicadores de Compromiso):** hashes de malware, IPs/dominios de C2, nombres de archivo raros, claves de registro/cron sospechosas, user-agents anómalos. Búscalos en **VirusTotal** y threat intel → quizás ya están ligados a un grupo conocido. Úsalos para cazar en TODAS tus máquinas (threat hunting).

### YARA — cazar malware por patrones
```yara
rule Malware_Ejemplo {
    meta:
        descripcion = "Detecta familia X"
    strings:
        $a = "comando_malicioso"
        $b = { 6A 40 68 00 30 00 00 }   // bytes específicos
        $c = "evil-c2.com" nocase
    condition:
        $a or $b or $c
}
```
```bash
yara mi_regla.yar archivo_sospechoso
yara -r mi_regla.yar /home/             # recursivo
vol -f mem.dmp yarascan.YaraScan --yara-file regla.yar   # YARA sobre la RAM
```

---

## 10. Tu kit de triage rápido

```bash
# Sistema sospechoso — triage en vivo
ps aux                    # procesos
ss -tunap                 # conexiones de red + procesos
last && lastb             # logins
sudo rkhunter --check     # rootkits
sudo chkrootkit
clamscan -r --infected /  # malware conocido
sudo lynis audit system   # estado general
journalctl -p err -S today
find / -mtime -1 -type f 2>/dev/null      # archivos modificados últimas 24h
crontab -l; ls /etc/cron*                 # persistencia
```

**REMnux** (distro de análisis de malware) tiene 200+ herramientas forenses listas.

---

## 🎓 Para practicar
- **CyberDefenders, Blue Team Labs Online** → escenarios de IR reales
- **DFIR.training, Volatility Labs** → retos de forense de memoria
- **The DFIR Report** (thedfirreport.com) → reconstrucciones de ataques reales paso a paso
- **Captura tu propia RAM** y analízala con volatility — práctica sin riesgo
- **Sigue con:** `libro-malware.md` y `libro-ciberseguridad.md`

> ⬛ *Todo atacante deja huellas. La forense es el arte de leerlas y contar la verdad.*
