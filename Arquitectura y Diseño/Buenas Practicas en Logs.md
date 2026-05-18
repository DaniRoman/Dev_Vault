# 1. Objetivo

```
No todos los logs son iguales,
pero todos deberían pasar por la misma política común.

1. Reducir volumen de logs innecesarios.
2. Evitar exposición de datos sensibles.
3. Estandarizar formato y campos mínimos.
4. Centralizar sanitización en un punto común.
5. Mantener capacidad de debug.6. Dejar un patrón reutilizable para el resto de servicios.
```

# 2. Que loggear, que no 

```txt
//No
password
token
access_token
refresh_token
id_token
authorization
cookie
set-cookie
api_key
secret
private_key
jwt
session_id
connection_string
database_url
card_number
cvv
iban
dni
passport
request body completo
response body completo
headers completos
query params completos

//Campos seguros y útiles:
event
service
environment
version
trace_id
span_id
request_id
method
route normalizada
status_code
duration_ms
error_type
error_code
operation
external_service
retry_count
tenant_id si aplica
user_id_hash si aplica

```

# 4. Punto global: dónde tocar

>[!tip]
>Definir los puntos globales de mi micro o boundarys

lugares del flujo donde puedes implementar lógica común sin tener que modificar cientos de líneas sueltas.
Por ejemplo, en vez de buscar todos los `logger.error(...)` del código, se crear un punto donde todos los errores pasan.

(Un **boundary** es el borde de entrada o salida de un sistema.)

En nuestra arquitectura los puntos globales/boundarys seria.

```txt
1. Entrada por API HTTP.
2. Entrada por RabbitMQ.
3. Entrada directa desde device.
4. Salida hacia otra API.
5. Salida hacia RabbitMQ.
6. Handler global de errores.
7. Logger wrapper común.
8. Pipeline de logs, si tenéis uno.

Por qué loggear en el boundary Porque ahí tienes el contexto completo:
qué operación era
qué mensaje era
qué endpoint era
qué device era
qué queue era
cuánto tardó
si se hizo ack/nack
qué status se respondió
```

## Middleware HTTP

El concepto de “middleware HTTP” aplica **solo cuando el microservicio recibe tráfico HTTP y loggear de forma centralizada:**
Pero el caso del micro que recibe trafico de RabbitMQ y del device directo ampliaríamos el concepto

***Middleware HTTP***

```txt
HTTP request entra
↓
middleware captura method, route, status, duration, trace_id
↓
controller/use case
↓
response
```

```json
{
  "event": "http_request_completed",
  "method": "POST",
  "route": "/devices/{id}/commands",
  "status_code": 200,
  "duration_ms": 54
}
```

### 2xx/3xx sampleado o solo gateway

>[!warning] 
>[[¿Qué significa 2xx 3xx sampleado o solo gateway?]]

## Para RabbitMQ no es middleware HTTP. Sería un **consumer wrapper** o **Rabbit interceptor**.

```txt
Mensaje Rabbit entra
↓
consumer wrapper captura queue, routing_key, message_id, duration, error
↓
handler de negocio
↓
ack/nack/retry
```

```json
{
  "event": "rabbit_message_processed",
  "queue": "device.events",
  "routing_key": "device.telemetry",
  "message_id": "msg-123",
  "duration_ms": 41
}

//SI FALLA 

{  
"level": "ERROR",  
"event": "rabbit_message_processing_failed",  
"queue": "device.events",  
"message_id": "msg-123",  
"error_type": "InvalidPayloadError"  
}
```

Para el device directo seria un **device connection wrapper**.

```txt
Device message entra
↓
connection handler / protocol adapter
↓
wrapper captura device_id_hash, protocol, message_type, duration
↓
translator
```

```json
{
  "event": "device_message_received",
  "protocol": "tcp",
  "message_type": "telemetry",
  "device_id_hash": "hmac_sha256:abc",
  "payload_size_bytes": 512
}
```

## Handler global de excepciones

handler global de excepciones punto común donde caen errores no gestionados o errores finales de una operación.
Un **boundary** es el borde de entrada o salida de un sistema.

```txt
1. Endpoint HTTP.
2. Consumer RabbitMQ.
3. Handler de conexión con device.
4. Job programado.
5. Cliente hacia otra API.
6. Publisher hacia Rabbit.
```


## 4.4 Interceptores de clientes externos

Para llamadas a otros servicios:

```
external_call_started opcional
external_call_completed
external_call_failed
```

Pero sin payload completo.

```json
{
  "level": "WARN",
  "event": "external_call_failed",
  "target_service": "payments",
  "operation": "authorize_payment",
  "status_code": 504,
  "duration_ms": 1200,
  "retry_count": 2,
  "trace_id": "abc"
}
```

---

# 5. Sanitización

### Redactar por nombre de campo

>[!warning]
>Redactar por nombre de campo¿?

Si el campo se llama `authorization`, `token`, `password`, etc.:

```json
{
  "authorization": "[REDACTED]",
  "password": "[REDACTED]"
}
```

### Redactar por patrón

Aunque el nombre del campo sea inocente, puede venir un token dentro del valor.
>[!warning]
>Redactar por patrón¿?.

```
Bearer eyJ...
JWT
API keys
emails si aplica
tarjetas
cookies
```

### Truncar por valores Grandes?

>[!warning]
>Como conseguimos eso¿?.

máximo 2 KB por campo
máximo 8 KB por evento completo
máximo 20 elementos en arrays
máximo profundidad 4 en objetos

### No permitir objetos arbitrarios

```ts
//Malo
logger.info("request", request);
logger.info("user", user);
logger.info("response", response);

//Bueno
logger.info("user_loaded", {
  user_id_hash,
  account_status,
  source
});
```

# Politica de niveles

## ERROR

Solo cuando algo falla de verdad:

```
operación fallida
excepción no controlada
fallo externo agotando retries
inconsistencia de datos
pérdida de mensaje
```

## WARN

Anomalía recuperable:

```
retry
fallback
latencia alta
circuit breaker abierto
datos incompletos pero procesables
```

## INFO

Evento relevante:

```
servicio iniciado
job terminado
pedido creado
pago autorizado
cambio de estado importante
```

## DEBUG

Solo temporal o local.

En producción:

>[!warning] 
>Como lo hago desactivable?


```
DEBUG desactivado por defecto
activable con TTL
por servicio / trace_id / tenant / usuario hasheado
```

# Antipatrones a buscar para modificar

```json
logger.info(request)
logger.info(response)
logger.info(headers)
logger.info(body)
logger.error(e) repetido en varias capas
console.log
printStackTrace
logs dentro de loops
logs de health checks
logs de cada operación interna trivial
logs con concatenación de strings
logs sin event name
logs sin trace_id/request_id

Authorization
Cookie
Bearer
password
token
secret
body
payload
headers
```

# Plan de trabajo

### Fase 1 Inventario

```txt
cuántos logs hay
en qué niveles
qué clases/módulos loggean más
si hay request/response bodies
si hay headers
si hay tokens
si hay logs duplicados de excepción
si hay logs dentro de loops
si hay access logs muy ruidosos
```

| Tipo                        |       Riesgo | Acción                         |
| --------------------------- | -----------: | ------------------------------ |
| Log de request body         |         Alto | Eliminar o allowlist           |
| Log de Authorization header |      Crítico | Redactar inmediatamente        |
| Log repetido de excepción   |        Medio | Centralizar en handler         |
| Health checks               | Bajo/volumen | Dropear                        |
| INFO en loop                | Alto volumen | Convertir a métrica o samplear |


### Fase 2 Definir estándar mínimo

```txt
campos obligatorios
niveles
campos prohibidos
eventos permitidos
reglas de sanitización
reglas de sampling
ejemplos buenos/malos
```

### Fase3 Implementar sanitizer común


```txt
componente reutilizable.

denylist de campos sensibles
redacción por regex
truncado
límite de profundidad
protección contra objetos enormes
tests unitarios
```

>[!warning] 
>Componente reutilizable? como wraper?

### Fase 4: Crear logger wrapper

>[!warning] 
>Que es mas útil tener dependiendo de mi arquitectura de microservicios?

^ac9cee
Logger wrapper común: normalmente será una **clase o módulo común**, pero el concepto importante es que sea el **punto obligatorio de entrada al logging**.. La idea es tener una **abstracción común** que todos usen.

```txt
Micro A ─┐
Micro B ─┼── usan misma librería/patrón → SafeLogger
Micro C ─┘
```

`saveLogger` se encargaría de...

```txt
	1. Añadir campos comunes.
	2. Sanitizar datos sensibles.
	3. Truncar valores demasiado grandes.
	4. Normalizar el formato.
	5. Añadir trace_id/request_id/message_id.
	6. Evitar que se loggeen objetos completos peligrosos.
	7. Aplicar reglas de nivel.
```

```ts
//NO
logger.info("Request received", request);
//SI
safeLogger.info("rabbit_message_processed", {  
	queue: "device.events",  
	message_id: messageId,  
	duration_ms: duration  
});  
  
safeLogger.error("device_message_processing_failed", error, {  
	queue: "device.events",  
	message_id: messageId  
});

```

#### Formato recomendado

Envelope común + campos específicos según del tipo de evento `info, warning...`

```json
//COMUN
{
  "timestamp": "2026-05-18T10:30:00.000Z",
  "level": "INFO",
  "event": "device_message_received",
  "service": "connection-layer",
  "environment": "prod",
  "version": "1.4.2",
  "trace_id": "abc123",
  "request_id": "req-123"
}
// + ERROR
{
  "timestamp": "2026-05-18T10:30:05.000Z",
  "level": "ERROR",
  "event": "device_message_processing_failed",
  "service": "translator-service",
  "environment": "prod",
  "trace_id": "abc123",
  "message_id": "msg-789",
  "queue": "device.events",
  "error_type": "InvalidPayloadError",
  "error_code": "INVALID_DEVICE_PAYLOAD",
  "error_message": "Payload validation failed"
}
// WARN
{
  "timestamp": "2026-05-18T10:30:02.000Z",
  "level": "WARN",
  "event": "external_api_retry",
  "service": "connection-layer",
  "environment": "prod",
  "target_service": "device-api",
  "operation": "send_command",
  "retry_count": 2,
  "duration_ms": 1200,
  "trace_id": "abc123"
}
```
Todos los logs deberían tener algo parecido a esto:

```json



{  
"level": "INFO",  
"event": "order_created",  
"service": "orders-api",  
"environment": "prod",  
"trace_id": "abc",  
"request_id": "req-123",  
"route": "/orders/{id}",  
"method": "POST",  
"status_code": 201,  
"duration_ms": 83}
```
