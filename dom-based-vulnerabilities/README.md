# 🔍 DOM-Based Vulnerabilities

Las vulnerabilidades basadas en DOM ocurren cuando JavaScript del lado cliente procesa datos no confiables de forma insegura, modificando el DOM de maneras que ejecutan código controlado por el atacante.

***

## 📥 Sources (controladas por el atacante):

* `window.location` (parámetros URL, hash, pathname)
* `document.referrer`
* `document.cookie`
* Mensajes web (`postMessage`)
* Web storage (`localStorage`, `sessionStorage`)

***

## 🪝 Sinks (funciones peligrosas):

* `innerHTML`, `outerHTML`
* `document.write()`
* `eval()`, `setTimeout()`, `setInterval()`
* `location`, `location.href`
* `element.src`, `element.setAttribute()`

***

## ⚠️ Tipos comunes:

* DOM XSS mediante web messaging
* Open redirection a través de parámetros URL
* Manipulación de cookies que lleva a XSS
* DOM clobbering (sobrescribir variables JavaScript con elementos HTML)

***

## 🛡️ Mitigación:

* Validar y sanear toda entrada no confiable
* Usar `textContent` en lugar de `innerHTML`
* Verificar el origen de los mensajes en handlers de `postMessage`
* Evitar usar sinks peligrosos con input del usuario
* Usar Content Security Policy
