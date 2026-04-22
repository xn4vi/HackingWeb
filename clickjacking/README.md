---
icon: folder-open
---

# Clickjacking

## 🎭 Clickjacking

Clickjacking (UI redressing) engaña a los usuarios para que hagan clic en algo diferente de lo que perciben, superponiendo elementos invisibles u opacos sobre componentes legítimos de la interfaz.

***

### ⚙️ Cómo funciona:

* El atacante carga el sitio objetivo en un iframe transparente
* Coloca contenido señuelo (botones, texto) en coordenadas exactas sobre los botones reales
* El usuario cree que está haciendo clic en el señuelo, pero en realidad hace clic en el iframe oculto

***

### 💣 Técnicas comunes:

* Superposición básica de iframe con posicionamiento CSS
* Prellenado de parámetros de formularios mediante URL
* Bypass de frame-busting con `sandbox="allow-forms"`
* Ataques multistep que requieren múltiples clics
* Encadenamiento con DOM XSS para impacto combinado

***

### 🛡️ Mitigación:

* Cabecera `X-Frame-Options: DENY`
* `Content-Security-Policy: frame-ancestors 'none'`
* JavaScript de frame-busting (puede ser bypassed)
* Cookies SameSite
