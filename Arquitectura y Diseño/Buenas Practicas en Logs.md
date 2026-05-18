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

Para llamadas a otros servicios

Pero si se publican miles por segundo, esto se samplea o se convierte en métrica.

>[!tip]
>Como y cuando samplear ¿?

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

“Redactar” significa reemplazar un dato sensible por algo seguro.

Si el campo se llama `authorization`, `token`, `password`, etc.:

```json
{
  "user": "akocloud",
  "password": "123456",
  "authorization": "Bearer eyJhbGciOi...",
  "device_id": "dev-123"
}
//SE REMPLAZAN 
{  
"user": "akocloud",  
"password": "[REDACTED]",  
"authorization": "[REDACTED]",  
"device_id": "dev-123"  
}
```

### Redactar por patrón

Aquí no se confía en el nombre del campo. Miras el **contenido**. Aunque el nombre del campo sea inocente, puede venir un token dentro del valor.

```json
{
  "message": "Calling API with Authorization: Bearer eyJhbGciOiJIUzI1NiIs..."
}
//Se aplica regex/patrones:
{
  "message": "Calling API with Authorization: Bearer [REDACTED]"
}
```

### Truncar por valores Grandes

```txt
Reglas razonables 

Máximo 2 KB por campo string.
Máximo 8 KB por evento completo.
Máximo 20 elementos por array.
Máximo profundidad 4 en objetos.
Máximo stacktrace 8-12 KB.
```

Si excede del limite

```txt
1. Eliminar campos no esenciales.
2. Truncar message/stacktrace/payload_preview.
3. Dejar campos críticos: event, level, service, trace_id, error_type.
```

serializar el evento y mirar cuánto ocupa.

>[!warning]
>Que es serializar, entiendo que todo esto es una función para cada caso no ¿?

## No permitir objetos arbitrarios

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

varias capas, o configuración por entorno, o TTL

>[!warning] 
>TTL? Redis?

En producción:

```
LOG_LEVEL=INFO
```

En desarrollo:

```
LOG_LEVEL=DEBUG
```

Regla:

```
Si LOG_LEVEL=INFO, los DEBUG no se emiten.
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

cuántos logs hay en qué niveles qué clases/módulos loggean más
- Si hay request/response bodies
- Si hay headers
- Si hay tokens
- Si hay logs duplicados de excepción
- Si hay logs dentro de loops
- Si hay access logs muy ruidosos

|Campo|Descripción|Ejemplo|
|---|---|---|
|Servicio|Nombre del microservicio|`connection-layer`|
|Módulo / archivo|Dónde está el log|`DeviceConsumer.ts`|
|Línea aproximada|Línea o función|`processMessage()`|
|Entrada|HTTP, Rabbit, Device, Job, Internal|`Rabbit`|
|Evento actual|Qué loggea ahora|`"Received payload"`|
|Nivel actual|DEBUG, INFO, WARN, ERROR|`INFO`|
|Frecuencia|Alta, media, baja|Alta|
|Volumen estimado|Logs/día o logs/min|`500k/día`|
|Contiene payload|Sí/No|Sí|
|Contiene headers|Sí/No|No|
|Contiene datos sensibles|Sí/No/Dudoso|Dudoso|
|Tiene trace_id/request_id/message_id|Sí/No|No|
|Es necesario|Sí/No/Parcial|Parcial|
|Problema|Ruido, sensible, duplicado, sin contexto|Payload completo|
|Acción|Mantener, eliminar, samplear, sanitizar, mover a DEBUG|Sanitizar + reducir|
|Nuevo evento propuesto|Nombre estructurado|`rabbit_message_received`|
|Nuevo nivel propuesto|INFO/WARN/ERROR/DEBUG|DEBUG|
|Campos permitidos|Qué campos se conservarán|`queue,message_id,duration_ms`|
|Owner|Responsable|Backend/platform|
|Prioridad|Alta/media/baja|Alta|

Ejemplo rellenado

| Servicio         | Módulo              | Entrada     | Log actual                    | Nivel | Frecuencia | Riesgo               | Acción                              | Nuevo evento                       |
| ---------------- | ------------------- | ----------- | ----------------------------- | ----- | ---------- | -------------------- | ----------------------------------- | ---------------------------------- |
| translator       | `DeviceConsumer`    | Rabbit      | `Received payload: {payload}` | INFO  | Alta       | Payload sensible     | Mover a DEBUG + truncar + sanitizar | `rabbit_message_received`          |
| translator       | `DeviceConsumer`    | Rabbit      | `Error processing message`    | ERROR | Media      | Duplicado            | Dejar solo en consumer boundary     | `rabbit_message_processing_failed` |
| connection-layer | `DeviceApiClient`   | HTTP client | `Calling API with headers`    | INFO  | Alta       | Authorization header | Eliminar headers + safe fields      | `external_http_call_started`       |
| api              | `CommandController` | HTTP        | `POST /command body...`       | INFO  | Alta       | Body completo        | Eliminar body + route normalizada   | `http_request_completed`           |


### Fase 2 Definir estándar mínimo común 

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

>[!tip]
En mi caso se utiliza `looger /akocloud-micros/src/micros/lib/logger.ts`


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




