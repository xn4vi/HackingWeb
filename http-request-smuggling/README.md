---
icon: folder-open
---

# HTTP request smuggling

Consiste en incrustar una petición dentro de otra petición. Con esto conseguimos que lleguen al back 2 peticiones y las considere como independientes, cuando únicamente estamos enviando una única petición:

<figure><img src="../../../.gitbook/assets/image (1324).png" alt="" width="331"><figcaption></figcaption></figure>

Esto sería una petición smuggleada. Estamos enviando una única petición pero al back le llegarán 2 ya que se "confunde" y se traga todo como si fuese una única petición.

Para que esto ocurra, tienen que cumplirse unas condiciones:

* Protocolo HTTP 1 obligatoriamente
* El manejo inadecuando de la cabecera TRANSFER-ENCODING



Explicación de transfer-encoding:

Para que esta vulnerabilidad exista, el front o el back deben interpretar la cabecera transfer-encoding, sino, esta vulnerabilidad no va a existir.

La cabecera transfer-encoding toma el valor de chunked. Esto especifica, que se va a enviar el cuerpo de la petición "por partes".

Ejemplo: (los numeros son un ejemplo, no estan bien puestos)

<figure><img src="../../../.gitbook/assets/image (1325).png" alt=""><figcaption></figcaption></figure>

La cabecera Content-length toma el valor del numero de bytes totales del cuerpo de la peticion

La cabecera chunked, le indica que se enviara por trozos y los numeros de la peticion, indican el tamaño de cada trozo. Los numeros, se ponen en hexadecimal para el TE.

***

### Escenario 1 CL.TE

El front interpreta CL y no interpreta TE

El backend interpreta TE y no CL

La petición llega al front, ya sabemos que la cabecera TE va a ser ignorada.

Si el CL no especifica exactamente el numero de bytes del cuerpo de la petición, va a rechazar la misma y nunca va a llegar al back. Ahora bien, si modificamos el valor del CL y le especifiamos el numero exacto, la peticion va a pasar al back

Ahora la peticion llega al back y aqui si se interpreta la cabecera TE. Con el valor 0, le decimos que el chunked (el trozo) termina ahi, entonces, lo siguiente lo va a interpretar como una nueva petición:

<figure><img src="../../../.gitbook/assets/image (1327).png" alt=""><figcaption></figcaption></figure>

En CL.TE, para validar que efectivamente estamos en este escenario podemos hacerlo asi:

<figure><img src="../../../.gitbook/assets/image (1314).png" alt=""><figcaption></figcaption></figure>

El front no admite la cabecera TE, asi que nos da un error de sincronizacion.

<figure><img src="../../../.gitbook/assets/image (1312).png" alt=""><figcaption></figcaption></figure>

Enviamos una petición bien formada que responde 200.

El chunked espera un 0 para que sea valido, si le ponemos una x, vemos que cambia la respuesta y no la acepta:

<figure><img src="../../../.gitbook/assets/image (1313).png" alt=""><figcaption></figcaption></figure>

Ahora ya estamos fijo que estamos en ese escenario.

Para poner bien el tamaño del CL, medimos hasta el 0:

<figure><img src="../../../.gitbook/assets/image (1315).png" alt=""><figcaption></figcaption></figure>

Al enviar la petición 2 veces, causamos el error 404 haciendo un GET a /error

***

### Escenario 2 TE.CL

Ahora el front SI interpreta TE

Vamos a construir una peticion smugleada

<figure><img src="../../../.gitbook/assets/image (1319).png" alt=""><figcaption></figcaption></figure>

Ese valor en rojo es el del chunked que interpreta el frontend, que se pone en hexadecimal y se calcula en base al texto seleccionado. En este caso, corresponde a 9f

El 0 del final, marca el final del chunked y el front dice: ok, llega el 0 se acaba el chunked.

Siguiente paso:

<figure><img src="../../../.gitbook/assets/image (1318).png" alt=""><figcaption></figcaption></figure>

Poner el CL de la segunda petición. Se calcula el base al texto seleccionado, pero le ponemos de mas, no hay problema.

Seguimos:

<figure><img src="../../../.gitbook/assets/image (1323).png" alt=""><figcaption></figcaption></figure>

Hay que colocar el valor del CL global, que incluye solo lo seleccionado. En este caso, 4

***

### 1. ¿Qué es HTTP Request Smuggling?

HTTP request smuggling es una técnica que interfiere con cómo un sitio web procesa secuencias de peticiones HTTP recibidas de uno o más usuarios. Cuando un servidor front-end (reverse proxy, load balancer, CDN) reenvía peticiones a un servidor back-end, comparten una conexión TCP para mayor eficiencia. Si estos dos servidores no están de acuerdo sobre dónde termina una petición y comienza la siguiente, un atacante puede anteponer contenido malicioso a la siguiente petición de otro usuario.

#### Impacto:

* Bypass de controles de seguridad del front-end (WAFs, restricciones de acceso)
* Robo de credenciales y cookies de sesión de otros usuarios
* Ejecución de XSS reflejado sin interacción del usuario
* Envenenamiento de cachés web para servir contenido malicioso
* Secuestro de cuentas admin mediante response queue poisoning

***

### 2. Cómo funciona Request Smuggling

El problema principal es el desacuerdo entre dos cabeceras que especifican la longitud del cuerpo de la petición:

#### Content-Length - especifica el tamaño del body en bytes:

```http
POST / HTTP/1.1
Content-Length: 11

hello=world
```

#### Transfer-Encoding: chunked - el body se envía en chunks:

```http
POST / HTTP/1.1
Transfer-Encoding: chunked

b
hello=world
0
```

Cuando ambas cabeceras están presentes y el front-end y el back-end priorizan diferentes, se produce un desync.

***

### 3. Variantes clásicas de Smuggling

#### CL.TE - Front-end usa Content-Length, back-end usa Transfer-Encoding:

```http
POST / HTTP/1.1
Content-Length: 30
Transfer-Encoding: chunked

0

GET /admin HTTP/1.1
Foo: x
```

#### TE.CL - Front-end usa Transfer-Encoding, back-end usa Content-Length:

```http
POST / HTTP/1.1
Content-Length: 4
Transfer-Encoding: chunked

5c
GPOST / HTTP/1.1
Content-Length: 15

x=1
0
```

#### TE.TE - Ambos soportan Transfer-Encoding, pero la ofuscación provoca desacuerdo:

```http
Transfer-Encoding: chunked
Transfer-Encoding: x
```

***

### 4. Variantes de HTTP/2 Smuggling

HTTP/2 introduce nuevos vectores cuando los servidores hacen downgrade a HTTP/1.1:

#### H2.CL:

* Front-end usa longitud de frame HTTP/2
* Back-end usa Content-Length tras downgrade

#### H2.TE:

* Front-end: longitud de frame HTTP/2
* Back-end: `Transfer-Encoding: chunked` → desync

#### CRLF Injection:

```
# Header HTTP/2
foo: bar\r\nTransfer-Encoding: chunked

# HTTP/1.1 lo interpreta como dos headers
```

#### Request Tunnelling:

* Se hace smuggling usando peticiones HEAD
* El front-end sobrelee y expone respuestas encapsuladas

***

### 5. Variantes impulsadas por el navegador

#### CL.0:

```http
POST /resources/image.svg HTTP/1.1
Content-Length: 30

GET /admin HTTP/1.1
Foo: x
0
```

#### 0.CL:

* Front-end ignora Content-Length
* Back-end lo procesa
* Requiere un early response gadget

#### Client-Side Desync:

```javascript
fetch(url, {method:'POST', body:'smuggled request', mode:'cors'})
.catch(() => fetch(url, {mode:'no-cors'}))
```

#### Pause-Based:

* Enviar headers
* Pausar antes del body
* El servidor responde sin consumir el body
* El body se convierte en la siguiente petición

***

### 6. Tipos de ataque

#### Response Queue Poisoning:

* Se inyecta una petición completa
* El back-end envía 2 respuestas
* El front-end espera 1 → desincronización
* Usuarios reciben respuestas incorrectas

#### Web Cache Poisoning:

* Se inyecta redirección maliciosa
* Se cachea para un recurso estático
* Todos los usuarios reciben contenido malicioso

#### Web Cache Deception:

* Se smugglea `GET /my-account`
* Se usa la sesión de la víctima
* La página se cachea como recurso público

***

### 7. Buenas prácticas de defensa

#### Usar HTTP/2 end-to-end:

* Evitar downgrade a HTTP/1.1
* HTTP/2 elimina ambigüedad en límites de petición

#### Normalizar peticiones ambiguas:

* Rechazar peticiones con `Content-Length` y `Transfer-Encoding`
* Eliminar `Transfer-Encoding` en downgrade
* Sanear secuencias `\r\n`

#### No reutilizar conexiones back-end:

* Usar una conexión por cliente
* Evitar contaminación entre usuarios
* Validar correctamente los límites de petición

#### Asegurar endpoints estáticos:

* Asegurar que consumen correctamente el body
* No asumir que no recibirán POST requests
