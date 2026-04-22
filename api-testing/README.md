---
icon: folder-open
---

# API Testing

### 🌐 ¿Qué es API Testing?

API (Application Programming Interface) testing consiste en evaluar la seguridad, funcionalidad y fiabilidad de APIs. A diferencia del testing web tradicional que se centra en interfaces de usuario, el API testing examina:

* Descubrimiento de endpoints - encontrar endpoints documentados y no documentados
* Manipulación de parámetros - probar cómo las APIs manejan parámetros modificados o inesperados
* Autenticación y autorización - verificar controles de acceso adecuados
* Validación de entrada - probar vulnerabilidades de inyección
* Lógica de negocio - explotar fallos en workflows de la API
* Rate limiting - comprobar vulnerabilidades de agotamiento de recursos

Las APIs son objetivos especialmente atractivos porque:

* No reciben el mismo nivel de revisión de seguridad que las interfaces de usuario
* Exponen funcionalidad interna y estructuras de datos
* Se asumen como “internas” pero en realidad son accesibles
* Manejan operaciones sensibles sin validación adecuada

***

### 🔌 Tipos de APIs

#### 🌍 REST APIs:

```http
GET /api/users/123
POST /api/users
PUT /api/users/123
DELETE /api/users/123
PATCH /api/users/123
```

* Arquitectura más común
* Usa métodos HTTP y códigos de estado
* Formato JSON o XML
* Stateless por diseño

#### 🧬 GraphQL APIs:

```graphql
query {
  user(id: "123") {
    name
    email
    posts {
      title
    }
  }
}
```

* Un único endpoint
* El cliente define los datos necesarios
* Introspection revela el esquema
* Control de acceso complejo

#### 📜 SOAP APIs:

```xml
<soap:Envelope>
  <soap:Body>
    <GetUser>
      <UserId>123</UserId>
    </GetUser>
  </soap:Body>
</soap:Envelope>
```

* Basado en XML
* Estructura más rígida
* Seguridad integrada (WS-Security)
* Legacy pero común

#### 🔄 WebSocket APIs:

```javascript
ws://api.example.com/socket
```

* Comunicación bidireccional en tiempo real
* Conexiones persistentes
* Modelo de seguridad distinto a HTTP

***

### 🕵️‍♂️ Técnicas de descubrimiento de APIs

#### 📄 Descubrimiento de documentación:

```
/api
/api/docs
/api-docs
/swagger
/swagger.json
/swagger-ui
/openapi.json
/api/v1
/api/v2
/graphql
/graphiql
/.well-known/openapi.json
```

#### 🧑‍💻 Analizar código cliente:

```javascript
// Buscar llamadas API en JS
fetch('/api/users/' + userId)
axios.post('/api/products', data)
```

#### 🌐 Enumeración de métodos HTTP:

```http
OPTIONS /api/users HTTP/1.1
# Devuelve: Allow: GET, POST, PUT, DELETE, PATCH
```

#### 🔄 Cambiar métodos:

```http
# Original
GET /api/users/123

# Probar otros métodos
POST /api/users/123
PUT /api/users/123
DELETE /api/users/123
PATCH /api/users/123
```

#### 🔢 Descubrimiento de versiones:

```
/api/v1/users
/api/v2/users
/api/v3/users
/v1/api/users
```

#### 💥 Fuzzing de parámetros:

```
/api/users?id=123
/api/users?userId=123
/api/users?user_id=123
/api/users/123
/api/123/users
```

***

### 🚨 Vulnerabilidades comunes en APIs

#### 🧨 Broken Object Level Authorization (BOLA):

```http
# Usuario A accede a datos de usuario B
GET /api/users/123
GET /api/users/456
```

#### 🔺 Broken Function Level Authorization:

```http
DELETE /api/admin/users/123
```

#### 💣 Mass Assignment:

```json
PATCH /api/users/123
{
  "email": "attacker@evil.com",
  "role": "admin",
  "isVerified": true
}
```

#### 📂 Excessive Data Exposure:

```json
{
  "id": 123,
  "username": "john",
  "email": "john@example.com",
  "password_hash": "...",
  "ssn": "123-45-6789",
  "api_key": "..."
}
```

#### ⏱️ Falta de Rate Limiting:

```javascript
for i in range(10000):
    requests.post('/api/login', data=credentials)
```

#### ⚙️ Security Misconfiguration:

```
/api/debug
/api/swagger-ui

Access-Control-Allow-Origin: *
```

***

### 🧬 Server-Side Parameter Pollution

#### 🧪 Query String Pollution:

```
username=admin

# Backend
/internal/api?username=admin&role=user

# Ataque
username=admin%26role=admin%23

# Resultado
/internal/api?username=admin&role=admin#&role=user
```

#### 🌐 REST URL Pollution:

```
# Normal
/api/v1/users/admin/field/email

# Ataque
username=../../v1/users/admin/field/passwordResetToken%23
```

#### ✂️ Truncation Attacks:

```
parameter=value%23ignored
parameter=value%00ignored
parameter=value%0Aignored
```

***

### 💣 Vulnerabilidades de Mass Assignment

#### 🧠 Entendiendo Mass Assignment:

```javascript
app.patch('/api/users/:id', (req, res) => {
  User.update(req.body, {where: {id: req.params.id}});
});
```

```json
{
  "email": "new@email.com",
  "role": "admin",
  "balance": 999999,
  "isVerified": true
}
```

#### 🔍 Cómo encontrarlo:

1. Observar respuestas de la API
2. Añadir campos extra
3. Probar campos privilegiados
4. Verificar si se actualizan

#### 🎯 Campos comunes:

* role
* admin
* isAdmin
* verified
* balance
* credits
* permissions

***

### 🔐 Autenticación y autorización en APIs

#### 🚫 Bypass de autenticación:

```
/api/internal/users
```

#### 🔓 Bypass de autorización:

```http
GET /api/orders/123
GET /api/orders/124
```

#### 🎟️ Manipulación de tokens:

```json
{
  "sub": "user123",
  "role": "user"
}
```

```http
Authorization: Bearer api_key_123
X-API-Key: 123456
```

***

### 🧪 Metodología de testing

#### 1️⃣ Discovery

1. Encontrar documentación
2. Analizar JavaScript
3. Testear rutas comunes
4. Enumerar métodos
5. Buscar versiones

#### 2️⃣ Authentication

1. Probar sin auth
2. Tokens inválidos/expirados
3. Manipulación de tokens
4. Bypass

#### 3️⃣ Authorization

1. IDOR
2. Funciones admin
3. Escalada horizontal
4. Escalada vertical

#### 4️⃣ Input Validation

1. Inyecciones
2. Parameter pollution
3. Datos excesivos
4. Caracteres especiales

#### 5️⃣ Business Logic

1. Mass assignment
2. Manipulación de precios
3. Bypass de workflows
4. Rate limiting

#### 6️⃣ Data Exposure

1. Datos sensibles
2. Errores verbosos
3. Info disclosure
4. GraphQL introspection

***

### 🛠️ Herramientas para API Testing

#### 🔍 Descubrimiento:

* Swagger UI
* Postman
* Burp Spider
* OWASP ZAP
* Amass, ffuf

#### 🧑‍💻 Manual:

* Burp Repeater
* Postman
* cURL
* HTTPie

#### 🤖 Automatizado:

* Burp Scanner
* OWASP ZAP Active Scan
* REST-Attacker
* APIFuzzer

#### 🎯 Especializadas:

* Arjun
* Kiterunner
* GraphQL Voyager
* JWT.io

***

### 🌍 Impacto en el mundo real

#### 📡 T-Mobile (2023):

* Vulnerabilidad API
* 37 millones afectados
* BOLA

#### 🚴 Peloton (2021):

* Datos privados expuestos
* Sin autenticación
* Excessive data exposure

#### 💸 Venmo (2018):

* Historial de transacciones público
* Information disclosure

#### 📱 Facebook (2019):

* Instagram expuso contraseñas
* Mass assignment

#### 🎯 Bug Bounties:

* IDOR muy común
* Mass assignment
* Parameter pollution
* Manipulación de precios

***

### 🛡️ Estrategias de defensa

#### 🔐 Diseño seguro:

```javascript
const allowedFields = ['email', 'name'];
const updates = {};
for (let field of allowedFields) {
  if (req.body[field]) updates[field] = req.body[field];
}
```

```javascript
if (!hasPermission(user, 'delete', resource)) {
  return res.status(403).json({error: 'Forbidden'});
}
```

```javascript
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 100
});
app.use('/api/', limiter);
```

#### 🛑 Autenticación y autorización:

* OAuth 2.0, JWT
* Validar tokens siempre
* RBAC
* No confiar en el cliente

#### 🧪 Validación de entrada:

* Whitelist
* Validar tipos
* Sanitizar
* Queries parametrizadas

#### 📤 Filtrado de salida:

* Solo datos necesarios
* Basado en permisos
* No exponer IDs internos
* Sanitizar errores

#### 📡 Cabeceras de seguridad:

```http
Content-Type: application/json
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
Strict-Transport-Security: max-age=31536000
```

#### 🚪 API Gateway:

* Autenticación centralizada
* Rate limiting
* Validación
* Logging

***

### ✅ Buenas prácticas de seguridad en APIs

#### 🧠 Fase de diseño:

* Threat modeling
* Least privilege
* Security by design
* Revisiones

#### 🧑‍💻 Desarrollo:

* Frameworks seguros
* Validación
* Tests de seguridad
* Code review

#### 🚀 Despliegue:

* Desactivar debug
* Eliminar endpoints de test
* Manejo de errores
* Monitorización

#### 🔄 Mantenimiento:

* Auditorías
* Pentesting
* Actualizar dependencias
* Monitorizar anomalías
