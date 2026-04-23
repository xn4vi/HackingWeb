# JWT

## 🔑 ¿Qué es un JSON Web Token?

Un JSON Web Token (JWT) es un formato de token compacto y seguro para URL que se utiliza para transmitir información (claims) entre partes. Consta de tres partes codificadas en Base64URL separadas por puntos:

```
<header>.<payload>.<signature>
```

➡️ **Ejemplo de Header:**

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

➡️ **Ejemplo de Payload:**

```json
{
  "iss": "portswigger",
  "exp": 1773753850,
  "sub": "wiener"
}
```

La firma se calcula sobre `base64url(header) + "." + base64url(payload)` utilizando el algoritmo especificado en el header y una clave secreta que posee el servidor.\
El servidor verifica la firma en cada petición: si es válida, confía en los datos del payload.

***

## ⚠️ Por qué los JWT son vulnerables

El problema principal es que el propio token le dice al servidor cómo debe verificarse.\
El header especifica el algoritmo, el identificador de clave (`kid`) e incluso, opcionalmente, la propia clave (`jwk`, `jku`).

Si el servidor confía ciegamente en estos parámetros, un atacante puede manipularlos para saltarse la verificación.

➡️ **Clases comunes de vulnerabilidades:**

* Sin verificación de firma: el servidor acepta cualquier token
* Aceptación de `alg: none`: el servidor acepta tokens sin firma
* Claves débiles: claves HMAC cortas o comunes crackeables con hashcat
* Inyección JWK: el servidor usa una clave pública proporcionada por el atacante en el header
* Inyección JKU: el servidor descarga la clave desde una URL controlada por el atacante
* Path traversal en `kid`: el servidor usa `kid` como ruta de archivo
* Confusión de algoritmo (RS256 → HS256): uso incorrecto de claves públicas como secretos HMAC

***

## 📐 Estructura del JWT en detalle

###  Parámetros del Header relevantes para ataques:

<table><thead><tr><th width="183.54541015625">Parámetro</th><th>Propósito</th><th>Superficie de ataque</th></tr></thead><tbody><tr><td>alg</td><td>Algoritmo de firma</td><td>Cambiar a <code>none</code> o a <code>HS256</code></td></tr><tr><td>kid</td><td>Identificador de clave</td><td>Path traversal</td></tr><tr><td>jwk</td><td>Clave pública embebida</td><td>Inyectar clave del atacante</td></tr><tr><td>jku</td><td>URL de claves</td><td>Apuntar a servidor malicioso</td></tr></tbody></table>

###  Claims del Payload relevantes:

<table><thead><tr><th width="185.3636474609375">Claim</th><th>Propósito</th><th>Objetivo del ataque</th></tr></thead><tbody><tr><td>sub</td><td>Identidad del usuario</td><td>Cambiar a administrador</td></tr><tr><td>iss</td><td>Emisor</td><td>Poco explotado</td></tr><tr><td>exp</td><td>Expiración</td><td>Extender validez</td></tr></tbody></table>

***

## 🚫 Ataque `alg: none`

La especificación JWT define `none` como un algoritmo válido que significa “sin firma”.

Algunas librerías lo aceptan.

**Ataque:**

1. Tomar un JWT válido
2. Cambiar `"alg": "HS256"` → `"alg": "none"`
3. Cambiar `"sub": "wiener"` → `"sub": "administrator"`
4. Re-codificar en Base64URL
5. Construir:

```
<header_modificado>.<payload_modificado>.
```

(Nótese el punto final sin firma)

***

## 🔨 Fuerza bruta de claves HMAC débiles

Cuando el algoritmo es HS256, HS384 o HS512, la clave es simétrica.

Si es débil, se puede crackear offline con hashcat:

```bash
hashcat -a 0 -m 16500 <jwt_token> jwt.secrets.list
```

* Modo 16500 → específico para JWT
* Una vez obtenida la clave, puedes firmar cualquier token modificado

***

## 🔐 Inyección en el header JWK

El parámetro `jwk` permite incluir una clave pública en el token.

La vulnerabilidad aparece cuando el servidor usa esa clave para verificarse a sí mismo.

➡️ **Flujo del ataque:**

1. Generar un par de claves RSA
2. Modificar el payload (ej: admin)
3. Añadir `jwk` con tu clave pública
4. Firmar con tu clave privada
5. El servidor verifica usando tu clave → firma válida

***

## 🌐 Inyección en el header JKU

El parámetro `jku` indica dónde descargar las claves.

Si el servidor no valida la URL:

➡️ **Ataque:**

1. Generar claves RSA
2. Subir un JWK Set a un servidor controlado
3. Añadir `jku` apuntando a ese servidor
4. Ajustar `kid`
5. Firmar el token
6. El servidor descarga tu clave y valida → acceso conseguido

***

## 🗂️ Path Traversal en `kid`

El parámetro `kid` identifica la clave.

Si se usa como ruta de archivo sin sanitizar:

```
../../../dev/null
```

En Linux:

* `/dev/null` → archivo vacío
* Si firmas con clave nula (`AA==` en Base64), la verificación puede pasar

***

## 🔄 Confusión de algoritmo (RS256 → HS256)

Este es el ataque más sutil.

En RS256:

* Se firma con clave privada
* Se verifica con clave pública

Problema: si el servidor también acepta HS256 y reutiliza la clave pública como secreto HMAC.

➡️ **Ataque:**

1. Obtener la clave pública (`/jwks.json`)
2. Cambiar `alg` a HS256
3. Firmar usando la clave pública como secreto
4. El servidor verifica con esa misma clave → firma válida

🔴 La clave pública NO es secreta.\
Pero si se usa como secreto HMAC, permite falsificar tokens.

Si no está expuesta, puede derivarse matemáticamente usando herramientas como `rsa_sign2n`.
