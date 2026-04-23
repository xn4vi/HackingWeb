# Authentication

## 🔐 ¿Qué son las vulnerabilidades de autenticación?

Las **vulnerabilidades de autenticación** son fallos de seguridad que permiten a un atacante:

* Saltarse mecanismos de login
* Acceder a cuentas sin autorización
* Escalar privilegios

No suelen deberse a fallos criptográficos, sino a errores de lógica, validación o implementación.

***

## 🚨 Fallos comunes de autenticación

* **Enumeración de usuarios**: distinguir usuarios válidos por respuestas o tiempos
* **Ataques de fuerza bruta**: probar múltiples contraseñas
* **Fallos en reset de contraseña**: lógica defectuosa en recuperación
* **Bypass de 2FA**: errores en validación o gestión de sesión
* **Problemas de sesión**: cookies manipulables, session fixation
* **Credential stuffing**: uso de credenciales filtradas de otras brechas

***

## 🧪 Técnicas de bypass

### 🔍 Análisis de respuestas

* Diferencias en mensajes o códigos HTTP
* Permiten detectar usuarios válidos

### ⏱️ Análisis de tiempos

* Diferencias en tiempo de respuesta
* Permiten inferir información

### 🌐 Rotación de IP

* Uso de headers como:

```http
X-Forwarded-For: 1.2.3.4
```

Para evitar rate limiting.

### 🧬 Manipulación de parámetros

* Modificar campos ocultos:
  * username
  * user\_id

### 🍪 Manipulación de cookies

* Forjar o modificar cookies de sesión

### 🤖 Automatización con macros

* Uso de Burp Suite para automatizar flujos complejos

***

## 📲 Vulnerabilidades en 2FA

* **Bypass directo**: acceder a URLs sin pasar por verificación
* **Códigos débiles**: PIN de 4-6 dígitos
* **Manipulación de cookies**: suplantar usuarios
* **Errores de lógica**: aceptar códigos incorrectos o expirados
* **Sin rate limiting**: intentos ilimitados

***

## 🔑 Vulnerabilidades en reset de contraseña

* **Tokens predecibles**
* **Reutilización de tokens**
* **Parámetros ocultos modificables**
* **Host Header Injection** en enlaces de reset
* **Falta de verificación por email**

***

## 🍪 Problemas de sesión y cookies

* Cookies generadas de forma débil
* Uso de Base64 sin cifrado
* Session fixation (IDs predecibles o fijables)
* Sesiones no invalidadas tras logout

***

## 🛡️ Estrategias de mitigación

### 🔐 Contraseñas seguras

* Longitud mínima
* Complejidad
* Historial

### 🚫 Bloqueo de cuentas

* Tras varios intentos fallidos
* Con umbrales razonables

### ⏱️ Rate limiting

* Por IP
* Por cuenta

### 📱 2FA seguro

* Códigos de al menos 6 dígitos
* TOTP
* Llaves hardware

### 🎟️ Tokens de reset seguros

* Aleatorios criptográficamente
* De un solo uso
* Con expiración

### 🍪 Gestión de sesión segura

* Cookies con flags:
  * HttpOnly
  * Secure
* Invalidación correcta

### 🕵️‍♂️ Evitar enumeración

* Respuestas idénticas para usuario válido/inválido

### 🤖 CAPTCHA

* Tras varios intentos fallidos

### 📊 Logging y monitorización

* Detectar patrones sospechosos
* Alertas de actividad anómala

***

## 🛠️ Herramientas y técnicas

### 💣 Burp Intruder

* Fuerza bruta
* Enumeración

### 🧪 Procesamiento de payloads

* Encadenar transformaciones:
  * MD5 → Base64

### 🤖 Macros

* Automatizar flujos de autenticación

### 📡 Headers personalizados

```http
X-Forwarded-For
X-Forwarded-Host
```

### 🔍 Filtrado de respuestas

* Detectar usuarios válidos
* Identificar diferencias
