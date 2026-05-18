política común + un punto global de sanitización + reducción de ruido.

Objetivo
1. Reducir volumen de logs innecesarios.
2. Evitar exposición de datos sensibles.
3. Estandarizar formato y campos mínimos.
4. Centralizar sanitización en un punto común.
5. Mantener capacidad de debug.
6. Dejar un patrón reutilizable para el resto de servicios.

Datos que no deben de mostrarse

```json
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
```

Estructura del log 

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
  "duration_ms": 83
}
```


>[!warning] Importante
### 4.1 Logger wrapper común

Algo tipo:

```
safeLogger.info(event, fields)safeLogger.warn(event, fields)safeLogger.error(event, fields)
```

Ese wrapper hace:

```
sanitizar camposlimitar tamañoinyectar trace_id/request_idnormalizar estructurabloquear campos prohibidos
```

La idea es que el código de negocio no llame directamente al logger base siempre que sea posible.

## 4.2 Middleware HTTP

Para requests entrantes:

```
request started/completeddurationstatusroutemethodtrace_id
```

Pero con cuidado: no loggear cada request exitoso interno si genera demasiado volumen.

Posible regla:

```
2xx/3xx: sampleado o solo gateway4xx: depende del caso5xx: siemprelatencia alta: siempre
```

## 4.3 Handler global de excepciones

Regla crítica:

```
Una excepción se loggea una sola vez.
```

Normalmente en el boundary:

```
controller adviceexception filterglobal error handlermiddleware de error
```

Evitar que repository, service, controller y middleware loggeen el mismo error.