# 1. Objetivo

```
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

# 3.Formato recomendado

Todo log debería tender a ser estructurado:

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

# 4. Punto global: dónde tocar

Logger wrapper común

```
safeLogger.info(event, fields)
safeLogger.warn(event, fields)
safeLogger.error(event, fields)
```

## Middleware HTTP

Requests entrantes:

```
request started/completed
duration
status
route
method
trace_id
```

no loggear cada request exitoso interno si genera demasiado volumen.

Posible regla:

```
2xx/3xx: sampleado o solo gateway
4xx: depende del caso
5xx: siempre
latencia alta: siempre
```

>[!warning] 
Que es sampleado o solo gateway!¿?¿

---
## Handler global de excepciones

```
Una excepción se loggea una sola vez.
```

Normalmente en el boundary:

>[!warning] 
Que es boundary?

```
controller advice
exception filter
global error handler
middleware de error
```

Evitar que repository, service, controller y middleware loggeen el mismo error.

---

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

>[!warning] No entiendo que es re

Si el campo se llama `authorization`, `token`, `password`, etc.:

```json
{
  "authorization": "[REDACTED]",
  "password": "[REDACTED]"
}
```

## 5.2 Redactar por patrón