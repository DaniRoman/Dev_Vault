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