<div align="center">

# 🕷️ HackingWeb

**Guía práctica de vulnerabilidades web — todo en español**

[![Labs](https://img.shields.io/badge/Labs-150%2B-00d4ff?style=flat-square&logo=hackthebox&logoColor=white)](https://github.com/xn4vi/HackingWeb)
[![Categorías](https://img.shields.io/badge/Categorías-28-00ff9d?style=flat-square)](https://github.com/xn4vi/HackingWeb)
[![Idioma](https://img.shields.io/badge/Idioma-Español-ff3e6c?style=flat-square)](https://github.com/xn4vi/HackingWeb)
[![Licencia](https://img.shields.io/badge/Uso-Educativo-ffb800?style=flat-square)](https://github.com/xn4vi/HackingWeb)

Repositorio de soluciones y writeups para laboratorios de seguridad web.<br>
Cada categoría incluye explicación teórica, payloads y resolución paso a paso.

[📚 Ver Categorías](#-categorías) · [✅ Checklist Web](./Checklist%20Web.md) · [⭐ Dar estrella](https://github.com/xn4vi/HackingWeb/stargazers)

</div>

---

## 📂 Categorías

### 🔴 Injection

| Categoría | Labs | Descripción |
|-----------|:----:|-------------|
| [💉 SQL Injection](./sql-injection/) | 16 | UNION attacks, Blind SQLi, OOB, bypass de login y enumeración de BBDD |
| [🖥️ XSS — Cross-Site Scripting](./cross-site-scripting-xss/) | 16 | Reflejado, almacenado, DOM-based, robo de cookies y bypass de filtros |
| [💻 OS Command Injection](./os-command-injection/) | 5 | Inyección simple, blind con delays, redirección de salida y OOB |
| [📄 XXE Injection](./xml-external-entity-xxe-injection/) | 9 | Entidades externas, blind OOB, exfiltración vía DTD y XInclude |
| [🧩 SSTI — Template Injection](./server-side-template-injection-ssti/) | 6 | ERB, Tornado, Freemarker, lenguaje desconocido y sandbox escape |
| [🗄️ NoSQL Injection](./nosql-injection/) | 4 | Bypass de auth y operator injection en MongoDB |

---

### 🟠 Autenticación y Control de Acceso

| Categoría | Labs | Descripción |
|-----------|:----:|-------------|
| [🔐 Authentication](./authentication/) | 13 | Bypass 2FA, enumeración, fuerza bruta y password reset poisoning |
| [🔑 Access Control](./access-control/) | 11 | IDOR, bypass por URL/HTTP, filtración de contraseñas y escalación de privilegios |
| [🪙 JWT Attacks](./jwt/) | 6 | Algoritmo none, firma débil, bypass de validación y ataques JWK |
| [🔗 OAuth Authentication](./oauth-authentication/) | 6 | Robo de tokens, open redirect, CSRF en flujos y implicit grant |

---

### 🟡 Vulnerabilidades del Lado Servidor

| Categoría | Labs | Descripción |
|-----------|:----:|-------------|
| [🌐 SSRF](./server-side-request-forgery-ssrf/) | 4 | Básico local/interno, blind OOB y bypass de filtros blacklist |
| [📦 Insecure Deserialization](./insecure-deserialization/) | 9 | PHP, Java, gadget chains y explotación de métodos mágicos |
| [📁 File Upload Vulnerabilities](./file-upload-vulnerabilities/) | 6 | Web shell, bypass content-type, path traversal en uploads |
| [🧠 Business Logic Vulnerabilities](./business-logic-vulnerabilities/) | 10 | Confianza en cliente, reglas rotas, máquinas de estado y dinero infinito |

---

### 🟢 Cache y Smuggling

| Categoría | Labs | Descripción |
|-----------|:----:|-------------|
| [☠️ HTTP Request Smuggling](./http-request-smuggling/) | 11 | CL.TE, TE.CL, TE.TE obfuscation y bypass de controles frontend |
| [🧪 Web Cache Poisoning](./web-cache-poisoning/) | 8 | Headers no indexados, cookies, parameter cloaking y Fat GET |
| [🎭 Web Cache Deception](./web-cache-deception/) | 5 | Manipulación de rutas, delimitadores, normalización y CSRF |

---

### 🔵 Client-Side

| Categoría | Labs | Descripción |
|-----------|:----:|-------------|
| [🖱️ Clickjacking](./clickjacking/) | 4 | Con CSRF token, precargado, bypass anti-frame y multi-step |
| [🔄 CSRF](./cross-site-request-forgery-csrf/) | 7 | Sin defensa, tokens duplicados, SameSite Lax y Referer bypass |
| [🌍 CORS Misconfiguration](./cross-origin-resource-sharing-cors/) | 3 | Básico con credenciales, null origin y protocolo inseguro |
| [🏠 HTTP Host Header Attacks](./http-host-header-attacks/) | 6 | Inyección en Host, password reset y bypass de autenticación |
| [🔗 DOM-Based Vulnerabilities](./dom-based-vulnerabilities/) | 4 | XSS via web messages, JSON.parse y open redirect en DOM |
| [☣️ Prototype Pollution](./prototype-pollution/) | 3 | Client-side via APIs, XSS en DOM y vectores alternativos |
| [🔌 WebSocket Attacks](./websockets/) | 3 | Manipulación de handshake, CSWSH y bypass de filtros XSS |

---

### 🟣 Avanzado

| Categoría | Labs | Descripción |
|-----------|:----:|-------------|
| [⚡ Race Conditions](./race-conditions/) | 5 | Bypass de rate limiting, límites de descuento y tokens de reset |
| [🔷 GraphQL Vulnerabilities](./graphql-api-vulnerabilities/) | 5 | Introspección, bypass de auth, batching y SSRF via GraphQL |
| [🛠️ API Testing](./api-testing/) | 5 | Documentación expuesta, parameter pollution y mass assignment |
| [🤖 Web LLM Attacks](./web-llm-attacks/) | 4 | Agencia excesiva, explotación de APIs y prompt injection |
| [🔀 Path Traversal](./path-traversal/) | 6 | Simple, absoluto, doble codificación y null byte |

---

## ✅ Checklist Web

Una checklist exhaustiva para pentesting web profesional que cubre todos los vectores de ataque.

➡️ **[Ver Checklist completa](./Checklist%20Web.md)**

---

## 🚀 Cómo usar este repositorio

1. **Elige una categoría** de la tabla de arriba
2. **Lee el README** de cada categoría para entender la vulnerabilidad
3. **Sigue los laboratorios** numerados en orden de dificultad
4. Cada lab incluye: contexto, payload, pasos y explicación

---

## ⚠️ Aviso legal

> Este repositorio es únicamente con fines **educativos**. Toda la información está orientada a entender y practicar seguridad web en entornos controlados. No uses estas técnicas en sistemas sin autorización explícita.

---

<div align="center">

Hecho con 🧠 por [xn4vi](https://github.com/xn4vi) · Si te es útil, dale una ⭐

</div>
