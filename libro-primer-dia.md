# 🌅 TU PRIMER DÍA EN CIBERSEGURIDAD
> El libro que me hubiera gustado tener cuando empecé. Para el que llega desde cero.
> Si nunca has hecho esto, empieza aquí. Versión 2.0

---

## 👋 Bienvenido

Si estás leyendo esto, acabas de tomar una de las mejores decisiones de tu vida. La ciberseguridad es uno de los campos más fascinantes, mejor pagados y con más futuro que existen. Y lo mejor: **puedes aprenderlo solo, gratis, desde tu casa.**

Este libro no asume que sabes NADA. Ni de Linux, ni de redes, ni de hacking. Solo asume que tienes curiosidad y ganas. Vamos paso a paso.

---

## 🎬 Primero: olvida lo que viste en las películas

El hacker de las películas teclea rapidísimo, ve pantallas verdes y entra a la NASA en 10 segundos. **Eso no existe.**

La realidad es más tranquila y más poderosa:
- La ciberseguridad es **90% investigar, leer y pensar**, 10% teclear.
- No "entras" a sistemas mágicamente — encuentras errores que alguien dejó.
- Es más parecido a ser **detective** que a ser un mago.
- La mayoría del tiempo estás aprendiendo cómo funciona algo para encontrarle el fallo.

Si te gusta resolver rompecabezas, entender cómo funcionan las cosas, y la sensación de "¡ajá, ya entendí!" — este mundo es para ti.

---

## 🗺️ Los caminos de la ciberseguridad

La "ciberseguridad" no es una sola cosa. Es un mundo con muchas especialidades. Estas son las principales para que sepas qué hay:

- **🔴 Red Team (ofensivo):** simulas ser el atacante para encontrar fallos antes que los malos. "Pentester" (probador de penetración).
- **🔵 Blue Team (defensivo):** proteges, detectas y respondes a ataques. Vigilas que nadie entre.
- **🔍 OSINT:** investigación con información pública. Encuentras todo sobre un objetivo usando solo lo que está al alcance de todos.
- **🦠 Análisis de malware:** diseccionas virus para entender cómo funcionan y detenerlos.
- **🔬 Forense digital:** eres el CSI digital — investigas qué pasó después de un ataque.
- **🌐 Seguridad web:** encuentras fallos en páginas y aplicaciones.
- **📡 Seguridad de redes / WiFi:** proteges (o auditas) redes.

**No tienes que elegir ahora.** Prueba un poco de todo y verás qué te emociona más. La mayoría empieza explorando y luego se especializa.

---

## ⚖️ LA REGLA MÁS IMPORTANTE (léela dos veces)

Antes de aprender CUALQUIER cosa, graba esto en tu mente:

> **Solo practicas en lo que es TUYO o en lo que tienes PERMISO de practicar.**

Esto no es opcional. Es la diferencia entre un profesional y un criminal:

- ✅ **Legal:** tu propia computadora, máquinas hechas para practicar (TryHackMe, HackTheBox, DVWA), un cliente que te contrató por escrito.
- ❌ **Delito:** la red de tu escuela, la web de una empresa, la cuenta de alguien — sin permiso. **Aunque "solo estés probando".**

En México (Código Penal) y EE.UU. (CFAA) hay **cárcel real** por acceder a sistemas sin permiso. Hay adolescentes con antecedentes por "solo curiosear".

**La buena noticia:** existen lugares LEGALES y GRATIS hechos para que practiques todo lo que quieras (te los muestro abajo). No necesitas atacar nada real para aprender. Nunca.

El conocimiento no es el delito. El **uso sin permiso**, sí. Sé de los buenos — ganan más, duermen tranquilos, y construyen carreras.

---

## 🧰 Qué necesitas para empezar (spoiler: casi nada)

No necesitas una computadora cara ni equipo especial. Necesitas:

1. **Una computadora** (la que tengas).
2. **Internet.**
3. **Curiosidad y paciencia** — esto es lo único que de verdad importa.

Eso es todo. Las herramientas son gratis. El conocimiento es gratis. Solo tienes que ponerle ganas.

---

## 🛠️ Monta tu primer laboratorio (seguro y legal)

Un "lab" es tu campo de práctica privado. Aquí puedes hacer lo que quieras sin meterte en problemas. Hay dos formas fáciles:

**Opción A — Linux dentro de tu Windows (recomendado para empezar):**
- **WSL** (Windows Subsystem for Linux) — instala Linux dentro de Windows con un comando. Sin borrar nada.
- O **VirtualBox** — una "computadora dentro de tu computadora" donde instalas Linux.

**Opción B — Plataformas online (cero instalación):**
- **TryHackMe** (tryhackme.com) — **el mejor lugar para empezar.** Te da máquinas para hackear LEGALMENTE, con lecciones guiadas paso a paso. Empieza aquí HOY.
- **HackTheBox** — más difícil, para cuando avances.

**Tu primera misión:** crea una cuenta gratis en **TryHackMe** y haz su ruta de principiante. Vas a hackear tu primera máquina (legal) en tu primer día. Esa sensación engancha. 😎

---

## 📚 Por dónde aprender (tu ruta)

El orden importa. No empieces por lo avanzado o te frustras. Esta es la ruta que funciona:

1. **Fundamentos primero** — cómo funcionan las **redes** y **Linux**. Aburrido al inicio, pero es la base de TODO. (Sin esto, las herramientas son magia que no entiendes.)
2. **Practica en TryHackMe** desde el día 1 — aprender haciendo.
3. **Elige un camino** que te emocione (web, OSINT, etc.) y profundiza.
4. **Un lenguaje de programación** (Python es ideal) — te deja crear tus propias herramientas.
5. **Repite, falla, aprende.** Nadie nace sabiendo. Cada error te enseña.

---

## 🚫 Errores de novato (evítalos)

- **Querer correr antes de caminar.** No intentes "hackear WiFi" el día 1. Aprende las bases.
- **Coleccionar herramientas sin entenderlas.** Tener Kali no te hace hacker. Entender QUÉ hace cada herramienta, sí.
- **Rendirte cuando algo no funciona.** El 90% de esto es resolver por qué algo falla. Eso ES el trabajo.
- **Practicar en sistemas ajenos "de prueba".** Ya lo dije, pero por su importancia: NO. Usa los labs legales.
- **Comparar tu día 1 con el año 5 de otro.** Todos empezamos sin saber nada.

---

## 🧠 La mentalidad del hacker

Lo que de verdad te hace bueno en esto no es memorizar comandos. Es una forma de **pensar**:

- **Curiosidad sin fin:** "¿cómo funciona esto por dentro?"
- **Pensar al revés:** "¿qué pasa si uso esto de una forma que no estaba prevista?"
- **No rendirse:** cuando algo no funciona, eso es el reto, no el final.
- **Paciencia:** las respuestas llegan a quien persiste.
- **Ética:** el poder se usa para proteger y construir, no para dañar.

Si tienes esa curiosidad — esa necesidad de entender cómo funcionan las cosas — ya tienes lo más importante. El resto se aprende.

---

## 🆓 Recursos gratis para tu camino

No necesitas pagar nada para aprender esto. Los mejores recursos son gratis:

**Plataformas para practicar (legales):**
- **TryHackMe** — rutas guiadas, ideal para empezar. Tiene un "Complete Beginner Path" gratis.
- **HackTheBox** — más difícil, para cuando avances. Su "Starting Point" es gratis.
- **picoCTF** — retos tipo juego, perfectos para principiantes.
- **OverTheWire (Bandit)** — domina la terminal de Linux jugando.

**Aprender teoría (gratis):**
- **Cisco Networking Academy** — cursos de redes gratis con certificado.
- **professormesser.com** — vídeos gratis de CompTIA (Network+, Security+).
- **YouTube:** NetworkChuck, John Hammond, LiveOverflow, IppSec (walkthroughs de HackTheBox).

**Certificaciones (el camino profesional):**
- **Gratis:** las insignias de TryHackMe, Google Cybersecurity Certificate (becas), Cisco.
- **De pago (más adelante):** CompTIA Security+ (la puerta de entrada al empleo), eJPT (pentesting básico), luego OSCP (la más respetada en ofensivo).
- No necesitas certificaciones para aprender — pero ayudan a conseguir el primer empleo.

**Comunidades (no estás solo):**
- Reddit: r/cybersecurity, r/hacking, r/netsec.
- Discords de TryHackMe y HackTheBox.

---

## 🎓 Tu plan para esta semana

1. **Hoy:** crea tu cuenta en **TryHackMe** y haz la primera lección.
2. **Esta semana:** empieza a leer sobre **redes** y **Linux** (los libros `libro-redes.md` y `libro-linux-desde-cero.md` son tu siguiente parada).
3. **Practica un poco cada día.** 30 minutos diarios > 5 horas una vez.
4. **No te rindas en la primera frustración.** Es parte del juego.

---

> ⬛ *Bienvenido al mundo. Todos empezamos exactamente donde estás tú ahora: sin saber nada, pero con ganas. Eso es suficiente para empezar. Lo demás se construye un día a la vez.*
>
> **Siguiente libro:** `libro-linux-desde-cero.md` — porque todo lo que viene corre sobre Linux.
