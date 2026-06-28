# 💻➡️🦹 DE PROGRAMADOR A HACKER
> Ya sabes programar. Eso es una ventaja ENORME. Aquí la conviertes en superpoder de seguridad.
> Para el que viene del código. Versión 2.0

---

## 🎯 Tu ventaja secreta

Si ya programas, tienes una ventaja brutal sobre el 90% de los que empiezan en seguridad. ¿Por qué?

- **Entiendes cómo se construye el software** → por eso entiendes cómo se rompe.
- **No le tienes miedo al código** → puedes leer exploits, escribir tus herramientas, automatizar todo.
- **Piensas en lógica** → la seguridad es pura lógica aplicada al revés.

La mayoría de los hackers tuvieron que aprender a programar DESPUÉS. Tú ya lo tienes. Solo necesitas **girar tu mente**: de "construir cosas que funcionan" a "encontrar cómo las cosas fallan".

---

## 🔄 El giro mental: de constructor a rompedor

Como programador, piensas en el **camino feliz**: "el usuario escribe su nombre, le doy la bienvenida".

Un hacker piensa en el **camino roto**: "¿y si en vez de mi nombre escribo código? ¿Y si mando 10,000 caracteres? ¿Y si dejo el campo vacío? ¿Y si pongo comillas?"

> **La regla de oro del hacking, en términos de programador:** *toda entrada del usuario es peligrosa hasta que se demuestre lo contrario.* Tú, como dev, has confiado en la entrada. Como hacker, NUNCA confías en ella.

Cada `input`, cada parámetro, cada campo de formulario, cada cabecera HTTP — para ti antes eran "datos". Ahora son **puertas que alguien no validó bien.**

---

## 🕳️ Las vulnerabilidades, explicadas para devs

Estas son las fallas clásicas. Como programador, las vas a entender al instante:

### Inyección SQL
Escribiste código así, ¿verdad?
```python
query = f"SELECT * FROM users WHERE name = '{user_input}'"
```
**Ese código es vulnerable.** Si `user_input` es `' OR '1'='1`, la query se vuelve `... WHERE name = '' OR '1'='1'` → siempre verdadera → el atacante entra. Tú creaste el bug confiando en la entrada. Un hacker lo explota.
**La lección que ya sabes (o aprenderás):** usa consultas parametrizadas. SIEMPRE.

### XSS (Cross-Site Scripting)
Mostraste lo que el usuario escribió sin limpiarlo:
```javascript
element.innerHTML = userComment;
```
Si el comentario es `<script>robarCookies()</script>`, se ejecuta en el navegador de otros. Inyectaste código ajeno sin querer.
**La lección:** escapa/sanitiza toda salida.

### Otras que entenderás rápido
- **IDOR:** tu API usa `/user/123` y no verifica que el 123 sea TUYO. Cambias a `/user/124` y ves datos ajenos. Un fallo de autorización que como dev cometes sin darte cuenta.
- **Buffer overflow:** mandar más datos de los que cabe una variable, sobrescribiendo memoria. Tu base de C/C++ te da ventaja aquí.
- **Race conditions:** dos operaciones que chocan en el tiempo. Si has hecho concurrencia, ya las conoces.

**Tu superpoder:** mientras otros memorizan estas fallas, tú las **entiendes** porque has escrito el código que las causa. Eso te hace aprender 10 veces más rápido.

---

## 🐍 Escribe tus propias herramientas

Aquí es donde de verdad brillas. Mientras otros solo USAN herramientas, tú las **creas**. Python es ideal (pero cualquier lenguaje sirve). Ejemplos que puedes escribir HOY:

### Un escáner de puertos (tu propio mini-nmap)
```python
import socket
from concurrent.futures import ThreadPoolExecutor

def escanear(host, puerto):
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.settimeout(1)
    if s.connect_ex((host, puerto)) == 0:
        print(f"[+] Puerto {puerto} ABIERTO")
    s.close()

def escanear_host(host):
    with ThreadPoolExecutor(max_workers=100) as ex:
        for puerto in range(1, 1025):
            ex.submit(escanear, host, puerto)

# escanear_host("127.0.0.1")   # SOLO tu propia máquina
```
En 15 líneas hiciste un escáner multihilo. Eso es lo que un no-programador no puede hacer.

### Un buscador de subdominios, un fuzzer, un cracker de hashes...
Todo lo que las herramientas famosas hacen, tú lo puedes recrear para **entenderlo por dentro.** Y entender por dentro = dominar.

---

## 🌐 Hacking web para programadores

Si has hecho aplicaciones web, tienes una ventaja gigante: **sabes cómo funcionan por dentro.** Conoces el frontend, el backend, la base de datos, las APIs, las cookies, los tokens.

Eso significa que cuando atacas una web, no estás adivinando — **sabes dónde están los puntos débiles** porque tú los has programado:
- Sabes que las validaciones del frontend (JavaScript) se pueden saltar → atacas el backend directo.
- Sabes cómo funcionan las sesiones y los JWT → buscas cómo robarlos o falsificarlos.
- Sabes cómo se arman las APIs REST → pruebas cada endpoint, cambias cada parámetro.

**Tu herramienta clave:** **Burp Suite** (gratis la versión Community). Es un proxy que intercepta y modifica todo el tráfico entre el navegador y la web. Para un programador es como tener un debugger del tráfico HTTP. Lo vas a amar.

**Tu campo de práctica:** **DVWA** y **PortSwigger Web Security Academy** (gratis, el mejor). Hechos para que rompas webs legalmente.

---

## ⚙️ Automatiza TODO (tu otra ventaja)

Los que no programan hacen las cosas a mano. Tú las automatizas. En seguridad eso es oro:

- Automatizar reconocimiento (escanear, buscar subdominios, recolectar info) con scripts.
- Encadenar herramientas (el resultado de una alimenta a la siguiente).
- Procesar miles de resultados con código en vez de a ojo.
- Crear tu propio "framework" de herramientas a tu medida.

Mientras otro tarda horas haciendo clic, tú corres un script y te vas por un café. Esa es la diferencia que te da saber programar.

---

## 📚 Librerías de Python que te van a encantar

Como dev, vas a apreciar estas:
- **`requests`** — para hablar con webs y APIs.
- **`scapy`** — crear y manipular paquetes de red a mano (poderosísima).
- **`beautifulsoup4`** — extraer datos de páginas (scraping).
- **`pwntools`** — el framework para exploiting y CTFs (binary exploitation).
- **`paramiko`** — SSH desde Python.
- **`impacket`** — protocolos de red de Windows.

---

## 🎮 Cómo practicar (aprovechando que programas)

1. **Reescribe herramientas que ya existen.** Haz tu propio escáner, tu fuzzer, tu buscador de subdominios. Entender por dentro = dominar.
2. **CTFs (Capture The Flag):** competencias de hacking legales y divertidas. **picoCTF** es perfecto para empezar, y hay muchos de programación/exploiting que te van a encantar.
3. **pwntools + binary exploitation:** si te gustan los retos de bajo nivel, este mundo es adictivo para programadores.
4. **Bug bounty (más adelante):** plataformas como HackerOne **te PAGAN** por encontrar fallos en webs reales, legalmente. Tu skill de programador te da ventaja real aquí.

---

## 🧭 Tu ruta acelerada (porque ya programas)

No necesitas el camino lento. Este es tu atajo:

1. **Fundamentos rápidos:** redes y Linux (lee `libro-redes.md` y `libro-linux-desde-cero.md` — vas a ir volando porque ya piensas en lógica).
2. **Salta a lo tuyo:** escribe tu primer escáner de puertos HOY (el de arriba). Sentirás el click.
3. **Hacking web:** instala Burp, métete a PortSwigger Academy. Tu base de dev lo hace fácil.
4. **CTFs:** picoCTF para practicar exploiting y reversing.
5. **Crea, crea, crea:** tu mejor forma de aprender es construir herramientas. Tú puedes hacerlo, otros no.

---

## 🔍 Tu superpoder único: secure code review

Esto casi nadie más lo puede hacer, pero tú SÍ: **leer código fuente buscando vulnerabilidades.** Las empresas pagan muy bien por esto.

Cuando revisas código con ojo de seguridad, buscas patrones peligrosos:

```python
# 🚩 BANDERAS ROJAS que saltan a la vista de un dev-hacker:

eval(user_input)                    # ejecución de código arbitrario
os.system(f"ping {user_input}")     # command injection
query = f"SELECT * WHERE id={id}"   # SQL injection
pickle.loads(user_data)             # deserialización insegura
element.innerHTML = userData        # XSS
open(user_path)                     # path traversal
password == stored_password         # comparación no constante (timing)
DEBUG = True                        # en producción = fuga de info
api_key = "sk-1234..."              # secreto hardcodeado
```

**Herramientas que automatizan esto (SAST):**
```bash
semgrep --config=auto .      # busca patrones vulnerables en tu código ⭐
bandit -r proyecto/          # análisis de seguridad para Python
gitleaks detect              # busca secretos/API keys en el repo
trufflehog git <repo>        # secretos en el historial de git
```

**Tu ventaja:** mientras un pentester sin código adivina desde fuera (caja negra), tú lees el código y ves el bug directo (caja blanca). Eso te hace valiosísimo en **bug bounty**, **auditorías** y **DevSecOps**.

---

## 💬 La verdad

Muchos hackers darían lo que fuera por tener tu base de programación. La mayoría tuvo que aprenderla a la fuerza, después, sufriendo. Tú ya la tienes.

Solo te falta **girar la mente**: de construir a romper, de confiar en la entrada a desconfiar de ella, de "¿cómo lo hago funcionar?" a "¿cómo lo hago fallar?".

Una vez que hagas ese giro, vas a avanzar más rápido que casi todos. **El código es tu ventaja. Úsala.**

---

> ⬛ *Las herramientas hacen usuarios. El código hace hackers. Tú ya escribes código — solo aprende a leerlo al revés, buscando dónde se rompe. Esa es toda la diferencia.*
>
> **Sigue con:** `libro-python-hacking.md`, `libro-web-hacking.md` y `libro-ciberseguridad.md` para profundizar.
