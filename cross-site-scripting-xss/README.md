# Cross-site scripting (XSS)

## 🔍 ¿Qué es Cross-Site Scripting (XSS)?

XSS es una vulnerabilidad web que permite a los atacantes inyectar JavaScript malicioso en sitios web, afectando a los usuarios que los visitan. Rompe la política de mismo origen del navegador, permitiendo a los atacantes acceder o manipular datos sensibles, suplantar usuarios o realizar acciones en su nombre.

***

## 🧪 Tipos de XSS

### 🔄 Reflected XSS:

El payload se incluye en la URL o en la petición y se refleja en la respuesta.\
Se activa cuando la víctima hace clic en un enlace manipulado.

### 💾 Stored XSS:

El payload se guarda en el servidor (por ejemplo, en un comentario).\
Se ejecuta automáticamente cuando cualquier usuario visita la página afectada.

### 🌐DOM-based XSS:

La vulnerabilidad está en el JavaScript del lado cliente.\
La entrada maliciosa fluye desde una fuente (como location.search) hacia un sink peligroso (como innerHTML o eval()).

***

## 📥 Sources y Sinks comunes

### 🔗 Sources (entrada controlada por el atacante):

* location.search
* document.referrer
* document.cookie
* location.hash

### 🪝 Sinks (destinos donde se inyecta peligrosamente la entrada):

* innerHTML
* outerHTML
* document.write()
* eval()
* setTimeout()
* Funciones del DOM de jQuery (html(), append(), attr())

***

## 📍 Contextos de XSS (dónde cae la inyección)

### 🏷️ Contexto HTML:

La entrada se inserta entre etiquetas\
Ejemplo: `<p>INPUT</p>`

### 🧩 Contexto de atributo:

Dentro de atributos HTML\
Ejemplo: `<a href="INPUT">`

### ⚙️ Contexto JavaScript:

Dentro de `<script>` o JS inline\
Ejemplo: `var msg = 'INPUT';`

### ⚡ Contexto de eventos:

En atributos como onclick, onerror\
Ejemplo: `<img src=x onerror="INPUT">`

### 🔙 Contexto template literal:

Dentro de backticks en JS\
Ejemplo: `` `Hello ${INPUT}` ``

### 🛡️ Contexto con CSP:

El entorno tiene Content Security Policy\
Los payloads deben evadir restricciones (por ejemplo, usando imágenes o escapes de sandbox)

***

## 💥 Impacto de XSS

* Robo de cookies, sesiones o credenciales
* Secuestro de cuentas o suplantación
* Modificación del contenido (defacement)
* Bypass de CSRF robando tokens
* Ejecución de acciones como la víctima

***

## 🛡️ Mitigaciones

### ✅ Validación de entrada:

* Filtrar la entrada según formato esperado

### 🔤 Output encoding:

* Codificar datos antes de renderizarlos

### 🛠️ CSP:

* Usar Content Security Policy para limitar scripts

### 🧰 Frameworks seguros:

* Usar librerías con protección integrada (ej: React)

***

## 🚪 Bypasses comunes

* Uso de `<img>` o `<svg>` con onerror/onload
* Romper atributos o strings JS con comillas
* Codificar payloads (`&lt;script&gt;`)
* Técnicas específicas de frameworks (Angular, CSP bypass)
