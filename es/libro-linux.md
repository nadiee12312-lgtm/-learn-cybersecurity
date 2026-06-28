# 🐧 LIBRO DE LINUX PARA SEGURIDAD
> Linux es la herramienta y el campo de batalla. Domínalo y dominas el 80% de la ciberseguridad.
> Versión 2.0 — BUFFEADO: escalada de privilegios, enumeración y más.

---

## ¿Por qué Linux es el sistema de los hackers?
- **Transparente:** todo es un archivo, todo se inspecciona y controla.
- **Es el sistema de los servidores:** la mayoría de lo que vas a atacar/defender corre Linux.
- **Las herramientas viven aquí:** nmap, metasploit, todo nació en Linux.
- **La terminal es poder:** lo que en otros sistemas son 10 clics, aquí es un comando automatizable.

---

## 1. La filosofía: "todo es un archivo"
**Absolutamente todo** se representa como archivo: documentos, dispositivos (`/dev/sda` es tu disco), procesos, configuración. Lo lees, escribes y controlas todo con las mismas herramientas. La idea más poderosa del sistema.

---

## 2. El árbol de directorios

```
/           → la raíz, el origen de todo
├── /bin    → comandos esenciales
├── /boot   → arranque y kernel
├── /dev    → dispositivos
├── /etc    → ⭐ CONFIGURACIÓN del sistema
├── /home   → carpetas de usuarios
├── /opt    → software opcional
├── /proc   → info de procesos en vivo (virtual)
├── /root   → carpeta del superusuario
├── /tmp    → temporales
├── /usr    → programas y librerías
└── /var    → ⭐ datos variables: LOGS (/var/log)
```

**Lo que un analista vigila:**
- `/etc/passwd` → usuarios · `/etc/shadow` → hashes de contraseñas (solo root)
- `/var/log/` → huellas de TODO · `/home/*/.ssh/` → llaves SSH
- `/etc/crontab`, `/etc/cron.*` → tareas programadas (persistencia de malware)

---

## 3. Comandos de supervivencia

```bash
pwd; ls -la; cd /ruta; tree                  # moverte y ver
find / -name "*.conf" 2>/dev/null            # buscar archivos
which nmap; locate archivo                    # localizar
cat / less / head -n20 / tail -f archivo      # leer (tail -f = en vivo)
grep "error" log                              # buscar dentro
cp / mv / rm / mkdir / touch / ln -s          # manipular (rm = sin papelera, ¡cuidado!)
```

---

## 4. Permisos — el corazón de la seguridad

`ls -l` muestra: `-rwxr-xr--`
- 1er carácter: tipo (`-` archivo, `d` carpeta, `l` enlace)
- **rwx** dueño · **r-x** grupo · **r--** otros (r=leer 4, w=escribir 2, x=ejecutar 1)

```bash
chmod 755 archivo     # rwx / r-x / r-x
chmod 600 id_rsa      # solo el dueño (llaves privadas)
chmod +x script.sh    # hacer ejecutable
chown usuario archivo # cambiar dueño
```

### ⭐ SUID/SGID — donde la seguridad se rompe
Un binario **SUID** se ejecuta con permisos de su **dueño**, no de quien lo corre. Si es de root → lo corres tú pero actúa como root. **Base de muchas escaladas de privilegios.**
```bash
find / -perm -4000 -type f 2>/dev/null    # buscar binarios SUID (lo 1º que hace un atacante)
```
Si encuentras un SUID raro, consulta **GTFOBins** (gtfobins.github.io) → te dice cómo abusarlo para ser root.

---

## 5. Usuarios y privilegios

```bash
whoami; id                # quién soy, mis grupos
sudo comando; sudo -l     # ejecutar como root / ver qué puedo correr con sudo
su - usuario              # cambiar de usuario
cat /etc/passwd           # todos los usuarios
```
**root (UID 0)** = dios del sistema. **Mínimo privilegio:** nunca trabajes como root para todo. Una mala config de `sudoers` (`sudo -l`) es una escalada directa.

---

## 6. ⭐ Escalada de privilegios (privesc) — de usuario a root

Lo que haces tras entrar a un sistema con un usuario limitado. **Enumera primero:**

```bash
# Enumeración manual rápida
id; sudo -l                              # ¿qué puedo correr como root?
find / -perm -4000 -type f 2>/dev/null   # binarios SUID
cat /etc/crontab; ls -la /etc/cron.*     # tareas programadas (¿editables?)
uname -a                                 # versión del kernel (¿exploit conocido?)
ps aux | grep root                       # procesos de root (¿explotables?)
cat /etc/passwd                          # usuarios
ls -la /home/*/.ssh/                     # llaves SSH
env; cat ~/.bash_history                 # variables, historial
```

**Vías comunes de privesc:**
- **sudo mal configurado:** `sudo -l` muestra un binario que puedes correr como root → GTFOBins te dice cómo abusarlo
- **SUID explotable:** un binario SUID en GTFOBins
- **Tareas cron editables:** un script que root ejecuta y tú puedes modificar
- **Kernel viejo:** un exploit público (DirtyCow, DirtyPipe, PwnKit...)
- **Credenciales en archivos:** contraseñas en `.bash_history`, configs, `.env`

**Herramientas que automatizan la enumeración:**
```bash
./linpeas.sh        # LinPEAS — lo escanea TODO y resalta vías de privesc ⭐
./linenum.sh        # LinEnum
```
> Practica privesc SOLO en máquinas de CTF (HackTheBox, TryHackMe) o tu lab.

---

## 7. Procesos

```bash
ps aux                  # todos los procesos
ps aux --sort=-%cpu     # por consumo de CPU
top / htop              # monitor en vivo
kill -9 PID             # matar a la fuerza
comando &; nohup comando &   # en segundo plano
```
**Seguridad:** ante un compromiso, `ps aux` buscando procesos raros (nombres extraños, mucha CPU, red). El malware corre como proceso — ahí lo cazas.

---

## 8. Red desde la terminal

```bash
ip a; ip route          # IPs y gateway
ss -tunap               # conexiones + procesos (¿C2?)
curl http://sitio; wget url
ssh usuario@ip; scp archivo user@ip:/ruta
nc -lvnp 4444           # escuchar (recibir shells)
```

---

## 9. Logs — donde viven las huellas

```bash
journalctl -xe                 # eventos con detalle
journalctl -p err -S today     # errores de hoy
journalctl -u sshd -f          # un servicio, en vivo
last; lastb                    # logins exitosos / fallidos
grep "Failed" /var/log/secure  # intentos de ataque (Fedora)
grep "Failed" /var/log/auth.log # (Debian/Ubuntu)
```
**Un atacante borra logs** (anti-forense). Por eso los pros mandan logs a un **SIEM** central (Wazuh, Graylog) donde el atacante no los toca.

---

## 10. Bash scripting — automatiza tu poder

```bash
#!/bin/bash
OBJETIVO=$1
[ -z "$OBJETIVO" ] && { echo "Uso: $0 <ip>"; exit 1; }

echo "[*] Escaneando $OBJETIVO..."
nmap -sV "$OBJETIVO" -oN "scan_$OBJETIVO.txt"
if grep -q "80/tcp\|443/tcp" "scan_$OBJETIVO.txt"; then
    echo "[+] Web encontrada, lanzando nikto..."
    nikto -h "$OBJETIVO"
fi
```
**Claves de bash:** `$1` argumentos · `$(cmd)` captura salida · `|` pipe · `>` sobrescribe `>>` añade · `&&` si éxito `||` si falla · `for x in lista; do ... done`.

---

## 11. Hardening — blindar un Linux

1. **Actualiza:** `sudo apt update && apt upgrade` (o `dnf update`)
2. **Firewall:** `ufw` / `firewalld` — cierra puertos innecesarios
3. **fail2ban:** bloquea fuerza bruta automáticamente
4. **SSH seguro:** sin login de root, solo llaves (no contraseñas), `MaxAuthTries` bajo
5. **Mínimo software** = menos superficie de ataque
6. **Audita:** `sudo lynis audit system` (score + recomendaciones)
7. **Rootkits:** `sudo rkhunter --check`, `chkrootkit`
8. **Sin `chmod 777`**, mínimo privilegio siempre
9. **Backups 3-2-1** = la única defensa real contra ransomware

```bash
sudo lynis audit system                       # score de seguridad
find / -perm -4000 -type f 2>/dev/null         # SUID
ss -tulpn                                      # qué está escuchando
last -10                                       # últimos logins
```

---

## 🎓 Para practicar
- **OverTheWire: Bandit** → el MEJOR wargame para dominar la terminal, gratis. Empieza HOY.
- **GTFOBins** (gtfobins.github.io) → cómo binarios normales escalan privilegios.
- **HackTheBox / TryHackMe** → practica privesc real en máquinas legales.
- **Linux Journey** (linuxjourney.com) → curso interactivo gratis.
- **Sigue con:** `libro-redes.md` y `libro-web-hacking.md`.

> ⬛ *La terminal no es difícil, es honesta. Hace exactamente lo que le dices. Aprende a decírselo bien.*
