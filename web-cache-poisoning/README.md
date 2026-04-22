# Web cache poisoning

## 🔍 ¿Qué es Web Cache Poisoning?

Web Cache Poisoning es un ataque en el que un atacante explota el comportamiento de una caché web para servir contenido malicioso a otros usuarios. El ataque funciona mediante:

* Identificar entradas que afectan a la respuesta del servidor pero que **NO están incluidas en la cache key**
* Crear una petición con valores maliciosos en esas entradas no incluidas en la cache key
* Conseguir que la respuesta envenenada se almacene en la caché
* Los usuarios posteriores que soliciten el mismo recurso reciben la respuesta cacheada envenenada

### 🗝️ Concepto clave - Cache Keys:

Una **cache key** es el conjunto de componentes de la petición que se utilizan para determinar si una respuesta cacheada coincide con una nueva petición.

Normalmente incluye:

* Método HTTP
* Ruta de la URL
* Cabecera Host
* A veces parámetros de query

Componentes que **NO están en la cache key (unkeyed inputs)** pueden aun así influir en la respuesta.

Esta diferencia entre lo que afecta a la respuesta y lo que determina la caché es donde está la vulnerabilidad.

### 🔄 Diferencia con Cache Deception:

* **Cache Deception**: Engaña a la caché para almacenar datos privados de la víctima
* **Cache Poisoning**: Inyecta contenido malicioso en la caché que se sirve a todos los usuarios

***

## ⚙️ Cómo funciona Web Cache Poisoning

###  Flujo básico:

1. El atacante identifica una entrada no incluida en la cache key (por ejemplo, `X-Forwarded-Host`)
2. El servidor usa esa entrada para generar la respuesta
3. El atacante envía una petición con valor malicioso
4. La caché guarda la respuesta usando solo la cache key
5. Otro usuario solicita la misma URL
6. La caché devuelve la respuesta envenenada
7. El navegador de la víctima ejecuta JavaScript malicioso

###  El desajuste:

```
Cache key:        GET / Host: vulnerable.com
Unkeyed input:    X-Forwarded-Host: attacker.com
Server response:  <script src="https://attacker.com/resources/js/tracking.js">
Cached as:        Response for "GET / Host: vulnerable.com"
Resultado:        Todos los usuarios cargan JS del atacante
```

***

## 🗝️ Cache keys y unkeyed inputs

###  Componentes típicos de la cache key:

✅ **Incluidos:**

* Método
* Ruta
* Host
* Query (a veces)
* Cookies específicas (a veces)

**🚫 No incluidos pero influyen:**

* X-Forwarded-Host
* X-Forwarded-Scheme
* X-Original-URL
* X-Host
* Cookies
* User-Agent (a veces)
* Accept-Language
* Origin

***

###  Cómo encontrarlos:

➡️ **Manual:**

1. Enviar petición con header extra
2. Ver si aparece en la respuesta
3. Enviar sin header
4. Si la respuesta cacheada no cambia → es unkeyed

➡️ **Automatizado:**

* Param Miner (Burp)
* Descubre headers ocultos

***

## 🎯 Tipos de Cache Poisoning

###  Basado en headers:

```http
X-Forwarded-Host: attacker.com
```

Respuesta:

```html
<script src="https://attacker.com/tracking.js"></script>
```

###  Basado en cookies:

```http
Cookie: fehost="}</script><script>alert(1)</script>
```

###  Basado en query string:

```http
GET /?'><script>alert(1)</script>
```

###  Basado en parámetros:

```http
utm_content='/><script>alert(1)</script>
```

***

## ⚔️ Técnicas de explotación

###  Headers unkeyed:

1. Encontrar header vulnerable
2. Subir JS malicioso
3. Envenenar caché
4. Usuarios cargan JS

###  Múltiples headers:

```http
X-Forwarded-Host: attacker.com
X-Forwarded-Scheme: http
```

Resultado: redirección maliciosa

###  Parameter Cloaking:

```
/js/geolocate.js?callback=setCountryCookie&utm_content=1;callback=alert(1)
```

* Cache interpreta uno
* Server interpreta dos

###  Fat GET:

```http
GET /js/geolocate.js
Content-Length: 18

callback=alert(1)
```

###  Normalización de URL:

```
/random<script>alert(1)</script>
```

***

## 🛤️ Vectores de ataque

###  XSS vía script:

* Target: tracking.js
* Ataque: header

###  DOM XSS:

```json
{"country":"<img src=1 onerror=alert(document.cookie) />"}
```

###  Redirección:

* X-Original-URL
* X-Forwarded-Host

###  Fragmentos internos:

* Caché interna persiste

***

## 💥 Cache Busters

###  Query:

```http
?cachebuster=abc123
```

###  Headers:

```http
Origin: https://cachebuster123.com
```

###  Cookies:

```http
Cookie: cachebuster=abc123
```

###  User-Agent:

```http
User-Agent: Mozilla cachebuster123
```

***

## 🕵️ Métodos de detección

###  Detectar caché

* X-Cache
* Age
* Cache-Control

### Unkeyed inputs

```http
X-Forwarded-Host: test123
```

###  Puntos de reflexión

* script
* link
* meta
* JS
* redirect

###  Verificar

1. Enviar payload
2. Ver respuesta
3. Repetir sin header
4. Si persiste → vulnerable

###  Automatizado:

```python
requests.get(...)
```

***

## 🛡️ Defensa

###  Minimizar inputs:

```
proxy_cache_key "$scheme$request_method$host$request_uri";
```

###  Eliminar headers:

* X-Forwarded-Host
* X-Original-URL

###  Cache-Control:

```http
Cache-Control: no-store
```

###  Validación:

```python
if forwarded_host not in allowed_hosts:
    forwarded_host = None
```

###  Encoding:

```html
<script src="/tracking.js"></script>
```

***

## 🚀 Técnicas avanzadas

###  Cache key injection:

```http
Origin: x\r\nContent-Length...
```

###  Fragmentación interna:

* Persistencia interna

###  Encadenamiento:

1. Redirect
2. Host
3. DOM XSS

###  Ataques dirigidos:

* User-Agent específico
