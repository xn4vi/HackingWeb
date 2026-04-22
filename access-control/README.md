---
icon: folder-open
---

# Access Control

### 🔐¿Qué es Access Control?

Access control (también llamado autorización) es la aplicación de restricciones sobre quién o qué está autorizado a realizar acciones o acceder a recursos. En aplicaciones web, esto significa:

* Control de acceso vertical: diferentes tipos de usuarios tienen acceso a distinta funcionalidad (admin vs usuario normal)
* Control de acceso horizontal: los usuarios solo pueden acceder a recursos que les pertenecen (el usuario A no puede ver los datos del usuario B)
* Control de acceso dependiente del contexto: el acceso depende del estado de la aplicación o del flujo (no puedes acceder al paso 3 sin completar el paso 2)

***

### 🚨 Vulnerabilidades comunes de Access Control

#### 🔺 Escalada de privilegios vertical:

* Usuarios normales accediendo a funcionalidad de admin
* Bypass de comprobaciones de roles para obtener privilegios elevados
* Manipulación de identificadores de rol en las peticiones

#### 🔁 Escalada de privilegios horizontal:

* Acceder a datos de otros usuarios modificando identificadores
* Insecure Direct Object References (IDOR)
* IDs de usuario predecibles o fáciles de adivinar

#### ⏭️ Bypass dependiente del contexto:

* Acceder a recursos fuera de secuencia
* Saltarse pasos requeridos en workflows
* Manipular variables de estado

***

### 🧩 Patrones de Broken Access Control

#### 🖥️ Controles en el lado cliente:

```http
// Vulnerable: Rol admin almacenado en cookie
Cookie: Admin=true

// Vulnerable: Rol en campo oculto
<input type="hidden" name="role" value="admin">

// Vulnerable: Rol en JSON
{"email": "user@example.com", "roleid": 2}
```

#### 🌐 Control de acceso basado en parámetros:

```http
// Vulnerable: ID de usuario en URL
/myaccount?id=123

// Vulnerable: Identificadores predecibles
/api/user/456/profile

// Ataque: cambiar parámetro
/myaccount?id=456
```

#### 🚪 Funcionalidad sin protección:

```http
// Vulnerable: Panel admin con URL predecible
/admin
/administrator
/admin-panel

// Vulnerable: URL expuesta en robots.txt o código fuente
Disallow: /admin
```

#### ⚙️ Mala configuración de la plataforma:

```http
// Vulnerable: Diferente control según método HTTP
GET /admin/deleteUser?username=carlos (bloqueado)
POST /admin/deleteUser (permitido)

// Vulnerable: Bypass mediante cabeceras
X-Original-URL: /admin
Referer: https://example.com/admin
```

***

### 🧠 Tipos de vulnerabilidades de Access Control

#### 🧨 Broken Object Level Authorization (BOLA/IDOR):

* Vulnerabilidad más común en APIs
* Referencia directa a objetos sin comprobación de autorización
* Ejemplo: `/api/users/123/transactions` accesible por cualquier usuario autenticado

#### 🔧 Broken Function Level Authorization:

* Falta de controles a nivel de función
* Usuarios acceden a funciones de otros roles
* Ejemplo: usuario normal llamando endpoints de admin

#### 🚫 Missing Access Control:

* Funcionalidad sin checks de autorización
* Basado únicamente en oscuridad (URLs no predecibles)
* Ejemplo: panel admin en `/admin-xyz123` sin autenticación

#### 🔄 Fallos en procesos multi-step:

* Control solo en algunos pasos
* Pasos finales accesibles directamente
* Ejemplo: paso 1 validado, paso 2 bypass

#### 📡 Control basado en cabeceras:

* Autorización basada en headers HTTP
* Los headers pueden ser manipulados
* Ejemplo: `Referer: /admin` concede acceso

***

### 💣 Técnicas de explotación

#### 🔢 Manipulación de parámetros:

```http
# IDOR básico
/user/profile?id=123 → id=456

# Enumeración de GUID
/api/user/a823b-c123-d456

# Referencias indirectas
/document/1 → /document/2
```

#### 🍪 Manipulación de cookies/sesión:

```http
# Manipulación de rol
Admin=false → Admin=true
roleid=1 → roleid=2

# Manipulación JWT
{"role": "user"} → {"role": "admin"}
```

#### 🌐 Manipulación de métodos HTTP:

```http
# Petición original bloqueada
POST /admin/upgrade HTTP/1.1

# Probar otros métodos
GET /admin/upgrade?username=victim HTTP/1.1
PUT /admin/upgrade HTTP/1.1
```

#### 📡 Inyección de cabeceras:

```http
# Bypass con X-Original-URL
GET / HTTP/1.1
X-Original-URL: /admin

# Bypass con X-Forwarded-For
X-Forwarded-For: 127.0.0.1

# Bypass con Referer
Referer: https://example.com/admin
```

#### ⏭️ Bypass de multi-step:

```http
# Saltar al paso final
POST /admin/upgrade-confirm HTTP/1.1
username=victim&confirmed=true
```

***

### 🔍 Cómo encontrar fallos de Access Control

#### 🧑‍💻 Técnicas manuales:

* Mapear niveles de privilegio: identificar roles y accesos
* Testear acceso horizontal: acceder a recursos de otros usuarios
* Testear acceso vertical: acceder a funciones de mayor privilegio
* Probar diferentes métodos HTTP
* Modificar parámetros: IDs, usernames, roles
* Eliminar parámetros: tokens, cookies
* Testear cabeceras: X-Original-URL, Referer, X-Forwarded-For
* Acceso directo: saltar pasos intermedios

#### 🤖 Testing automatizado:

* Burp Suite Autorize
* OWASP ZAP access control testing
* Scripts comparando respuestas privilegiadas vs no privilegiadas

***

### 💥 Impacto de las vulnerabilidades de Access Control

#### 📂 Data breach:

* Acceso a información sensible
* Exposición de datos financieros, PII, historiales médicos
* Violaciones de cumplimiento (GDPR, HIPAA)

#### 📈 Escalada de privilegios:

* Usuario normal obtiene acceso admin
* Toma completa de la aplicación
* Capacidad de modificar/eliminar cualquier dato

#### 💸 Abuso de lógica de negocio:

* Transacciones no autorizadas
* Account takeover
* Daño reputacional

#### 🕸️ Movimiento lateral:

* Acceso a cuentas de otros usuarios
* Extracción masiva de datos
* Ataques dirigidos a cuentas de alto valor

***

### 🌍 Ejemplos reales

#### 🧨 Vulnerabilidades IDOR:

* Facebook (2019): acceso a fotos privadas por IDs predecibles
* Instagram (2020): ver cuentas privadas manipulando parámetros
* T-Mobile (2021): exposición de datos vía IDOR en API

#### 🔺 Escalada de privilegios:

* Uber (2016): panel admin accesible sin autenticación
* GitHub (2020): bypass de control de acceso en repositorios
* PayPal: usuario podía añadirse como admin manipulando parámetros

#### ⚙️ Mala configuración de plataforma:

* Buckets cloud públicos
* Paneles admin descubiertos por enumeración
* APIs sin checks de autorización

***

### 🛡️ Estrategias de mitigación

#### 🧱 Defense in Depth:

* Nunca confiar en controles del cliente
* Implementar checks en cada request
* Lógica de autorización centralizada
* Enfoque "deny by default"

#### 🔐 Principios de diseño seguro:

```python
# Mal: confiar en el cliente
role = request.cookies.get('role')
if role == 'admin':
    allow_access()

# Bien: verificación en servidor
user = get_user_from_session()
if has_permission(user, 'admin_access'):
    allow_access()
```

#### ✅ Checks de autorización adecuados:

* Verificar identidad en cada request
* Comprobar permisos sobre recursos específicos
* Validar que el usuario es dueño del recurso
* No exponer IDs internos directamente

#### 🔗 Referencias indirectas:

```python
# Mal
/api/document/1234

# Mejor
/api/document/my-document-slug
```

#### 🔒 Principio de mínimo privilegio:

* Permisos mínimos necesarios
* Privilegios elevados temporales
* Auditorías periódicas

#### 📊 Rate limiting y monitorización:

* Detectar enumeración
* Alertar sobre escaladas de privilegio
* Loggear decisiones de autorización

***

### 🧪 Metodología de testing

#### 📋 Enfoque paso a paso:

* Reconocimiento: mapear funcionalidades y roles
* Autenticación: probar distintos usuarios
* Testing horizontal
* Testing vertical
* Testing de métodos HTTP
* Testing de parámetros
* Testing de cabeceras
* Testing de workflows

#### 💡 Casos comunes:

* ¿Puede el usuario A acceder al perfil del usuario B?
* ¿Puede un usuario normal acceder al panel admin?
* ¿Puede modificar su rol?
* ¿Puede saltarse el pago en checkout?
* ¿Cambiar el método HTTP hace bypass?
* ¿Funciona X-Original-URL?
