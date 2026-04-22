---
icon: folder-open
---

# Insecure deserialization

## 🔍 ¿Qué es Serialization y Deserialization?

Serialization es el proceso de convertir estructuras de datos complejas (objetos) en un formato plano que puede almacenarse o transmitirse. Deserialization es el proceso inverso: reconstruir el objeto a partir de su forma serializada.

#### ➡️ Formatos comunes de serialización:

* PHP: `serialize()` / `unserialize()`
* Java: `ObjectInputStream` / `ObjectOutputStream`
* Ruby: `Marshal.dump` / `Marshal.load`
* Python: módulo `pickle`
* JSON: formato de texto agnóstico al lenguaje
* .NET: `BinaryFormatter`, `DataContractSerializer`

#### 📍 Ejemplo de serialización en PHP:

```php
// Objeto
$user = new User('admin', true);

// Serializado
O:4:"User":2:{s:8:"username";s:5:"admin";s:5:"admin";b:1;}

// Desglose del formato
O:4:"User"      // Objeto de clase "User" (4 caracteres)
2:              // 2 propiedades
s:8:"username"  // Propiedad string "username" (8 chars)
s:5:"admin"     // Valor "admin" (5 chars)
s:5:"admin"     // Propiedad "admin"
b:1             // Booleano true
```

***

### ⚠️ ¿Por qué es peligrosa la Deserialización insegura?

Cuando las aplicaciones deserializan datos controlados por el usuario sin validación adecuada, los atacantes pueden:

#### 💀 Ejecución remota de código:

* Inyectar objetos maliciosos que ejecutan código durante la deserialización
* Explotar “magic methods” que se ejecutan automáticamente (PHP `__wakeup`, Java `readObject`)
* Encadenar clases existentes para construir exploits (“gadget chains”)

#### 👑 Escalada de privilegios:

* Modificar roles o permisos serializados
* Cambiar flags de admin en cookies de sesión
* Bypass de autenticación manipulando tokens

#### 🔄 Manipulación de datos:

* Alterar la lógica de la aplicación mediante propiedades del objeto
* Explotar lógica de negocio con objetos manipulados
* Activar funcionalidades no previstas

***

### 🧪 Serialización en diferentes lenguajes

#### 🐘 Serialización en PHP:

```php
// Objeto serializado
O:4:"User":2:{s:8:"username";s:6:"wiener";s:5:"admin";b:0;}

// Tipos de datos
s:5:"hello"   // String
i:42          // Integer
b:1           // Boolean (true)
b:0           // Boolean (false)
a:2:{...}     // Array con 2 elementos
O:4:"User"    // Objeto de clase User
```

#### ☕ Serialización en Java:

```java
// Formato binario con magic bytes
AC ED 00 05  // Número mágico de serialización Java

// Codificado en Base64 comienza con
rO0AB...     // Identifica objeto serializado Java
```

#### 💎 Serialización en Ruby (Marshal):

```ruby
// Formato binario
Marshal.dump(obj)
Marshal.load(data)

// Común en sesiones de Ruby on Rails
```

***

### 🛡️ Vulnerabilidades comunes

#### ✨ Explotación de Magic Methods:

Los métodos mágicos en PHP se ejecutan automáticamente durante la deserialización:

```php
__construct()   // Creación del objeto
__destruct()    // Destrucción del objeto
__wakeup()      // Durante unserialize()
__toString()    // Cuando se trata como string
__call()        // Llamada a métodos inaccesibles
```

**Ejemplo vulnerable:**

```php
class Logger {
    private $logfile;
    
    function __destruct() {
        // Se ejecuta al destruir el objeto
        unlink($this->logfile);  // Borra archivo
    }
}

// El atacante establece logfile a /etc/passwd
// Cuando el objeto se deserializa y destruye, se borra el archivo
```

#### 📥 Explotación de readObject() en Java:

```java
private void readObject(ObjectInputStream in) {
    in.defaultReadObject();
    // Lógica personalizada de deserialización
    // Puede ejecutar código u operaciones peligrosas
}
```

#### 🍪 Manipulación de cookies:

```
// Cookie de sesión
session=TzoxOiJVc2VyIjoyOntz...

// Decodificar
O:4:"User":2:{s:4:"role";s:4:"user"}

// Modificar a admin
O:4:"User":2:{s:4:"role";s:5:"admin"}

// Re-encodear y usar
```

#### 🔀 Type Juggling (PHP):

```php
// Comparación débil
if ($token == $stored_token) {
    grant_access();
}

// Si stored_token es string, usar token = 0
// 0 == "cualquier_string" evalúa a true
```

***

### 🔗 Gadget Chains

Las gadget chains enlazan clases existentes para lograr ejecución de código sin llamar directamente a funciones peligrosas.

#### 📋 Concepto:

```
Input del atacante → Clase A → Clase B → Clase C → Función peligrosa
```

#### 🔄 Flujo de ejemplo:

* Deserializar objeto malicioso
* `__wakeup()` llama método en una propiedad
* Esa propiedad es otro objeto con `__toString()`
* `__toString()` dispara método en otro objeto
* El objeto final ejecuta `eval()`, `system()`, etc.

#### 📍 Librerías populares:

* ysoserial: gadget chains en Java
* PHPGGC: gadget chains en PHP
* Apache Commons Collections (Java)
* Symfony Framework (PHP)
* Ruby Universal Gadget Chain

***

### ⚔️ Técnicas de explotación

#### 🍪 Manipulación básica de cookies:

```
# Decodificar cookie
echo "BASE64_COOKIE" | base64 -d

# Modificar objeto
# admin=false → admin=true

# Re-encodear
echo "MODIFIED_OBJECT" | base64
```

#### 🛠️ Usando ysoserial (Java):

```shellscript
# Generar payload malicioso
java -jar ysoserial.jar CommonsCollections4 'rm /tmp/file' | base64

# Reemplazar cookie de sesión
```

#### 🛠️ Usando PHPGGC (PHP):

```bash
# Generar gadget chain Symfony
phpggc Symfony/RCE4 exec 'whoami' | base64

# Firmar si es necesario
php -r '$obj="..."; $key="secret"; echo hash_hmac("sha1", $obj, $key);'
```

#### 🧩 Desarrollo de gadget chain custom:

```php
// Analizar código fuente filtrado
// Identificar magic methods
// Encontrar cadenas hacia funciones peligrosas
// Construir objeto serializado malicioso
```

#### 📦 Deserialización PHAR:

```php
// PHAR contiene metadatos serializados
phar://path/to/file.phar

// Operaciones de archivo disparan deserialización
file_exists('phar://uploads/image.jpg')
// Aunque sea jpg, se deserializa el metadata PHAR
```

***

### 🎯 Patrones de ataque

#### 🔎 Encontrar datos serializados:

* Cookies de sesión
* Parámetros API
* Subida de archivos (PHAR)
* Campos ocultos
* Base de datos
* Caché

#### 📍 Indicadores de serialización:

```
# PHP
O:4:"User":2:{...}
a:3:{i:0;s:5:"admin"...}

# Java (Base64)
rO0ABXNy...
AC ED 00 05

# Ruby
BAhvOg...
```

#### 📋 Metodología de testing:

* Identificar datos serializados
* Decodificar y analizar formato
* Probar modificaciones simples
* Testear magic methods
* Buscar gadget chains
* Crear gadget chain custom si hay código

***

### 📖 Ejemplos reales

#### ➡️ Equifax (2017):

* Vulnerabilidad en Apache Struts
* Fallo de deserialización en plugin REST
* Brecha masiva de datos

#### ➡️ Jenkins RCE:

* Vulnerabilidades de deserialización en Java
* Explotable vía CLI
* Gadget chain CommonsCollections

#### ➡️ Ruby on Rails:

* CVE-2013-0156 (XML)
* CVE-2013-0333 (JSON)
* Permitían RCE

#### ➡️ WordPress plugins:

* Fallos en `unserialize()`
* Frecuentes en sistemas de licencias
* Problemas en manejo de sesiones

***

### 🛡️ Prevención y mitigación

#### 🚫 Evitar deserializar datos no confiables:

```php
// Mal
$user = unserialize($_COOKIE['session']);

// Mejor
$user = json_decode($_COOKIE['session'], true);

// Óptimo
$user = verify_and_decrypt_session($_COOKIE['session']);
```

#### 🔐 Comprobación de integridad:

```php
$data = serialize($obj);
$signature = hash_hmac('sha256', $data, $secret_key);
$cookie = base64_encode($data . '|' . $signature);

// Verificar antes de deserializar
list($data, $sig) = explode('|', base64_decode($cookie));
if (hash_hmac('sha256', $data, $secret_key) === $sig) {
    $obj = unserialize($data);
}
```

#### 📛 Restringir clases:

```php
$allowed = ['User', 'Session'];
unserialize($data, ['allowed_classes' => $allowed]);
```

#### 🔄 Alternativas seguras:

* JSON en lugar de serialización nativa
* Protobuf
* MessagePack
* Evitar `pickle`, `Marshal`, etc con input del usuario

#### ✅ Validación de entrada:

* Validar tipos de objetos
* Verificar valores de propiedades
* Añadir checks de autorización
* No confiar solo en el estado del objeto

***

### 🕵️ Detección y defensa

#### 📊 Monitorización:

* Loggear deserializaciones
* Alertar tipos de objetos inesperados
* Detectar gadget chains
* Monitorizar errores

#### 🛠️ WAF:

* Bloquear payloads conocidos
* Detectar magic bytes
* Rate limit
* Detectar patrones de gadget chains

#### ✅ Checklist de code review:

* ¿Se deserializa input del usuario?
* ¿Hay magic methods peligrosos?
* ¿Propiedades controlables por atacante?
* ¿Existen gadget chains?
* ¿Hay verificación de integridad?
* ¿Clases whitelisteadas?

#### 🔒 Cabeceras de seguridad:

```http
Content-Security-Policy: default-src 'self'
X-Content-Type-Options: nosniff

Set-Cookie: session=...; HttpOnly; Secure; SameSite=Strict
```
