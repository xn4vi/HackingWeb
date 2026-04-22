# 🔌 WebSockets

WebSockets permiten comunicación full-duplex entre cliente y servidor, pero introducen vectores de ataque únicos cuando no están correctamente asegurados.

## ⚙️ Cómo funcionan los WebSockets

* Handshake de upgrade HTTP a `ws://` o `wss://`
* Conexión bidireccional persistente
* Mensajes enviados sin overhead de HTTP
* Sin protección automática contra CSRF como en peticiones tradicionales

***

## 🚨 Vulnerabilidades comunes

* Manipulación de mensajes: interceptar/modificar mensajes WebSocket
* Cross-Site WebSocket Hijacking (CSWSH): similar a CSRF pero para handshakes WebSocket
* Bypass de validación de entrada: filtros en HTTP pero no en mensajes WebSocket
* Bypass de autenticación: autenticación débil en el handshake

***

## 🛡️ Mitigación

* Usar tokens CSRF en el handshake
* Validar el origen de los mensajes
* Implementar autenticación/autorización adecuada
* Sanear todo el contenido de los mensajes WebSocket
* Usar `wss://` (WebSockets cifrados)
