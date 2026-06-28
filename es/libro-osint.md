# 🔎 LIBRO DE OSINT
> Inteligencia de Fuentes Abiertas. El arte de saberlo todo usando solo información pública.
> Versión 2.0 — BUFFEADO: herramientas reales, dorks, frameworks y técnicas pro.

---

## ¿Qué es OSINT?
**OSINT (Open Source Intelligence)** es obtener información valiosa de **fuentes públicas y legales**: redes sociales, buscadores, registros públicos, metadatos, filtraciones. No se hackea nada — se **investiga**. Es la base de:
- La **fase de reconocimiento** de todo pentest.
- Investigaciones policiales y periodísticas.
- Verificación de identidades y due diligence.
- Conocer tu propia exposición (defensivo).

**La idea central:** la gente y las organizaciones dejan migajas de información por todos lados. Por separado no dicen nada; **juntas, cuentan toda la historia.** OSINT es el arte de juntar esas migajas.

---

## 1. La metodología OSINT (no es buscar al azar)

El OSINT profesional sigue un ciclo:

1. **Definir el objetivo:** ¿qué quiero saber exactamente?
2. **Recolectar:** reunir datos de todas las fuentes posibles.
3. **Procesar:** organizar lo recolectado (grafos, hojas de cálculo).
4. **Analizar:** conectar puntos, encontrar patrones.
5. **Inteligencia:** convertir datos sueltos en conclusiones útiles.

**Pivoting (la técnica clave):** cada dato es un trampolín al siguiente. Un email → username → perfiles → fotos → metadatos → ubicación. Saltas de un dato a otro construyendo el panorama completo. **Siempre busca el siguiente pivote.**

---

## 2. OSINT de personas — herramientas por tipo de dato

### 👤 Username (la mina de oro — la gente reúsa el mismo nick)
```bash
sherlock usuario               # cientos de plataformas
maigret usuario                # 2500+ sitios + extrae datos del perfil
```
- **WhatsMyName** (whatsmyname.app) — versión web, muy completa
- **Namechk / Instant Username** — disponibilidad rápida en muchos sitios

### 📧 Email
```bash
holehe correo@ejemplo.com      # en qué servicios está registrado
```
- **Have I Been Pwned** (haveibeenpwned.com) → ¿en qué filtraciones apareció?
- **Hunter.io** → emails de una empresa + patrón (nombre.apellido@)
- **Epieos** (epieos.com) → herramienta web potente para email y teléfono
- **GHunt** → todo sobre una cuenta de Google
- Patrón corporativo predecible: `nombre.apellido@empresa.com`

### 📱 Teléfono
- **PhoneInfoga** → carrier, tipo de línea, footprint
- **Epieos** → apps vinculadas (¿tiene WhatsApp/Telegram?)
- Búsqueda inversa: pega el número en Google con comillas, Truecaster, etc.

### 🪪 Nombre real
- Google con comillas: `"Nombre Apellido"`, `"Nombre Apellido" ciudad`
- Registros públicos, padrones electorales, LinkedIn
- **Pipl, That'sThem, FastPeopleSearch** (EE.UU.)

### 🌐 Redes sociales (cada una expone algo distinto)
- **LinkedIn** → trabajo, colegas, historial laboral
- **Instagram** → ubicaciones, rutina, círculo cercano
- **Twitter/X** → opiniones, horarios de actividad (zona horaria)
- **Facebook** → familia, amigos, eventos
- **Análisis de fotos:** fondo, ropa, reflejos, matrículas, carteles → contexto y ubicación
- **Análisis de horarios:** cuándo publica → su zona horaria y rutina diaria

### 🖼️ Búsqueda inversa de imágenes
- **Yandex** → el MEJOR para rostros (mejor que Google)
- **Google Images, TinEye, Bing Visual Search**
- **PimEyes** → reconocimiento facial (de pago, potente)
- **Search4Faces** → caras en redes sociales rusas

---

## 3. Google Dorking — buscar como profesional

Operadores que convierten Google en herramienta de hacking pasivo:

```
site:ejemplo.com                    → solo ese dominio
filetype:pdf                        → solo un tipo de archivo
intitle:"index of"                  → directorios abiertos
inurl:admin                         → URLs con "admin"
intext:"password"                   → páginas con esa palabra
site:ejemplo.com -www               → subdominios
cache:ejemplo.com                   → versión guardada por Google
related:ejemplo.com                 → sitios similares
"Nombre Apellido" filetype:pdf      → documentos sobre alguien
```

### Combos potentes (Google Hacking)
```
site:ejemplo.com filetype:pdf "confidencial"
intitle:"index of" "backup"
inurl:"/wp-content/" site:ejemplo.com               → WordPress
filetype:env "DB_PASSWORD"                          → configs filtradas
intitle:"index of" "parent directory" "*.sql"       → bases de datos expuestas
inurl:viewerframe?mode=                             → cámaras IP abiertas
site:pastebin.com "empresa.com"                     → leaks en pastes
site:github.com "empresa.com" password              → secretos en código
```
La **Google Hacking Database (GHDB)** en exploit-db.com tiene **miles** de estos dorks listos. Los mismos operadores funcionan en **Bing, DuckDuckGo, Yandex**.

---

## 4. OSINT de dominios e infraestructura

```bash
whois ejemplo.com                   # quién lo registró, fechas, contacto
dig ejemplo.com ANY                 # registros DNS (A, MX, NS, TXT)
dig -x 1.2.3.4                      # DNS inverso

# Subdominios (cada uno = superficie de ataque)
subfinder -d ejemplo.com
amass enum -d ejemplo.com
# crt.sh (web) — certificados SSL revelan subdominios ocultos

# Recolección masiva (emails, hosts, empleados)
theHarvester -d ejemplo.com -b all
```

### Fuentes web clave
- **crt.sh** → certificados SSL (subdominios ocultos)
- **Shodan.io** → "el Google de los dispositivos": servidores, cámaras, IoT, paneles expuestos. Dorks: `org:"empresa"`, `port:3389`, `webcam`
- **Censys.io** → similar a Shodan, muy técnico
- **Wayback Machine** (web.archive.org) → versiones viejas (info que borraron)
- **DNSDumpster / SecurityTrails** → mapa visual de infraestructura DNS
- **ViewDNS.info** → 30+ herramientas DNS/IP en un sitio
- **BuiltWith / Wappalyzer** → tecnologías que usa un sitio

---

## 5. Imágenes, geolocalización y metadatos

### Metadatos (EXIF) — lo que las fotos esconden
```bash
exiftool foto.jpg
```
Una foto puede revelar: **coordenadas GPS exactas**, modelo de cámara/teléfono, fecha/hora, software de edición, hasta número de serie. Documentos Office/PDF revelan autor, empresa, rutas internas.
> **Defensa:** borra metadatos antes de publicar → `exiftool -all= foto.jpg`

### Geolocalización por la imagen (GEOINT)
Cuando NO hay GPS, deduces la ubicación por el contenido:
- Carteles, idiomas, matrículas, arquitectura, vegetación, postes de luz, enchufes
- **Sombras** → hora y orientación (norte/sur)
- Cruzar con **Google Earth / Street View / Mapillary**
- **SunCalc** → posición del sol según fecha/hora/lugar
- El juego **GeoGuessr** entrena justo esta habilidad

---

## 6. OSINT de filtraciones (breaches)

Cuando hackean una empresa, los datos acaban públicos. Investigar (legalmente):
- **Have I Been Pwned** → ¿tu email/teléfono está filtrado?
- **Dehashed, IntelligenceX, Snusbase** → búsqueda en datos filtrados (de pago)
- **Pastebin, Ghostbin** → a veces se filtran datos ahí
- **Google dork:** `site:pastebin.com "dominio.com"`

**Para defensa:** revisa tus emails en HIBP. Si salen → cambia esas contraseñas YA + activa 2FA.

---

## 7. Frameworks que automatizan TODO

Cuando quieras escalar de herramientas sueltas a investigación automatizada:
- **SpiderFoot** → automatiza 200+ módulos OSINT, genera grafos. El más completo gratis.
- **Maltego** → el estándar de la industria; mapea relaciones visualmente (versión Community gratis)
- **recon-ng** → framework con estructura tipo Metasploit para recon
- **OSINT Framework** (osintframework.com) → directorio gigante de herramientas por categoría
- **OBSIDIAN** (github.com/nadiee12312-lgtm/Obsidian-framework) → framework OSINT local, gratis y open-source, con grafo de relaciones y timeline

**El valor está en CONECTAR**, no solo recolectar:
- **Grafos** (tipo Maltego): nodos unidos por relaciones → revelan conexiones invisibles en una lista
- **Timelines:** ordenar eventos → revelan patrones y rutinas
- **Correlación:** el mismo dato (teléfono, username) en dos sitios → los conecta

---

## 8. OPSEC — protégete mientras investigas

Si investigas, NO dejes rastro tú mismo:
- **No uses tu cuenta personal** (algunas redes avisan quién te vio — LinkedIn)
- **Cuentas sock puppet:** identidades falsas dedicadas a investigar, sin datos reales tuyos, con su propio email/teléfono
- **VPN o Tor** para ocultar tu IP (`proxychains` para forzar herramientas por Tor)
- **Máquina virtual o navegador separado** solo para investigaciones
- **No interactúes** con el objetivo (no des like, no sigas, no comentes) — te delata
- **Cuidado con los webhooks/links** que el objetivo controla (pueden capturar TU IP)

---

## 9. Ética y legalidad (la línea que NO se cruza)

OSINT usa info **pública**, pero eso no lo hace siempre legal o ético:
- ✅ **Legal:** ver perfiles públicos, buscar en Google, leer registros públicos
- ⚠️ **Zona gris:** crear perfiles falsos, recopilación masiva de datos personales
- ❌ **Ilegal:** acosar, stalkear, doxear, usar datos para dañar, acceder a lo privado

**El doxing** (publicar datos privados de alguien para dañarlo) es ilegal y destruye vidas. El OSINT se usa para **proteger e investigar legítimamente** — verificar una estafa, conocer tu propia exposición, investigaciones autorizadas. Esa es la diferencia entre un investigador y un criminal.

---

## 🎓 Para practicar
- **TraceLabs** → CTFs de OSINT que ayudan a encontrar personas desaparecidas (OSINT para el bien)
- **Sherlock/Maigret sobre tu propio username** → mira qué tan expuesto estás
- **GeoGuessr** → entrena geolocalización
- **OSINT Dojo, Cipher387** → retos y recursos
- **Sigue con:** `libro-redes.md` (infraestructura) y `libro-python-hacking.md` (automatiza tu recon)

> ⬛ *La información quiere ser libre. El OSINT es el arte de escucharla. Úsalo para proteger, no para dañar.*
