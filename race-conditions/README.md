---
icon: folder-open
---

# Race conditions

### 🔍 ¿Qué son las Race Conditions?

Las **Race Conditions** ocurren cuando el resultado de un proceso depende del tiempo o del orden de los eventos, especialmente cuando múltiples hilos o procesos acceden a recursos compartidos sin la sincronización adecuada. En aplicaciones web, esto suele manifestarse cuando:

* Se procesan múltiples solicitudes simultáneamente
* Las comprobaciones de validación no han finalizado antes de operaciones posteriores
* El estado no está correctamente bloqueado durante procesos de múltiples pasos
* Existen ventanas de tiempo entre operaciones de comprobación y uso

La característica clave es que la vulnerabilidad solo aparece cuando las operaciones ocurren en una secuencia de tiempo específica, lo que las hace difíciles de reproducir y explotar de forma fiable.

***

### 🧬 Tipos de Race Conditions

#### ⚔️ Limit Overrun:

* Aplicar el mismo código de descuento múltiples veces simultáneamente
* Superar límites de recursos (créditos, intentos, cuotas)
* Saltarse restricciones de “uso único”

#### 🚫 Rate Limit Bypass:

* Enviar múltiples intentos de login en paralelo
* Evitar mecanismos de limitación
* Saltarse temporizadores de bloqueo

#### 🔄 Multi-Endpoint Races:

* Explotar el timing entre diferentes endpoints
* Añadir elementos durante el proceso de checkout
* Cambios de estado a través de múltiples peticiones

#### 📍 Single-Endpoint Races:

* Múltiples solicitudes al mismo endpoint con datos distintos
* Race conditions en verificación de email
* Operaciones de actualización sin bloqueo suficiente

#### ⏱️ Time-Sensitive Operations:

* Generación de tokens basada en timestamp
* Tokens predecibles en ventanas de tiempo estrechas
* Race conditions en creación de sesión

#### 🧩 Partial Construction:

* Acceder a objetos antes de que se complete su inicialización
* Explotar estados nulos o indefinidos

***

### ⚡ Cómo funcionan las Race Conditions

#### 🔄 Patrón clásico Check-Then-Use:

```python
if user.balance >= price:
    # Ventana de race aquí
    user.balance -= price
    process_purchase()
```

#### ❓ Qué ocurre:

* Request 1: Comprueba balance (OK)
* Request 2: Comprueba balance (OK)
* Request 1: Deduce balance
* Request 2: Deduce balance → el balance se vuelve negativo

#### ✅ Sincronización correcta:

```python
with transaction_lock:
    if user.balance >= price:
        user.balance -= price
        process_purchase()
```

***

### 🎯 Patrones vulnerables comunes

#### 💸 Aplicación de códigos de descuento:

Flujo normal:

1. Comprobar si el código ha sido usado → No
2. Aplicar descuento
3. Marcar código como usado

Race condition:

* Request 1: Comprueba → No usado
* Request 2: Comprueba → No usado
* Request 1: Aplica descuento
* Request 2: Aplica descuento

#### ⏳ Rate Limiting:

Flujo normal:

1. Comprobar contador < límite
2. Procesar petición
3. Incrementar contador

Race:

* Múltiples peticiones pasan el check antes del incremento

#### 📧 Verificación de email:

* Dos cambios simultáneos
* Tokens inconsistentes

***

### 🛠️ Técnicas de explotación

#### 📦 Single-Packet Attack (HTTP/2):

* Enviar todas las peticiones en un solo paquete TCP
* Llegan simultáneamente

#### 🧰 Burp Suite:

1. Crear grupo de requests
2. Enviar en paralelo
3. Activar HTTP/2
4. concurrentConnections=1

#### ⚡ Turbo Intruder:

```python
engine = RequestEngine(...)
engine.queue(...)
engine.openGate('race')
```

#### 🚀 Optimización:

* Minimizar latencia
* Usar conexiones rápidas
* Reducir tamaño de request

***

### 🔎 Métodos de detección

#### ⚠️ Señales:

* Operaciones de una sola ejecución
* Tokens de un solo uso
* Sistemas de balance

#### 🧠 Enfoque:

1. Identificar operaciones críticas
2. Enviar múltiples requests
3. Comprobar si:
   * Se exceden límites
   * Se reutilizan códigos
   * Ocurren efectos múltiples

#### 🤖 Automatización:

* Burp Repeater
* Turbo Intruder
* Scripts con threading

***

### 🌐 Escenarios reales

#### 🛒 E-commerce:

* Aplicar cupones múltiples veces
* Comprar sin saldo suficiente

#### 🔑 Autenticación:

* Bypass de intentos de login
* Fuerza bruta en paralelo

#### 💰 Sistemas financieros:

* Doble gasto
* Manipulación de saldo

#### 📝 Registro:

* Saltar verificación
* Crear cuentas sin validación

***

### 💥 Impacto

#### ☠️ Crítico:

* Pérdida financiera
* Bypass de autenticación
* Toma de cuentas

#### 🔥 Alto:

* Bypass de rate limiting
* Transacciones no autorizadas

#### ⚠️ Medio:

* Reutilización de códigos
* Disrupción de flujos

***

### 📖 Ejemplos reales

* Starbucks: generación infinita de dinero
* E-commerce: cupones duplicados
* Bancos: doble gasto
* Bug bounty: bypass de rate limit

***

### 🛡️ Mecanismos de defensa

#### 🗄️ Base de datos:

```sql
SELECT * FROM accounts FOR UPDATE;
```

#### 📱 Aplicación:

```python
select_for_update()
```

#### 🔒 Locking optimista:

* Uso de versiones
* Detección de cambios concurrentes

#### 📊 Gestión de estado:

* Operaciones atómicas
* Evitar check-then-use

#### 🛑 Rate limiting:

```python
redis.incr()
```

#### 🔑 Tokens:

* Aleatorios
* Alta entropía
* No basados solo en tiempo

***

### 📋 Metodología de testeo

#### 🎯 Paso a paso:

1. Identificar:
   * Balance
   * Cupones
   * Tokens
2. Preparar:
   * Capturar request
   * Configurar Burp
3. Ejecutar:
   * 10–20 requests simultáneas
   * Usar HTTP/2
4. Verificar:
   * Límites superados
   * Tokens reutilizados
   * Estado inconsistente

#### ❌ Problemas comunes:

* Pocas requests
* No usar HTTP/2
* Latencia alta

***

### 🚀 Explotación avanzada

#### 🔗 Multi-stage:

* Encadenar múltiples races

#### 🧱 Construcción parcial:

* Acceder antes de inicialización

#### ⏰ Predicción temporal:

* Tokens basados en tiempo

#### ⚔️ Encadenamiento:

* Race + IDOR
* Race + CSRF
* Race + lógica de negocio

***

### 🧰 Herramientas y recursos

#### 🛠️ Burp Suite:

* Turbo Intruder
* HTTP/2

#### 📜 Scripts:

```python
asyncio + aiohttp
```

#### 📊 Análisis:

* Comparación de tiempos
* Logs
* Estado de la aplicación
