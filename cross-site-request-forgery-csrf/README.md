# Cross-site request forgery (CSRF)

## 🔍 ¿Qué es CSRF?

**Cross-Site Request Forgery (CSRF)** es un ataque que obliga a un usuario autenticado a ejecutar acciones no deseadas en una aplicación web en la que ya ha iniciado sesión.

Funciona porque:

* El navegador envía automáticamente credenciales (cookies, headers de autenticación)
* La aplicación confía en esas credenciales sin validar el origen
* El atacante construye una petición maliciosa
* El navegador de la víctima la envía
* El servidor la procesa como legítima

### 🎯 Impacto

* Cambio de contraseña o email
* Transferencias de dinero
* Modificación de configuración
* Eliminación de datos
* Acciones administrativas
* Puede combinarse con XSS

***

## ⚡ Cómo funciona un ataque CSRF

### 🔄 Flujo básico

```html
<form action="https://victim-bank.com/transfer" method="POST">
    <input name="amount" value="10000">
    <input name="to" value="attacker-account">
</form>
<script>document.forms[0].submit();</script>
```

Cuando la víctima visita la web del atacante:

1. El formulario se envía automáticamente
2. El navegador incluye cookies de sesión
3. El servidor procesa la petición
4. Se ejecuta la acción (ej. transferencia)

***

## 🛡️ Mecanismos de defensa

### 🔑 Tokens CSRF

```html
<input name="csrf" value="a8b7c6d5e4f3g2h1">
```

* Token único por sesión
* Validado en el servidor

### 🍪 Cookies SameSite

```http
Set-Cookie: session=abc123; SameSite=Strict
Set-Cookie: session=abc123; SameSite=Lax
Set-Cookie: session=abc123; SameSite=None; Secure
```

* **Strict**: nunca se envía en cross-site
* **Lax**: solo en navegación GET principal
* **None**: siempre (requiere Secure)

### 📍 Validación de Referer

* Comprobar que el dominio coincide
* Bloquear si es externo

### 🛠️ Cabeceras personalizadas

```javascript
fetch('/api/action', {
    headers: {'X-CSRF-Protection': 'true'}
})
```

Los navegadores no incluyen headers personalizados en requests simples.

***

## 🚪 Técnicas de bypass

### ❌ Fallos en validación de token

➡️ **Sin validación**

```http
POST /change-email
email=attacker@evil.com
```

➡️ **Token no ligado a sesión**

* Se puede reutilizar token de otro usuario

➡️ **Token en cookie no segura**

* Se puede inyectar mediante CRLF

### 🔄 Bypass por método HTTP

```http
POST /change-email   (requiere token)
GET /change-email    (no requiere)
```

### 🍪 Bypass SameSite

➡️ **Method override**

```http
GET /change-email?_method=POST&email=evil@evil.com
```

➡️ **Redirect desde mismo sitio**

```http
GET /redirect?url=/change-email?email=evil@evil.com
```

➡️ **Ventana de refresco de cookies**

* Login OAuth refresca cookies
* Ventana temporal para ataques

➡️ **XSS en subdominio**

* Permite saltarse SameSite=Strict
* Ej: `cms.victim.com` → ataque a `victim.com`

### 🌐 Bypass Referer

➡️ **Eliminar Referer**

```html
<meta name="referrer" content="never">
```

➡️ **Validación débil**

```http
Referer: https://evil.com/?victim.com
```

***

## 📋 Metodología de testeo

### 1️⃣ Identificar acciones críticas

* Cambio de contraseña
* Cambio de email
* Transferencias
* Configuración
* Eliminación de datos

### 2️⃣ Testear protecciones

1. Eliminar token
2. Reutilizar token
3. Usar token de otro usuario
4. Cambiar POST → GET
5. Eliminar Referer

### 3️⃣ Testear SameSite

* Ver atributo en cookies
* Probar method override
* Buscar redirects
* Revisar OAuth
* Buscar XSS en subdominios

### 4️⃣ Crear exploit

1. HTML con formulario malicioso
2. Auto-submit con JS
3. Hostear en servidor atacante
4. Engañar a la víctima

***

## ✅ Buenas prácticas de defensa

### 🔑 Usar tokens CSRF

```python
if request.form['csrf_token'] != session['csrf_token']:
    abort(403)
```

### 🍪 Usar SameSite

```http
Set-Cookie: session=abc; SameSite=Strict; Secure
```

### 📍 Validar Origin / Referer

```python
if request.headers.get('Origin') not in allowed_origins:
    abort(403)
```

### 🔄 Double Submit Cookies

* Token en cookie y en formulario
* El servidor compara ambos

### 🛡️ Protecciones del framework

```beancount
{% csrf_token %}
```

```ruby
<%= form_authenticity_token %>
```

```javascript
app.use(csrf())
```
