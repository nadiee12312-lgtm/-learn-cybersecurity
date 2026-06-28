# 🛡️ LIBRO DE CIBERSEGURIDAD
> Guía completa: el arsenal esencial, las herramientas del mundo, cómo atacan los criminales y cómo defenderte y rastrearlos de vuelta.
> Versión 2.0 — De cero a profesional. El libro maestro de la academia.

---

## ⚠️ ANTES DE EMPEZAR — Lee esto

Este libro enseña ciberseguridad **ofensiva y defensiva** con un único propósito: que aprendas cómo funcionan los ataques **para poder defenderte de ellos**. Eso se llama "pensar como atacante para defender como profesional".

**Las reglas de oro, sin excepción:**

1. **Solo atacas lo que es tuyo o lo que tienes permiso escrito para atacar.** Tu propio lab, máquinas vulnerables hechas para practicar (DVWA, HackTheBox, TryHackMe, VulnHub), o un cliente que te contrató y firmó un papel.
2. **Atacar un sistema ajeno sin permiso es un delito**, aunque "solo estuvieras probando". En México (Código Penal Federal art. 211 bis) y en EE.UU. (CFAA) hay cárcel real por esto.
3. **"Hackear de vuelta" (hack back) al que te atacó también es ilegal** en casi todo el mundo. En este libro aprenderás a **rastrear y reportar**, no a contraatacar. Esa es la diferencia entre un profesional y un criminal.
4. El conocimiento no es el delito. **El uso sin permiso, sí.**

Un profesional de seguridad usa exactamente las mismas herramientas que un criminal. Lo único que los separa es el **permiso** y la **intención**. Tú vas a ser de los buenos.

---

## 📖 Cómo pensar la ciberseguridad

Antes de las herramientas, la mentalidad. Toda la ciberseguridad gira alrededor de dos equipos:

- **🔴 Red Team (ofensivo):** simula al atacante. Busca agujeros antes de que un criminal los encuentre.
- **🔵 Blue Team (defensivo):** detecta, responde y limpia. Vigila, alerta y echa al intruso.
- **🟣 Purple Team:** la mezcla — ataca y defiende a la vez para mejorar ambos lados.

### Las fases de un ataque — La "Cyber Kill Chain"

Todo ataque, del más simple al más sofisticado, sigue más o menos estos pasos (modelo de Lockheed Martin):

1. **Reconocimiento** — el atacante investiga al objetivo (OSINT, escaneo de puertos).
2. **Armamento (Weaponization)** — prepara el arma: un exploit, un documento con malware.
3. **Entrega (Delivery)** — la manda: email de phishing, USB infectado, enlace falso.
4. **Explotación** — el arma dispara: se aprovecha una vulnerabilidad.
5. **Instalación** — el malware se instala y se queda.
6. **Comando y Control (C2)** — el atacante controla la máquina a distancia.
7. **Acciones sobre el objetivo** — roba datos, cifra archivos (ransomware), espía.

Si rompes **cualquier eslabón** de esta cadena, el ataque falla. Por eso defender es romper eslabones.

### MITRE ATT&CK — el diccionario del enemigo

**MITRE ATT&CK** (attack.mitre.org) es una enciclopedia gratuita que cataloga **todas las técnicas reales** que usan los atacantes, con códigos como `T1566` (Phishing) o `T1059` (ejecución por línea de comandos). Los defensores profesionales lo usan para saber qué buscar. Memorízalo como referencia: cuando veas un ataque, lo puedes clasificar con ATT&CK.

---

# 🗡️ PARTE 1 — EL ARSENAL ESENCIAL

Aquí va, una por una, cada herramienta esencial del oficio: qué es, para qué sirve, qué te enseña y un ejemplo de uso. Organizadas por la fase del ataque/defensa donde se usan.

---

## 🔍 1.1 — Reconocimiento y OSINT

El reconocimiento es la **primera fase de todo**. Sin información, no hay ataque ni defensa. Estas herramientas reúnen datos públicos del objetivo.

### `whois`
- **Qué es:** consulta la base de datos pública de dominios y rangos de IP.
- **Para qué sirve:** saber quién registró un dominio, cuándo, en qué empresa, con qué servidores DNS y, a veces, email y teléfono de contacto.
- **Qué enseña:** que internet tiene un "registro civil" público. Casi todo dominio deja rastro de su dueño.
- **Ejemplo:** `whois ejemplo.com`
- **Defensa:** las empresas usan "WHOIS privacy" para ocultar estos datos. Si los tuyos están expuestos, es info que un atacante aprovecha.

### `dig`
- **Qué es:** consulta servidores DNS (el "directorio telefónico" de internet que traduce nombres a IPs).
- **Para qué sirve:** ver registros A (IP), MX (correo), NS (servidores de nombres), TXT (SPF/DKIM), CNAME (alias).
- **Qué enseña:** cómo funciona DNS por dentro, que es la base de toda comunicación en internet.
- **Ejemplo:** `dig ejemplo.com ANY` · `dig MX ejemplo.com`
- **Truco ofensivo:** los registros DNS revelan subdominios, proveedores de correo y a veces servidores internos mal configurados.

### `theHarvester`
- **Qué es:** recolector automático de OSINT (en Kali/Parrot).
- **Para qué sirve:** junta emails, subdominios, nombres de empleados e IPs de un objetivo desde buscadores y fuentes públicas.
- **Qué enseña:** cuánta "superficie" expone una empresa sin darse cuenta.
- **Ejemplo:** `theHarvester -d ejemplo.com -b all`

### `sherlock`
- **Qué es:** busca un nombre de usuario en cientos de redes sociales y plataformas.
- **Para qué sirve:** ver en qué sitios existe un username (rastreo de identidad digital).
- **Qué enseña:** que reusar el mismo nick en todos lados te hace fácil de rastrear.
- **Ejemplo:** `sherlock nombredeusuario`

### `maigret`
- **Qué es:** el hermano mayor de Sherlock — busca en 2500+ sitios y extrae datos del perfil.
- **Para qué sirve:** investigación OSINT profunda de una persona por su username.
- **Ejemplo:** `maigret nombredeusuario`

### `exiftool`
- **Qué es:** lee y edita los **metadatos** de archivos (fotos, PDFs, documentos).
- **Para qué sirve:** una foto puede traer escondida la **ubicación GPS exacta**, el modelo de cámara/teléfono, fecha y hora. Un PDF puede traer el nombre real del autor y el software usado.
- **Qué enseña:** que los archivos "hablan" más de lo que crees. Es OSINT puro.
- **Ejemplo:** `exiftool foto.jpg`
- **Defensa:** borra metadatos antes de publicar archivos: `exiftool -all= foto.jpg`

### `nmap` — el rey del reconocimiento de red
- **Qué es:** el escáner de redes y puertos más famoso del mundo. Tu herramienta más importante.
- **Para qué sirve:** descubrir qué máquinas hay en una red, qué puertos tienen abiertos, qué servicios corren (y sus versiones), e incluso qué sistema operativo usan.
- **Qué enseña:** cómo "ve" un atacante tu red. Cada puerto abierto es una puerta potencial.
- **Ejemplos clave:**
  - `nmap -sV --open 192.168.1.0/24` → escanea toda tu red local, muestra puertos abiertos y versiones.
  - `nmap -sC -sV -p- objetivo` → escaneo completo con scripts y todos los puertos.
  - `nmap -O objetivo` → intenta detectar el sistema operativo.
  - `nmap --script vuln objetivo` → busca vulnerabilidades conocidas.
- **El NSE (Nmap Scripting Engine):** nmap trae cientos de scripts (`/usr/share/nmap/scripts/`) que detectan desde vulnerabilidades hasta credenciales por defecto. Esto solo ya es un curso entero.

### `RedTiger-Tools`
- **Qué es:** suite OSINT de reconocimiento (RedTiger-Tools, en GitHub).
- **Para qué sirve:** múltiples módulos de reconocimiento de personas, IPs y redes en una sola interfaz.

---

## 🔬 1.2 — Análisis de vulnerabilidades web

### `nikto`
- **Qué es:** escáner de servidores web.
- **Para qué sirve:** revisa un sitio web buscando archivos peligrosos, configuraciones inseguras, versiones viejas de software y miles de problemas conocidos.
- **Qué enseña:** que un servidor web mal configurado grita sus debilidades.
- **Ejemplo:** `nikto -h http://objetivo.com`
- **Nota:** es "ruidoso" — cualquier defensa decente lo detecta al instante. Bueno para aprender, malo para ser sigiloso.

### `sqlmap`
- **Qué es:** automatiza la detección y explotación de **inyección SQL** (SQLi), una de las vulnerabilidades web más peligrosas.
- **Para qué sirve:** si una web no filtra bien lo que escribes en un formulario, sqlmap puede llegar a leer (¡o borrar!) toda la base de datos: usuarios, contraseñas, todo.
- **Qué enseña:** por qué NUNCA se debe confiar en la entrada del usuario, y por qué existen las "consultas parametrizadas".
- **Ejemplo:** `sqlmap -u "http://objetivo.com/item?id=1" --dbs --batch`
- **Defensa:** validar y parametrizar todas las entradas, usar ORMs, principio de mínimo privilegio en la BD.

---

## 💥 1.3 — Explotación

### `msfconsole` (Metasploit Framework)
- **Qué es:** el framework de explotación más potente que existe. Una navaja suiza con miles de exploits, payloads y módulos.
- **Para qué sirve:** lanzar exploits contra vulnerabilidades conocidas, generar malware de prueba (payloads), establecer sesiones de control (Meterpreter), hacer post-explotación.
- **Qué enseña:** todo el ciclo de un ataque, de principio a fin, en un entorno controlado. Es LA herramienta para aprender explotación.
- **Conceptos que debes dominar:**
  - **Exploit:** el código que aprovecha la vulnerabilidad.
  - **Payload:** lo que se ejecuta tras explotar (ej. una shell inversa).
  - **Meterpreter:** un payload avanzado que te da control total: subir/bajar archivos, capturar teclas, webcam, etc.
- **Flujo básico:**
  ```
  msfconsole
  search tipo:exploit nombre_servicio
  use exploit/...
  set RHOSTS objetivo
  set PAYLOAD ...
  exploit
  ```
- **Practica SOLO en:** Metasploitable, DVWA, o máquinas de HackTheBox/TryHackMe.

### `searchsploit` + ExploitDB
- **Qué es:** buscador offline de exploits públicos (de exploit-db.com). Disponible en Kali.
- **Para qué sirve:** dado un software y versión, encontrar si ya existe un exploit público.
- **Ejemplo:** `searchsploit apache 2.4` · `searchsploit -m 12345` (copia el exploit).
- **Qué enseña:** que el software viejo y sin parchear es oro para un atacante. **Actualizar es defensa.**

---

## 🔑 1.4 — Contraseñas y cracking

Entender cómo se rompen las contraseñas es entender por qué deben ser largas y únicas.

### Concepto clave: hashes
Los sistemas no guardan tu contraseña en texto, guardan un **hash** (una huella matemática irreversible). Cuando inicias sesión, hashean lo que escribes y comparan. "Crackear" es probar millones de contraseñas, hashear cada una, y ver cuál coincide.

### `john` (John the Ripper)
- **Qué es:** crackeador de contraseñas clásico, muy versátil.
- **Para qué sirve:** romper hashes de contraseñas (de Linux, ZIP, PDF, etc.) por diccionario o fuerza bruta.
- **Ejemplo:** `john --wordlist=/opt/wordlists/rockyou.txt hashes.txt`
- **Qué enseña:** que una contraseña como "Maria123" cae en segundos.

### `hashcat`
- **Qué es:** el crackeador más rápido del mundo (usa GPU, aunque en tu CPU también funciona, más lento).
- **Para qué sirve:** romper casi cualquier tipo de hash a velocidad bruta.
- **Ejemplo:** `hashcat -m 0 -a 0 hashes.txt /opt/wordlists/rockyou.txt` (modo 0 = MD5).
- **Los "-m" son el tipo de hash:** 0=MD5, 100=SHA1, 1000=NTLM (Windows), 1800=sha512crypt (Linux), 22000=WPA2 (WiFi).
- **Qué enseña:** el poder real de la fuerza bruta moderna, y por qué 8 caracteres ya no bastan.

### `hydra`
- **Qué es:** crackeador de logins **en línea** (contra servicios vivos).
- **Para qué sirve:** probar usuarios/contraseñas contra SSH, FTP, RDP, formularios web, etc.
- **Ejemplo:** `hydra -l admin -P /opt/wordlists/rockyou.txt ssh://192.168.1.50`
- **Qué enseña:** por qué existen el bloqueo tras X intentos y la autenticación de dos factores (2FA).
- **Defensa:** fail2ban, 2FA, límites de intentos. **Nunca uses esto contra servicios que no sean tuyos.**

### `rockyou.txt`
- **Qué es:** la wordlist más famosa: 14 millones de contraseñas reales filtradas en 2009.
- **Para qué sirve:** el diccionario base para todo cracking. Está en `/opt/wordlists/rockyou.txt`.
- **Qué enseña:** que si tu contraseña está en rockyou, ya no es una contraseña.

---

## 🧬 1.5 — Ingeniería inversa y análisis de binarios

### `radare2` (r2)
- **Qué es:** framework de ingeniería inversa de código abierto. Desensamblador, debugger y analizador.
- **Para qué sirve:** abrir un programa compilado (un `.exe`, un binario de Linux, malware) y entender qué hace por dentro, sin tener el código fuente.
- **Qué enseña:** cómo piensa una computadora a bajo nivel (ensamblador), cómo se analiza malware y cómo se encuentran vulnerabilidades en software.
- **Ejemplo:** `r2 -A binario` luego `afl` (listar funciones), `pdf @main` (ver función main).
- **Nota:** curva de aprendizaje empinada, pero es de las habilidades mejor pagadas en seguridad.

### `Ghidra`
- **Qué es:** el desensamblador y **descompilador** de la NSA, liberado gratis. Está en `/opt/ghidra/`.
- **Para qué sirve:** lo mismo que radare2 pero con interfaz gráfica y un **descompilador** que convierte el ensamblador de vuelta a algo parecido a código C legible. Es ORO para analizar malware.
- **Qué enseña:** análisis de malware profesional, CTFs de reversing, encontrar backdoors.
- **Inicio:** `/opt/ghidra/ghidraRun`

### `binwalk`
- **Qué es:** analiza archivos binarios y firmware buscando archivos ocultos dentro (en Kali).
- **Para qué sirve:** extraer el contenido de imágenes de firmware de routers/IoT, encontrar archivos escondidos dentro de otros.
- **Ejemplo:** `binwalk -e firmware.bin`
- **Qué enseña:** análisis de dispositivos IoT y esteganografía.

---

## 🛡️ 1.6 — Defensa, auditoría y antimalware (tu lado Blue Team)

Estas ya son herramientas **defensivas** — las usas para proteger y revisar TU sistema.

### `clamscan` (ClamAV)
- **Qué es:** antivirus de código abierto.
- **Para qué sirve:** escanear archivos y carpetas buscando malware conocido.
- **Ejemplo:** `clamscan -r --infected /home` (recursivo, solo muestra infectados).
- **Qué enseña:** cómo funciona la detección por firmas.

### `rkhunter` (Rootkit Hunter)
- **Qué es:** detector de **rootkits** (malware que se esconde a nivel profundo del sistema).
- **Para qué sirve:** revisar si tu Linux tiene rootkits, backdoors o archivos del sistema modificados.
- **Ejemplo:** `sudo rkhunter --check`
- **Qué enseña:** cómo se ocultan los atacantes y cómo cazarlos.

### `lynis`
- **Qué es:** auditor de seguridad y "hardening" para sistemas Linux/Unix.
- **Para qué sirve:** revisa TODO tu sistema y te da un reporte con un "índice de endurecimiento" y recomendaciones concretas para asegurarlo.
- **Ejemplo:** `sudo lynis audit system`
- **Qué enseña:** hardening real. Es de lo mejor para aprender a blindar un servidor.

### `yara`
- **Qué es:** el "lenguaje" para escribir reglas que identifican malware por patrones.
- **Para qué sirve:** los analistas escriben reglas YARA que describen una familia de malware; luego escanean archivos o memoria buscando coincidencias. Es el estándar de la industria.
- **Ejemplo:** `yara mi_regla.yar archivo_sospechoso`
- **Qué enseña:** detección de amenazas a nivel profesional (threat hunting).

### `volatility3` (comando `vol`)
- **Qué es:** el framework de **análisis forense de memoria RAM** más importante.
- **Para qué sirve:** analizar un volcado de la memoria de una máquina para ver qué procesos corrían, qué conexiones de red había, qué malware estaba activo — incluso cosas que no dejan rastro en el disco.
- **Ejemplo:** `vol -f memoria.dmp windows.pslist` (lista procesos).
- **Qué enseña:** forense digital avanzado. Cuando un sistema es hackeado, la RAM tiene las respuestas.

### `suricata`
- **Qué es:** un **IDS/IPS** (Sistema de Detección/Prevención de Intrusiones) de alto rendimiento.
- **Para qué sirve:** vigila el tráfico de tu red en tiempo real y **alerta (o bloquea)** cuando ve algo malicioso — un escaneo, un exploit conocido, tráfico a un servidor C2.
- **Ejemplo:** `sudo suricata -i wlan0` (monitorea una interfaz).
- **Qué enseña:** detección de intrusiones en red, el corazón del Blue Team. Esta es la herramienta que te avisaría si alguien te ataca.

### `tcpdump`
- **Qué es:** capturador de paquetes de red por línea de comandos.
- **Para qué sirve:** ver, en crudo, todo el tráfico que entra y sale de tu máquina.
- **Ejemplo:** `sudo tcpdump -i wlan0 -n port 80`
- **Qué enseña:** cómo viaja realmente la información por la red. Base del análisis de tráfico.

---

## 🌐 1.7 — Anonimato y redes

### `tor`
- **Qué es:** la red de anonimato más famosa. Enruta tu tráfico por 3 servidores aleatorios cifrados.
- **Para qué sirve:** ocultar tu IP real y origen. Acceso a la red .onion.
- **Qué enseña:** cómo funciona el anonimato en internet (y sus límites — Tor no te hace invencible).

### `proxychains`
- **Qué es:** fuerza a cualquier programa a pasar por un proxy (o por Tor).
- **Para qué sirve:** anonimizar herramientas que no soportan proxy de forma nativa.
- **Ejemplo:** `proxychains nmap -sT objetivo` (escaneo a través de Tor).
- **Qué enseña:** encadenamiento de proxies y ofuscación de origen.

### `aircrack-ng`
- **Qué es:** la suite para auditoría de redes **WiFi**.
- **Para qué sirve:** capturar tráfico WiFi, capturar el "handshake" de una red WPA2 y crackear su contraseña.
- **Qué enseña:** por qué WPA2 con contraseña débil es inseguro, y por qué existe WPA3.
- **Nota:** necesita un adaptador WiFi con **modo monitor** (como la Alfa AWUS036ACS). Solo en TUS redes.

### `nc` (netcat) y `socat`
- **Qué son:** la "navaja suiza" de las conexiones de red. Crean conexiones TCP/UDP crudas.
- **Para qué sirven:** transferir archivos, crear shells (de prueba), escuchar puertos, depurar servicios.
- **Ejemplo:** `nc -lvnp 4444` (escucha en el puerto 4444 — típico para recibir una shell inversa de prueba).
- **Qué enseñan:** los fundamentos absolutos de cómo se comunican las máquinas. Esenciales en CTFs.

---

# 🌍 PARTE 2 — HERRAMIENTAS QUE NO TIENES (pero debes conocer)

Estas completan tu mapa mental de la profesión. La mayoría vive en distros de seguridad (Kali, Parrot, BlackArch) o las instalas cuando las necesites.

## Reconocimiento y escaneo
- **`masscan`** — escáner de puertos ultrarrápido: escanea TODO internet en minutos. Donde nmap es un bisturí, masscan es una metralleta. *(en BlackArch.)*
- **`gobuster` / `ffuf` / `feroxbuster`** — fuerza bruta de directorios y archivos ocultos en webs (`/admin`, `/backup`, etc.). Indispensables en pentesting web. *(En BlackArch.)*
- **`nuclei`** — escáner de vulnerabilidades moderno basado en plantillas de la comunidad. El estándar actual. *(En BlackArch.)*
- **`amass` / `subfinder`** — enumeración masiva de subdominios. *(En BlackArch.)*
- **`recon-ng`** — framework de OSINT con estructura tipo Metasploit.
- **`Shodan`** — buscador de dispositivos conectados a internet (cámaras, servidores, IoT). "El Google de los hackers". Web: shodan.io.
- **`Maltego`** — herramienta gráfica para mapear relaciones entre personas, dominios, emails (con grafo visual de conexiones).

## Análisis de tráfico y red
- **`Wireshark`** — el analizador de paquetes con interfaz gráfica. El estándar mundial. Es tcpdump pero visual y con superpoderes. **Apréndelo sí o sí.**
- **`bettercap`** — la navaja suiza para ataques **Man-in-the-Middle** (MITM) en redes. *(En Parrot.)*
- **`Responder`** — captura credenciales en redes Windows envenenando protocolos (LLMNR/NBT-NS). Clave en pentesting interno. *(En BlackArch.)*

## Pentesting web
- **`Burp Suite`** — LA herramienta para hacking web. Un proxy que intercepta y modifica todo el tráfico entre tu navegador y la web. Imprescindible para pentesting web profesional. Hay versión Community gratis.
- **`OWASP ZAP`** — la alternativa gratuita y open source a Burp. Excelente para empezar.

## Active Directory / Windows (redes empresariales)
- **`BloodHound`** — mapea gráficamente las rutas de ataque en un dominio Windows/Active Directory. Revela cómo llegar de un usuario normal a administrador. Revolucionó el pentesting empresarial.
- **`CrackMapExec` / `nxc`** — navaja suiza para atacar redes Windows (SMB, WinRM, LDAP). *(En BlackArch.)*
- **`evil-winrm` / `impacket`** — shells y herramientas de protocolos Windows. *(En BlackArch.)*
- **`mimikatz`** — extrae contraseñas y tickets de la memoria de Windows. Legendario. (Se corre en el Windows objetivo.)

## Forense y respuesta
- **`Autopsy`** — interfaz gráfica para forense de discos. Recupera archivos borrados, analiza líneas de tiempo.
- **`Wazuh`** — plataforma SIEM/XDR open source: junta logs de muchas máquinas, detecta amenazas y alerta.
- **`Velociraptor`** — caza de amenazas y respuesta a incidentes en endpoints.

## Esteganografía y CTF
- **`steghide` / `stegseek`** — esconden y extraen datos dentro de imágenes/audio. Muy usados en CTFs.
- **`CyberChef`** — "la navaja suiza del cifrado" en web: decodifica, descifra y transforma datos. App de Google. **Tenla en favoritos.**

---

# 🦹 PARTE 3 — CÓMO ATACAN LOS HACKERS NO ÉTICOS

> *Para defenderte de algo, primero tienes que entender cómo funciona.* Esta parte explica las **técnicas y tecnologías** de los criminales — no para que las uses, sino para que las **reconozcas y las pares**.

## 3.1 — Ingeniería social: el hackeo del humano
La técnica #1 del mundo real. **El humano es el eslabón más débil**, siempre. No importa qué tan buena sea tu seguridad técnica si alguien te convence de darle tu contraseña.

- **Phishing:** emails/mensajes falsos que imitan a un banco, empresa o amigo para robar credenciales o instalar malware. (MITRE T1566.)
- **Spear phishing:** phishing dirigido y personalizado a una persona específica (usando OSINT previo).
- **Vishing / Smishing:** lo mismo por llamada de voz o SMS.
- **Pretexting:** inventar una historia creíble ("soy de soporte técnico") para ganar confianza.
- **Baiting:** dejar un USB infectado "olvidado" para que alguien lo conecte por curiosidad.

**Cómo lo reconoces:** urgencia artificial ("¡tu cuenta será bloqueada YA!"), remitentes raros, enlaces que no coinciden con el texto, errores de ortografía, peticiones de datos sensibles. **Ante la duda, no hagas clic y verifica por otro canal.**

## 3.2 — Tipos de malware
- **Virus:** se pega a un archivo y se replica cuando lo ejecutas.
- **Gusano (worm):** se propaga solo por la red, sin que hagas nada (ej. WannaCry).
- **Troyano:** se disfraza de programa legítimo. Tú lo instalas creyendo que es otra cosa.
- **RAT (Remote Access Trojan):** un troyano que le da **control remoto total** al atacante: archivos, webcam, micrófono, teclado.
- **Keylogger:** registra cada tecla que pulsas (incluidas contraseñas).
- **Ransomware:** cifra todos tus archivos y exige rescate en cripto. La amenaza más cara del mundo hoy.
- **Rootkit:** se esconde en lo más profundo del sistema para que ni el antivirus lo vea (por eso existe rkhunter).
- **Spyware / Stalkerware:** espía tu actividad en secreto.
- **Botnet:** una red de miles de máquinas infectadas ("zombies") controladas a la vez para ataques masivos.

## 3.3 — Command & Control (C2)
Una vez que el malware está dentro, necesita "llamar a casa" para recibir órdenes. Ese canal es el **C2 (Command and Control)**. Frameworks como **Cobalt Strike** (legítimo, pero muy usado por criminales), **Sliver** o **Mythic** gestionan ejércitos de máquinas infectadas.

**Cómo se esconde el C2:**
- **Domain fronting / DNS tunneling:** esconde el tráfico de control dentro de tráfico que parece normal (consultas DNS, peticiones HTTPS a sitios famosos).
- **Beaconing:** el malware "llama a casa" cada X tiempo de forma intermitente para no levantar sospechas.

**Cómo se detecta:** tráfico periódico y regular a una IP/dominio raro. Aquí brilla **Suricata** (un IDS) y el análisis de logs.

## 3.4 — Lo que hace el atacante una vez dentro
1. **Escalada de privilegios:** de usuario normal a administrador/root (aprovechando bugs o malas configuraciones).
2. **Persistencia:** se asegura de seguir dentro aunque reinicies (tareas programadas, servicios, claves de registro). (MITRE T1547.)
3. **Movimiento lateral:** salta de una máquina a otra dentro de la red, buscando el objetivo valioso. (Aquí usan BloodHound, CrackMapExec.)
4. **Evasión de defensas:** desactiva el antivirus, borra logs, ofusca su malware para no ser detectado. (MITRE T1562.)
5. **Exfiltración:** roba los datos, normalmente cifrados y poco a poco para no ser notado.
6. **Anti-forense:** borra sus huellas — limpia logs, cambia marcas de tiempo, usa cifrado.

## 3.5 — Ataques de red comunes
- **MITM (Man-in-the-Middle):** el atacante se mete en medio de tu comunicación y la lee/modifica (con bettercap, por ejemplo). Por eso importa HTTPS.
- **ARP Spoofing:** engaña a tu red local para que el tráfico pase por la máquina del atacante.
- **DNS Spoofing:** te manda a una web falsa aunque escribas la dirección correcta.
- **DoS / DDoS:** saturan un servicio con tráfico hasta tumbarlo (uno o miles de orígenes).
- **Evil Twin:** un punto WiFi falso que imita al real para que te conectes y robarte datos.

## 3.6 — La mente del atacante: cómo piensan y dónde estudiarlos (legalmente)

Pediste ver "con tus propios ojos" cómo razona un atacante. Es una de las mejores cosas que puedes estudiar — **pero hay una forma correcta y una forma que te puede arruinar la vida.** Lee esto con cuidado.

### Cómo PIENSA realmente un atacante (la mentalidad)
Un atacante no piensa en "herramientas", piensa en **preguntas**. Esta es su lógica mental, paso a paso:

1. *"¿Qué hay aquí?"* — Mapea todo. Nunca asume. Lista cada puerto, cada servicio, cada subdominio, cada empleado. (Reconocimiento obsesivo.)
2. *"¿Qué está mal configurado o desactualizado?"* — Busca el eslabón débil, no el fuerte. No ataca la puerta blindada; busca la ventana abierta.
3. *"¿Cuál es el camino de menor resistencia?"* — Prefiere engañar a un humano (phishing) antes que romper cifrado. La pereza es eficiencia para él.
4. *"Si fuera el dueño, ¿qué error habría cometido yo?"* — Piensa como el defensor para encontrar sus descuidos. Contraseñas reutilizadas, paneles de admin olvidados, backups públicos.
5. *"¿Cómo me quedo sin que me vean?"* — Persistencia y sigilo. Entrar es fácil; quedarse sin ser detectado es el arte.
6. *"¿Cómo borro mis huellas?"* — Siempre piensa en la salida y en no dejar rastro.

**La clave de la mentalidad:** un atacante ve sistemas como **rompecabezas con reglas que se pueden doblar**, no como cosas que "deben" usarse de cierta forma. Un formulario de login no es para iniciar sesión: es una entrada de datos que quizás no valida bien. Esa forma de mirar el mundo —"¿qué pasa si lo uso al revés de como se supone?"— es el corazón del hacking.

### Dónde estudiar esto LEGALMENTE (comunidades de hackers éticos)
Aquí los hackers comparten cómo piensan, en abierto y sin nada ilegal:

- **Write-ups de CTF y de HackTheBox** — son el oro puro. Un hacker documenta paso a paso CÓMO resolvió una máquina y, más importante, **POR QUÉ pensó cada cosa**. Lees su razonamiento en vivo. Busca "HackTheBox writeup" o en **ctftime.org**.
- **0x00sec.org** — foro de hacking técnico y ético, comunidad seria que discute malware, exploits y técnicas con enfoque de aprendizaje.
- **Reddit:** r/netsec, r/hacking, r/AskNetsec, r/blueteamsec — discusiones diarias de profesionales.
- **YouTube (ver a un hacker pensar en vivo):**
  - **LiveOverflow** — explica la mentalidad y el "por qué" detrás de los exploits.
  - **IppSec** — resuelve máquinas de HackTheBox narrando cada pensamiento.
  - **John Hammond** — análisis de malware y CTFs.
  - **NahamSec** — bug bounty y hacking web real.
- **Conferencias DEF CON y Black Hat** — miles de charlas gratis en YouTube donde los mejores del mundo explican sus técnicas. Busca "DEF CON" + un tema.
- **Discords:** los de TryHackMe, HackTheBox y bug bounty — gente aprendiendo y compartiendo todo el día.

### Cómo piensan los CRIMINALES de verdad (sin meterte donde no debes) — Threat Intelligence
Para ver la mente de los atacantes **reales** (grupos de ransomware, APTs, estafadores), no entras a sus foros: **lees los reportes de quienes los cazan.** Las empresas de seguridad publican análisis detallados de operaciones criminales reales:

- **The DFIR Report** (thedfirreport.com) — reconstruyen ataques reales hora por hora. *"El atacante entró a las 14:00, a las 14:20 hizo esto..."* Es como leer la mente del criminal con todo el contexto.
- **Mandiant, CrowdStrike, Cisco Talos, Unit 42 (Palo Alto)** — sus blogs analizan grupos criminales reales.
- **Krebs on Security** (krebsonsecurity.com) — un periodista que investiga el cibercrimen por dentro.
- **MITRE ATT&CK** — cataloga las técnicas exactas de cada grupo criminal conocido.
- **Malware Bazaar / abuse.ch** — muestras reales de malware para estudiar (con MUCHO cuidado, en máquina aislada como REMnux).

### ⚠️ Los foros criminales reales — por qué NO debes entrar
Existen foros y mercados (en la web normal y en la dark web) donde se vende malware, datos robados, accesos hackeados y exploits. **Sé que da curiosidad. No entres. Esto es en serio:**

1. **Es ilegal navegar y participar.** Solo registrarte o comprar algo te convierte en cómplice de un delito. Hay menores que han terminado con cargos por esto.
2. **Están vigilados por la policía.** El FBI, la Europol y la Guardia Nacional de México infiltran y tumban estos foros constantemente (RaidForums, BreachForums, Genesis Market — todos cayeron, y arrestaron a sus usuarios, incluidos adolescentes).
3. **Te van a estafar o infectar.** Ese mundo se basa en engañarse entre ellos. El 90% de lo que se vende ahí es estafa o malware diseñado para hackearte a TI.
4. **Te cierra el futuro.** Las empresas de seguridad (donde querrás trabajar) revisan tu rastro digital. Un historial en foros criminales te cierra la carrera que estás construyendo. Para siempre.

**La verdad importante:** no necesitas entrar a un foro criminal para aprender a defenderte de los criminales — igual que un médico no necesita contagiarse para curar. Todo el conocimiento real, profundo y útil está en los recursos **legales** de arriba. Los criminales no son más listos por estar en esos foros; son más **tontos por arriesgar su libertad.** Tú juega el juego largo.

---

# 🚨 PARTE 4 — CÓMO RESPONDER A UN ATAQUE (Blue Team)

> Te están atacando, o crees que sí. ¿Qué haces? Esto es **Respuesta a Incidentes (Incident Response)**. Hay un método profesional, no improvisar.

## 4.1 — El proceso de respuesta a incidentes (modelo NIST)
Los profesionales siguen **6 fases**. Memorízalas:

1. **Preparación:** ANTES del ataque. Tener backups, logs activos, un plan, herramientas listas. (Aquí entra todo tu hardening con `lynis`.)
2. **Identificación / Detección:** ¿realmente hay un incidente? ¿Qué tan grave? Confirmar con evidencia.
3. **Contención:** detener la hemorragia. Aislar la máquina afectada (desconectarla de la red), sin apagarla todavía (perderías la RAM, que es evidencia).
4. **Erradicación:** eliminar la causa — quitar el malware, cerrar el agujero por el que entró.
5. **Recuperación:** restaurar desde backups limpios, devolver los sistemas a producción, vigilando que no vuelva.
6. **Lecciones aprendidas:** ¿cómo entró? ¿cómo evitarlo? Documentar todo. Es la fase que la gente se salta y por la que los vuelven a hackear.

## 4.2 — Tu kit de respuesta
Si sospechas que tu máquina fue comprometida, este es el flujo:

```bash
# 1. ¿Conexiones de red sospechosas activas? (¿C2?)
ss -tunap                 # conexiones actuales con su proceso
sudo tcpdump -i wlan0 -n  # ver el tráfico crudo en vivo

# 2. ¿Procesos raros corriendo?
ps aux --sort=-%cpu | head
top

# 3. ¿Rootkits o backdoors escondidos?
sudo rkhunter --check
sudo chkrootkit            # (si lo instalas)

# 4. ¿Malware en archivos?
clamscan -r --infected /home

# 5. Auditar el estado de seguridad del sistema
sudo lynis audit system

# 6. ¿Quién inició sesión y cuándo?
last            # historial de logins
lastb           # intentos fallidos de login
who             # sesiones activas ahora

# 7. Revisar los logs del sistema en busca de huellas
sudo journalctl -p err -S today          # errores de hoy
sudo grep -i "failed\|invalid" /var/log/auth.log   # logins fallidos
```

## 4.3 — Análisis forense (si quieres entender QUÉ pasó)
- **RAM:** captura la memoria y analízala con **`volatility3`** (`vol`) — verás procesos ocultos y conexiones que ya no están en el disco.
- **Detección por patrones:** escribe o descarga reglas **`yara`** para identificar la familia de malware.
- **Red:** monta **`suricata`** como IDS para que vigile en tiempo real y alerte de futuros ataques.

## 4.4 — Defensas que debes montar
- **`fail2ban`** — bloquea automáticamente IPs que fallan muchos logins (anti-fuerza bruta). **Instálalo ya**, es de lo más útil.
- **Firewall** — `firewalld` o `ufw`. Cierra todo lo que no necesites.
- **`suricata`** — tu IDS de red.
- **`Wazuh`** — un SIEM central que junta y correlaciona todo.
- **2FA en todo** — la defensa más rentable contra robo de credenciales.
- **Backups 3-2-1** — 3 copias, 2 medios distintos, 1 fuera de sitio. La ÚNICA defensa real contra ransomware.

---

# 🕵️ PARTE 5 — RASTREAR AL ATACANTE DE VUELTA (Atribución)

> Quieres saber **quién** te atacó y desde dónde. Esto se llama **atribución** y **threat intelligence**. 

## ⚠️ La regla legal antes que nada
**NO puedes "hackear de vuelta" (hack back).** Acceder a la máquina del atacante, aunque te haya atacado primero, es un delito en México, EE.UU. y casi todo el mundo. Lo que SÍ puedes y debes hacer:
- **Rastrear con información pública** (OSINT, logs, threat intel).
- **Documentar** todo con evidencia.
- **Reportar** a las autoridades (en México: la Guardia Nacional / Policía Cibernética) y al proveedor del atacante.

La atribución es **difícil** incluso para profesionales: los atacantes serios usan Tor, VPNs, máquinas intermedias hackeadas y servidores en otros países. Pero los atacantes torpes (la mayoría) dejan rastros.

## 5.1 — Paso 1: Reúne la evidencia del ataque
Antes de rastrear, necesitas datos del atacante. De tus logs sacas:
- **La(s) IP(s) de origen** del ataque (de `auth.log`, de Suricata, de tcpdump).
- **Las marcas de tiempo** exactas.
- **El método** usado (qué puerto, qué exploit, qué user-agent).
- **Cualquier archivo** que dejaron (malware, scripts).

```bash
# IPs que más intentaron entrar por SSH
sudo grep "Failed password" /var/log/auth.log | grep -oE "[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+" | sort | uniq -c | sort -rn
```

## 5.2 — Paso 2: Investiga la IP del atacante (OSINT — legal)
Con la IP, usa **tus propias herramientas** y servicios públicos:

```bash
# ¿De dónde es la IP? ¿De quién es?
whois 1.2.3.4                    # dueño del rango, proveedor (ISP), país
dig -x 1.2.3.4                   # nombre DNS inverso (PTR)
nmap -sV 1.2.3.4                 # ¿qué corre esa IP? (con cuidado — solo info)
```

Y servicios web públicos de threat intelligence:
- **AbuseIPDB** (abuseipdb.com) — ¿esa IP ya está reportada por atacar a otros? Casi siempre sí.
- **VirusTotal** — reputación de IPs, dominios y archivos.
- **Shodan** (shodan.io) — ¿qué servicios expone esa IP? ¿Es un servidor hackeado intermedio?
- **ip-api.com / ipinfo.io** — geolocalización aproximada y ISP.
- **GreyNoise** — ¿es un escáner masivo de internet o un ataque dirigido a ti?

## 5.3 — Paso 3: Analiza lo que dejaron
Si dejaron malware o scripts, ahí hay pistas de oro:
- **`exiftool`** sobre cualquier archivo → metadatos, idioma, software usado.
- **`Ghidra` / `radare2`** sobre el malware → strings, dominios C2 embebidos, idioma del código, rutas del desarrollador (a veces dejan `C:\Users\SuNombre\...`).
- **`yara`** → identifica de qué familia/grupo es el malware (eso ya es atribución parcial).
- Busca los **IOCs (Indicadores de Compromiso)**: IPs, dominios, hashes de archivos. Búscalos en VirusTotal y bases de threat intel — quizás ya están ligados a un grupo conocido.

## 5.4 — Paso 4: Cuenta la historia (correlación)
Junta todo: IP + geolocalización + horario del ataque + tipo de malware + idioma + objetivos. Con eso construyes un **perfil del atacante**. Esto es exactamente lo que hace un framework OSINT con grafo de relaciones, timeline y correlación. Para eso están hechos.

## 5.5 — Honeypots: que el atacante venga a ti
La técnica más elegante de atribución: poner una **trampa**.
- **Qué es un honeypot:** un sistema falso y vulnerable a propósito, diseñado para que los atacantes lo ataquen. No tiene nada de valor — su único trabajo es **registrar TODO** lo que el atacante hace.
- **Para qué sirve:** estudias las técnicas del atacante en vivo, capturas su malware y sus IPs, y aprendes muchísimo, sin riesgo real.
- **Herramientas:** `Cowrie` (honeypot SSH/Telnet), `T-Pot` (suite completa de honeypots con dashboards), `Canarytokens` (archivos-trampa que te avisan si alguien los abre).
- **Proyecto barato:** un Raspberry Pi corriendo Cowrie es un honeypot perfecto para aprender, por muy poco dinero.

## 5.6 — Paso 5: Reporta (lo correcto y legal)
- **Al ISP del atacante:** el `whois` te da el contacto de abuso (`abuse@proveedor.com`). Repórtalo con tus logs.
- **A AbuseIPDB:** reporta la IP para proteger a otros.
- **A las autoridades:** si el ataque fue serio, en México es la **Guardia Nacional (Policía Cibernética)**: `cert-mx` / `policiacibernetica@gn.gob.mx`.
- **Guarda toda la evidencia** intacta (logs, archivos, capturas) por si hay proceso legal.

---

# 🎓 CIERRE — Tu ruta de aprendizaje

Para construir tu camino en seguridad, sigue este orden:

1. **Domina los fundamentos:** redes (TCP/IP, DNS, HTTP), Linux y un lenguaje (ya tienes Python). Sin esto, las herramientas son magia que no entiendes.
2. **Practica en entornos legales y gratis:**
   - **TryHackMe** — el mejor para empezar, guiado paso a paso.
   - **HackTheBox** — más difícil, máquinas reales para hackear (legalmente).
   - **OverTheWire** — wargames de terminal, geniales para Linux.
   - **PortSwigger Web Security Academy** — gratis y el mejor para hacking web.
   - **DVWA / Metasploitable / VulnHub** — máquinas vulnerables para tu propio lab.
3. **Aprende los dos lados:** no seas solo Red Team. Los mejores entienden ataque Y defensa.
4. **Certificaciones (para el futuro):** CompTIA Security+ (base), luego eJPT, PNPT u OSCP (las que valen oro en pentesting).
5. **Lee y mantente al día:** MITRE ATT&CK, los reportes de threat intel, y r/netsec.

## La frase que resume todo
> "Con un gran poder viene una gran responsabilidad."

Estás aprendiendo a una edad en la que muy pocos lo hacen. Eso es un poder real. Úsalo para **construir y proteger**, no para destruir. Los mejores hackers del mundo — los que ganan más, los respetados — son los que eligieron el lado correcto. Sé uno de ellos.

---

## 📚 Otros libros de la academia
```
libro-primer-dia.md           — empieza aquí si vienes de cero
libro-linux-desde-cero.md     — Linux para quien nunca tocó una terminal
libro-programador-a-hacker.md — si ya programas, tu atajo
libro-redes.md                — fundamentos de redes
libro-linux.md                — Linux para seguridad
libro-python-hacking.md       — crea tus propias herramientas
libro-web-hacking.md          — hacking web con payloads reales
libro-osint.md                — investigación con fuentes abiertas
libro-criptografia.md         — cifrado y cómo se rompe
libro-forense.md              — investigación post-ataque
libro-malware.md              — análisis de código malicioso
```

> ⬛ *Aprende, protege, y mantente del lado correcto.*
