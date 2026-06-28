# 🐧 LINUX DESDE CERO PARA HACKERS
> Si vienes de Windows y nunca tocaste una terminal, este libro es para ti.
> De "¿qué es esto?" a moverte con confianza. Versión 2.0

---

## 🤔 Espera, ¿por qué Linux?

Vienes de Windows y te preguntas por qué todo el mundo de la seguridad usa Linux. Razones reales:

- **Es transparente:** puedes ver y controlar TODO. Nada está escondido.
- **Es el sistema de los servidores:** la mayoría de lo que vas a auditar/proteger corre Linux.
- **Las herramientas viven aquí:** casi todas las herramientas de seguridad nacieron en Linux.
- **La terminal es poder:** lo que en Windows necesita 10 clics, aquí es un comando que puedes automatizar.
- **Es gratis y libre.**

No tienes que abandonar Windows. Puedes tener Linux **dentro** de Windows (con WSL) o en una máquina virtual. Lo mejor de los dos mundos.

---

## ⌨️ La terminal: tu nuevo superpoder

Lo que más asusta al principio es la **terminal** (esa pantalla negra con texto). Pero piénsalo así:

> En Windows le dices a la computadora qué hacer con **clics**. En Linux se lo dices con **palabras**. Y las palabras son más precisas, más rápidas, y se pueden **automatizar**.

La terminal no es difícil. Es **honesta**: hace exactamente lo que le dices. Una vez que le pierdes el miedo, no vas a querer volver a los clics.

Cuando abres la terminal, ves algo así:
```
usuario@maquina:~$
```
Eso se llama "el prompt". El `$` significa "estoy listo, dame una orden". El `~` significa que estás en tu carpeta personal (tu "home").

---

## 🚶 Tus primeros pasos (comandos de navegación)

Empecemos a movernos. Escribe estos y presiona Enter:

```bash
pwd          # "print working directory" → ¿dónde estoy ahora?
ls           # "list" → ¿qué hay en esta carpeta?
ls -la       # lista TODO, hasta los archivos ocultos, con detalles
cd carpeta   # "change directory" → entra a una carpeta
cd ..        # sube un nivel (a la carpeta de arriba)
cd ~         # vuelve a tu carpeta personal (home)
clear        # limpia la pantalla
```

**Practica ahora:** abre tu terminal, escribe `pwd`, luego `ls`, luego entra a alguna carpeta con `cd`. Muévete. Pierde el miedo. No vas a romper nada solo mirando.

---

## 📂 Cómo está organizado Linux

En Windows tienes `C:\`. En Linux todo cuelga de una sola raíz: `/`. Es como un árbol:

```
/           → la raíz, el origen de todo
├── /home   → las carpetas de los usuarios (la tuya: /home/tunombre)
├── /etc    → la configuración del sistema (importante en seguridad)
├── /var    → datos que cambian, como los LOGS (/var/log) ← clave en forense
├── /bin    → los programas/comandos
├── /tmp    → archivos temporales
└── /root   → la carpeta del superusuario (el jefe del sistema)
```

No te lo aprendas de memoria ahora. Solo entiende que **todo es una sola estructura de carpetas**, y que `/home/tunombre` es donde vives tú.

---

## 📄 Trabajar con archivos

```bash
cat archivo.txt      # muestra el contenido de un archivo
less archivo.txt     # lo muestra paginado (sale con la tecla q)
nano archivo.txt     # abre un editor sencillo para escribir (Ctrl+X para salir)
touch nuevo.txt      # crea un archivo vacío
mkdir micarpeta      # crea una carpeta
cp archivo destino   # copia
mv archivo destino   # mueve o renombra
rm archivo           # borra (¡CUIDADO, no hay papelera!)
```

⚠️ **Ojo con `rm`:** en Linux no hay papelera de reciclaje. Lo que borras, se va para siempre. Y **NUNCA** escribas `rm -rf /` (eso borra todo el sistema — es una broma cruel que circula, no caigas).

---

## 🔍 Buscar cosas (súper útil)

```bash
find / -name "archivo.txt" 2>/dev/null   # busca un archivo por nombre
grep "palabra" archivo.txt               # busca una palabra DENTRO de un archivo
which nmap                               # ¿dónde está instalado un programa?
history                                  # ver los comandos que ya escribiste
```

El `grep` es de tus mejores amigos — busca texto dentro de archivos. Lo vas a usar todo el tiempo (en logs, en resultados, en todo).

---

## 🔐 Permisos (el concepto clave de Linux)

En Linux, cada archivo tiene **permisos**: quién puede leerlo, escribirlo o ejecutarlo. Cuando haces `ls -l` ves algo como:

```
-rwxr-xr--  usuario  archivo
```

No te abrumes. La idea simple:
- **r** = leer (read)
- **w** = escribir (write)
- **x** = ejecutar (execute)

Y para hacer un archivo ejecutable (por ejemplo, un script):
```bash
chmod +x script.sh    # ahora se puede ejecutar
./script.sh           # así lo corres
```

Entenderás los permisos a fondo con la práctica. Por ahora, quédate con: **cada archivo tiene reglas de quién puede hacer qué.**

---

## 👑 sudo: pedir permiso de jefe

Algunas cosas necesitan permisos de administrador (el "root", el jefe del sistema). Para eso usas `sudo`:

```bash
sudo comando      # ejecuta ese comando como administrador (te pedirá tu contraseña)
```

**Regla de oro:** no uses `sudo` para todo a lo loco. Úsalo solo cuando de verdad lo necesites. Es poderoso, y con poder viene responsabilidad (y la posibilidad de romper algo).

---

## 📦 Instalar programas

En Windows descargas un `.exe`. En Linux instalas desde la terminal, fácil y seguro:

```bash
# En Kali / Ubuntu / Debian (usan apt):
sudo apt update              # actualiza la lista de programas
sudo apt install nombre      # instala un programa

# Ejemplo: instalar nmap
sudo apt install nmap
```

Esto descarga e instala el programa de forma segura, sin virus ni instaladores raros. Es una de las cosas más cómodas de Linux.

---

## 🛠️ Comandos esenciales para seguridad

Ya que vas a hacer ciberseguridad, estos te van a servir desde el inicio:

```bash
ip a                 # ver tu dirección IP y tus tarjetas de red
ping google.com      # probar si tienes conexión
ss -tunap            # ver las conexiones de red activas
ps aux               # ver todos los programas que están corriendo
top                  # monitor de procesos en vivo (sale con q)
nmap -sV localhost   # tu primer escaneo (a tu propia máquina)
```

**Tu primer ejercicio real:** corre `nmap -sV localhost`. Eso escanea TU PROPIA máquina y te dice qué servicios tiene abiertos. Es legal (es tuya), y es tu primer paso de verdad en el reconocimiento.

---

## 🐉 Sobre tu Kali

Si instalaste **Kali Linux**, ¡bien! Kali es una versión de Linux que ya viene con **cientos de herramientas de seguridad** preinstaladas. Es el "Linux de los hackers".

Pero recuerda lo del otro libro: **tener Kali no te hace hacker.** Kali es una caja de herramientas. Este libro te enseña a usar la caja. El conocimiento es lo que vale, no las herramientas.

Por ahora, solo **muévete por Kali, practica los comandos de arriba, y acostúmbrate.** Las herramientas vienen después, cuando entiendas las bases.

---

## 💡 Trucos que te harán la vida fácil

- **Tecla Tab:** autocompleta nombres. Escribe `cd Desc` y presiona Tab → completa `Descargas`. Te ahorra muchísimo.
- **Flechas arriba/abajo:** navega por los comandos que ya escribiste. No vuelvas a teclear todo.
- **Ctrl + C:** detiene un comando que se quedó corriendo.
- **`comando --help`:** casi todo comando te explica cómo usarse con `--help`.
- **`man comando`:** el manual completo de un comando (sale con q).

---

## 🔗 El truco que lo cambia todo: pipes y redirecciones

Esto es lo que hace a Linux tan poderoso. Puedes **conectar comandos** entre sí:

```bash
# El pipe | manda la salida de un comando a otro
ls -la | grep ".txt"          # lista archivos, pero SOLO los .txt
cat archivo.txt | grep "error" # muestra solo las líneas con "error"
ps aux | grep firefox          # busca el proceso de firefox

# Redirecciones: guardar la salida en un archivo
comando > archivo.txt          # guarda (sobrescribe)
comando >> archivo.txt         # guarda (añade al final)
nmap localhost > escaneo.txt   # guarda el resultado del escaneo
```

**Por qué importa:** en seguridad encadenas comandos todo el tiempo. "Escanea → filtra lo interesante → guárdalo". El pipe `|` es tu mejor amigo. Practícalo.

---

## 📋 Tu chuleta (guárdala)

```
NAVEGAR:   pwd · ls -la · cd · cd .. · cd ~ · clear
ARCHIVOS:  cat · less · nano · touch · mkdir · cp · mv · rm
BUSCAR:    find · grep · which · history
SISTEMA:   ip a · ps aux · top · ss -tunap · df -h · free -h
PERMISOS:  chmod +x · sudo · chmod 600
AYUDA:     comando --help · man comando
ATAJOS:    Tab (autocompletar) · ↑↓ (historial) · Ctrl+C (parar)
```

---

## 🎯 Tu plan de práctica

1. **Hoy:** abre la terminal y practica los comandos de navegación (`pwd`, `ls`, `cd`). Solo muévete.
2. **Mañana:** crea archivos y carpetas, muévelos, bórralos (en tu carpeta personal). Juega.
3. **Esta semana:** corre `nmap -sV localhost` y `ip a`. Empieza a sentirte cómodo.
4. **Cada día:** un comando nuevo. En un mes te mueves con confianza.

**El mejor recurso para practicar Linux gratis:** **OverTheWire: Bandit** (overthewire.org/wargames/bandit). Es un juego que te enseña Linux nivel por nivel. Empieza HOY, es adictivo.

---

> ⬛ *La terminal no es difícil, es honesta. Hace exactamente lo que le dices. Domínala y dominas la herramienta más poderosa de la seguridad. Un comando a la vez.*
>
> **Siguiente libro:** `libro-programador-a-hacker.md` — para convertir tu skill de programación en superpoder.
