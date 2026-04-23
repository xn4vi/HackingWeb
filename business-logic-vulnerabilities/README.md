# Business logic vulnerabilities

## 🔍 ¿Qué son las vulnerabilidades de lógica de negocio?

Las **vulnerabilidades de lógica de negocio** son fallos en el diseño o implementación de una aplicación que permiten a un atacante abusar de funcionalidades legítimas.

A diferencia de vulnerabilidades técnicas (SQLi, XSS), estas se basan en **suposiciones incorrectas sobre el comportamiento del usuario**.

***

## ⚠️ Por qué son peligrosas

* Difíciles de detectar (no las encuentran herramientas automáticas)
* Difíciles de prevenir (el código puede estar "bien" técnicamente)
* Dependientes del contexto
* Alto impacto (financiero, acceso indebido, manipulación de datos)
* Frecuentemente ignoradas

***

## 🎯 Tipos comunes de fallos

### 👥 Confianza en el cliente

* Manipulación de precios en formularios
* Modificación de cantidades
* Cambio de roles en cookies

### ✅ Validación insuficiente

* Cantidades negativas
* Sin límites máximos
* Estados inválidos

### 🔄 Flujos de estado defectuosos

* Saltarse pasos obligatorios
* Acceder a endpoints fuera de orden
* Eliminar peticiones intermedias

### ⚖️ Controles inconsistentes

* Validación distinta según endpoint
* Autorizaciones incoherentes

### 📐 Overflow / Underflow

* Precios negativos por overflow
* Límites superados

### ⚡ Race conditions

* Peticiones simultáneas
* Doble gasto

### 💰 Lógica de cupones

* Acumulación indebida
* Reutilización
* Aplicación múltiple

***

## 📌 Ejemplos reales

### 🛒 E-commerce

* Cambiar precios
* Cantidades negativas → dinero
* Overflow en carrito
* Abuso de cupones

### 🔑 Control de acceso

* Escalar a admin
* Manipular parámetros

### 🔐 Autenticación

* Eliminar validaciones
* Saltar pasos

### 💰 Sistemas financieros

* Errores de redondeo
* Repetición de transacciones
* Manipulación de balances

***

## 🔎 Metodología de detección

### 🗺️ Entender la aplicación

* Mapear flujos
* Identificar supuestos
* Documentar validaciones

### 📏 Probar límites

* Valores mínimos/máximos
* Negativos
* Cero
* Valores grandes

### 🔀 Manipular flujos

* Saltar pasos
* Cambiar orden
* Repetir acciones

### 🛠️ Manipular parámetros

* Precio
* Cantidad
* IDs
* Roles

### 🔁 Testear estados

* Iniciar en pasos intermedios
* Usar tokens antiguos

***

## 🧩 Patrones vulnerables

### 👥 Confianza en cliente

```json
{
  "product": "laptop",
  "price": 999
}
```

### ✅ Validación insuficiente

```javascript
total = price * quantity  // Puede ser negativo
```

### 🔄 Falta de control de flujo

```
/checkout/select → /payment → /confirm
```

Ataque: ir directo a `/confirm`.

### ⚖️ Validación inconsistente

* Registro restringido
* Update sin restricciones
* Admin basado en email

***

## ⚔️ Técnicas de explotación

### 💥 Integer overflow

* Forzar valores extremos
* Convertir precios en negativos

### 🎟️ Coupon stacking

* Combinar cupones
* Automatizar

### ⏭️ Bypass de workflow

* Saltar validaciones
* Acceso directo a endpoints

### 🔐 Abuso criptográfico

* Uso de errores como oracle
* Modificación de tokens

### 🔣 Problemas de parsing

* Emails mal interpretados
* Encoding extraño

***

## 💥 Impacto

### 💰 Financiero

* Pérdidas económicas
* Fraude
* Abuso de descuentos

### 🔑 Control de acceso

* Acceso admin
* Escalada de privilegios

### 📊 Integridad de datos

* Manipulación de inventario
* Fraude en pedidos

### 📉 Reputación

* Pérdida de confianza
* Problemas legales

***

## 🛡️ Mitigación

### 🖥️ Validación en servidor

* No confiar en el cliente
* Recalcular todo

### 🔁 Control de estado

* Validar cada paso
* No permitir saltos

### 🧪 Testing

* Revisiones manuales
* Threat modeling
* Edge cases

### 🚧 Restricciones de entrada

* Límites claros
* Tipos adecuados

### ⚛️ Atomicidad

* Transacciones
* Locks

### 🔒 Mínimo privilegio

* Permisos restrictivos
* Verificación constante

### 🛡️ Defensa en profundidad

* Logs
* Rate limiting
* Alertas

***

## 📖 Casos reales

* Steam (2015): cantidades negativas → crédito
* Amazon (2014): manipulación de precios
* E-commerce: overflow en carrito
* Apps bancarias: race conditions
* Apps de transporte: abuso de referidos
