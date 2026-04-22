# 📂 Path Traversal

**Path Traversal** es una vulnerabilidad web que permite a un atacante leer archivos arbitrarios del servidor manipulando rutas de archivos.

Mediante secuencias especiales como `../` (dot-dot-slash), el atacante puede salir del directorio previsto y acceder a archivos sensibles como configuraciones, credenciales o archivos del sistema.

***

## ⚙️ Cómo funciona

Cuando una aplicación usa input del usuario para construir rutas sin validación adecuada:

```bash
Ruta esperada: /var/www/images/image.png
Ataque:        /var/www/images/../../../etc/passwd
Resultado:     /etc/passwd
```

Las secuencias `../` permiten subir en el árbol de directorios y acceder a rutas fuera del entorno permitido.

***

## 🎯 Objetivos comunes

### 🐧 Sistemas Unix/Linux

* `/etc/passwd` → información de usuarios
* `/etc/shadow` → hashes de contraseñas (requiere privilegios)
* `/etc/hosts` → resolución DNS local
* `/var/log/` → logs del sistema y aplicaciones
* `~/.ssh/id_rsa` → claves privadas SSH

### 🪟 Sistemas Windows

* `C:\Windows\System32\drivers\etc\hosts` → DNS
* `C:\Windows\win.ini` → configuración
* `C:\boot.ini` → configuración de arranque

***

## 🧪 Técnicas de bypass

### ➡️ Traversal básico

```
../../../../etc/passwd
```

### ➡️ Rutas absolutas

```
/etc/passwd
```

### ➡️ Bypass de filtros no recursivos

```
....//....//etc/passwd
```

(Tras limpiar una vez → `../../etc/passwd`)

### ➡️ URL Encoding

```
..%2f..%2fetc/passwd
..%252f..%252fetc/passwd
```

### ➡️ Prefijo de ruta

```
/var/www/images/../../../etc/passwd
```

### ➡️ Inyección de byte nulo

```
../../../../etc/passwd%00.png
```

### ➡️ Encoding Unicode / UTF-8

```
..%c0%af..%c0%afetc/passwd
```

***

## ❌ Por qué ocurre

* Validación insuficiente del input
* Uso de listas negras débiles (solo bloquean `../`)
* Sanitización no recursiva
* Validación solo de extensión de archivo
* Falta de normalización de rutas (canonicalización)

***

## 💥 Impacto

* Exposición de datos sensibles (configuración, credenciales, API keys)
* Acceso al código fuente
* Robo de credenciales (hashes, claves SSH, tokens)
* Reconocimiento del sistema
* Escalada de privilegios
* En algunos casos, escritura arbitraria de archivos

***

## 🔍 Métodos de detección

### 🧑‍💻 Manual

* Buscar parámetros como: `?file=`, `?path=`, `?filename=`
* Probar `../`, `..\\`, rutas absolutas
* Usar variantes codificadas
* Analizar respuestas del servidor

### 🤖 Automatizado

* Burp Suite Intruder con wordlists de traversal
* Herramientas como:
  * dotdotpwn
  * Path Traversal Scanner
* Fuzzing con rutas conocidas del sistema

***

## 🛡️ Mitigación

### ✅ Validación de entrada

* Usar listas blancas de archivos permitidos
* Bloquear secuencias de traversal
* Usar identificadores en lugar de nombres de archivo

### 🧭 Canonicalización de rutas

* Resolver la ruta absoluta antes de usarla
* Verificar que permanece dentro del directorio permitido

Funciones típicas:

* PHP: `realpath()`
* .NET: `Path.GetFullPath()`

### 🔒 Sandboxing

* Usar `chroot` o contenedores
* Ejecutar con permisos mínimos

### 🧱 Uso de frameworks

* Utilizar funciones seguras para servir archivos
* Evitar acceso directo al sistema de archivos

### 🛡️ Defensa en profundidad

* Varias capas de validación
* Logging y monitorización
* Auditorías y pentesting periódicos

***

## 🌍 Ejemplos reales

* **CVE-2019-11510**: Pulse Secure VPN → robo de credenciales
* **CVE-2021-41773**: Apache → Path Traversal + RCE
* **CVE-2018-1000861**: Jenkins plugin vulnerable
* CMS como WordPress o Joomla (especialmente plugins)
