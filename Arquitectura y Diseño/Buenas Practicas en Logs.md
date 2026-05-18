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