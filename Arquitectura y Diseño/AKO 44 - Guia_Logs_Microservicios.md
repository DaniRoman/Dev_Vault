# Guía de Mejora de Logs en Microservicios

---

## Bloque 1 — Explicación conceptual

### Paso 1. Por qué mejorar los logs

Los logs son la principal herramienta para entender qué ocurre en producción. Sin embargo, un sistema de logging mal diseñado genera tres problemas concretos:

- **Exposición de datos sensibles**: tokens, contraseñas, cookies o datos personales pueden acabar en un sistema de logs y convertirse en un vector de ataque.
- **Ruido excesivo**: demasiados logs dificultan encontrar los que importan y aumentan costes de almacenamiento.
- **Falta de contexto**: logs sin estructura ni correlación hacen imposible rastrear un fallo a través de varios servicios.

El objetivo no es loggear menos ni más, sino loggear mejor: con estructura, con contexto, sin datos sensibles y en el nivel correcto.

---

### Paso 2. Qué problemas queremos evitar

| Problema | Causa habitual |
|---|---|
| Datos sensibles en logs | Loggear objetos completos (request, body, headers) |
| Logs duplicados de error | Cada capa loggea la misma excepción |
| Logs sin contexto | Ausencia de `trace_id`, `event`, `service` |
| Ruido masivo | Logs en loops, health checks, operaciones triviales |
| Formato inconsistente | Cada servicio usa su propio estilo |
| Objetos arbitrarios | `logger.info("request", request)` vuelca todo |

---

### Paso 3. Buenas prácticas generales

1. Usar siempre un **logger común centralizado** (`SafeLogger`). Ningún servicio llama al logger directamente.
2. Todos los logs deben ser **JSON estructurado**, nunca texto libre concatenado.
3. Cada log debe tener un campo `event` con un nombre estable y descriptivo.
4. **Una excepción se loggea una sola vez**, en el punto más cercano al boundary, no en cada capa.
5. Prohibido loggear objetos completos: request, response, body, headers. Solo campos específicos y seguros.
6. Los logs de `DEBUG` están desactivados en producción por defecto.

---

### Paso 4. Formato recomendado

Todo log tiene un **envelope común** más campos específicos según el tipo de evento.

**Envelope común (obligatorio en todos los logs):**

```json
{
  "timestamp": "2026-05-18T10:30:00.000Z",
  "level": "INFO",
  "event": "device_message_received",
  "service": "connection-layer",
  "environment": "prod",
  "version": "1.4.2",
  "trace_id": "abc123"
}
```

**Campos adicionales según contexto:**

| Campo | Cuándo incluirlo |
|---|---|
| `request_id` | Entradas HTTP |
| `message_id` | Mensajes RabbitMQ |
| `device_id_hash` | Conexiones directas con dispositivos |
| `duration_ms` | Operaciones con tiempo medible |
| `error_type` | Solo en logs de error |
| `error_code` | Solo en logs de error |
| `route` | Requests HTTP |
| `method` | Requests HTTP |
| `status_code` | Respuestas HTTP |
| `retry_count` | Operaciones con reintentos |
| `target_service` | Llamadas a servicios externos |

**Catálogo de nombres de evento:**

```
http_request_completed         rabbit_message_received
http_request_failed            rabbit_message_processed
device_connected               rabbit_message_processing_failed
device_disconnected            rabbit_message_published
device_message_received        external_http_call_completed
device_message_rejected        external_http_call_failed
validation_failed              command_translated
command_sent_to_device
```

---

### Paso 5. Niveles de log

| Nivel | Cuándo usarlo | Ejemplo |
|---|---|---|
| `DEBUG` | Diagnóstico temporal, solo en desarrollo | Payload parcial sanitizado para inspección |
| `INFO` | Evento importante, esperado y normal | Mensaje procesado, comando enviado |
| `WARN` | Anomalía recuperable | Retry, timeout recuperado, fallback activado |
| `ERROR` | Fallo real de operación | Mensaje no procesado, API externa caída |

**Regla de entorno:**
- Producción → `LOG_LEVEL=INFO` (los DEBUG no se emiten)
- Desarrollo → `LOG_LEVEL=DEBUG`

---

### Paso 6. Sanitización

La sanitización garantiza que ningún dato sensible llegue al sistema de logs. Se aplica en dos niveles:

**Por nombre de campo** — si el campo tiene un nombre conocido como sensible, se redacta su valor:

```json
// Entrada
{ "password": "123456", "authorization": "Bearer eyJ...", "device_id": "dev-123" }

// Salida sanitizada
{ "password": "[REDACTED]", "authorization": "[REDACTED]", "device_id": "dev-123" }
```

**Campos que siempre se redactan:**
`password`, `token`, `access_token`, `refresh_token`, `id_token`, `authorization`, `cookie`, `set-cookie`, `api_key`, `secret`, `private_key`, `jwt`, `session_id`, `connection_string`, `database_url`, `card_number`, `cvv`, `iban`, `dni`, `passport`

**Por patrón de contenido** — aunque el nombre del campo sea inocente, se detectan valores con forma de token o credencial mediante expresiones regulares:

```json
// Entrada
{ "message": "Calling API with Authorization: Bearer eyJhbGciOiJIUzI1NiIs..." }

// Salida sanitizada
{ "message": "Calling API with Authorization: Bearer [REDACTED]" }
```

**Por tamaño** — los valores que excedan los límites se truncan:

| Límite | Valor |
|---|---|
| Máximo por campo string | 2 KB |
| Máximo por evento completo | 8 KB |
| Máximo elementos en array | 20 |
| Máxima profundidad de objeto | 4 niveles |
| Máximo stacktrace | 8–12 KB |

Si un evento supera el límite total: eliminar campos no esenciales primero, luego truncar `message` y `stacktrace`, y conservar siempre `event`, `level`, `service`, `trace_id`, `error_type`.

---

### Paso 7. Reducción de ruido

Un log de calidad no es uno que loggea todo, sino uno que loggea lo necesario.

**Qué no loggear nunca:**
- Requests y responses completos
- Headers completos
- Bodies completos
- Query params completos
- Health checks
- Operaciones internas triviales (getters, validaciones menores)
- Logs dentro de loops de alto volumen

**Cómo reducir sin perder visibilidad:**
- Mover logs de operaciones frecuentes y normales a `DEBUG`.
- Para operaciones de muy alto volumen (miles por segundo), usar **muestreo (sampling)**: loggear 1 de cada N eventos y convertir el resto en métricas.
- Loggear solo en el boundary de entrada/salida, no en cada capa interna.

---

### Paso 8. Puntos globales donde intervenir (boundaries)

Un boundary es el borde de entrada o salida de un sistema. Loggear ahí y no en cada capa interna permite centralizar el logging sin tocar cientos de líneas de código.

Los boundaries de nuestra arquitectura son:

| Boundary | Qué captura |
|---|---|
| **Middleware HTTP** | method, route normalizada, status_code, duration_ms, trace_id |
| **Consumer wrapper RabbitMQ** | queue, routing_key, message_id, duration_ms, ack/nack |
| **Device connection wrapper** | protocol, message_type, device_id_hash, payload_size_bytes |
| **Handler global de excepciones** | Todos los errores no gestionados de cualquier fuente |
| **Interceptor de clientes externos** | target_service, operation, status_code, duration_ms, retry_count |
| **SafeLogger (logger wrapper)** | Punto de entrada único para todos los logs |

**Ejemplo por tipo de boundary:**

```json
// HTTP
{ "event": "http_request_completed", "method": "POST", "route": "/devices/{id}/commands", "status_code": 200, "duration_ms": 54 }

// RabbitMQ (éxito)
{ "event": "rabbit_message_processed", "queue": "device.events", "routing_key": "device.telemetry", "message_id": "msg-123", "duration_ms": 41 }

// RabbitMQ (fallo)
{ "level": "ERROR", "event": "rabbit_message_processing_failed", "queue": "device.events", "message_id": "msg-123", "error_type": "InvalidPayloadError" }

// Device directo
{ "event": "device_message_received", "protocol": "tcp", "message_type": "telemetry", "device_id_hash": "hmac_sha256:abc", "payload_size_bytes": 512 }

// Llamada externa fallida
{ "level": "WARN", "event": "external_call_failed", "target_service": "payments", "operation": "authorize_payment", "status_code": 504, "duration_ms": 1200, "retry_count": 2 }
```

---

### Paso 9. Errores comunes que evitar

```
logger.info(request)              → vuelca el objeto completo
logger.info(response)             → ídem
logger.info(headers)              → expone Authorization, Cookie, etc.
logger.error(e) en cada capa      → el mismo error se loggea 3-4 veces
console.log / printStackTrace     → no estructurado, no sanitizado
logs dentro de loops              → genera ruido masivo
logs de health checks             → ruido sin valor operativo
logs sin campo "event"            → imposible filtrar o alertar
logs sin trace_id                 → imposible correlacionar en distribuido
strings concatenados en el log    → rompe el JSON estructurado
objetos arbitrarios como contexto → pueden contener datos sensibles
```

---

## Bloque 2 — Fase de implementación paso a paso

> Esta guía está pensada para aplicarse en dos microservicios piloto y reutilizarse después en otros servicios o en la API principal.

---

### Paso 1. Inventario inicial de logs

Antes de tocar nada, conocer el estado actual de cada servicio. Para cada log registrado, rellenar la siguiente tabla:

| Campo | Descripción | Ejemplo |
|---|---|---|
| Servicio | Nombre del microservicio | `connection-layer` |
| Módulo / archivo | Dónde está el log | `DeviceConsumer.ts` |
| Función / línea | Dónde exactamente | `processMessage()` |
| Tipo de entrada | HTTP, Rabbit, Device, Job, Internal | `Rabbit` |
| Log actual | Qué se loggea ahora | `"Received payload: {payload}"` |
| Nivel actual | DEBUG / INFO / WARN / ERROR | `INFO` |
| Frecuencia | Alta / media / baja | Alta |
| Contiene payload | Sí / No | Sí |
| Contiene headers | Sí / No | No |
| Contiene datos sensibles | Sí / No / Dudoso | Dudoso |
| Tiene trace_id | Sí / No | No |
| Problema detectado | Ruido / sensible / duplicado / sin contexto | Payload completo |
| Acción propuesta | Mantener / eliminar / mover a DEBUG / sanitizar | Sanitizar + DEBUG |
| Nuevo evento propuesto | Nombre estructurado | `rabbit_message_received` |
| Campos permitidos finales | Qué campos se conservarán | `queue, message_id, duration_ms` |
| Prioridad | Alta / media / baja | Alta |

---

### Paso 2. Detección de logs problemáticos

Buscar en el código los siguientes patrones de riesgo:

**Patrones de código a buscar:**
```
logger.info(request)
logger.info(response)
logger.info(body)
logger.info(headers)
logger.error(e)           ← repetido en varias capas
console.log
printStackTrace
```

**Palabras clave en el contenido de los logs:**
```
Authorization   Bearer   password   token   secret
body            payload  headers    Cookie
```

**Situaciones estructurales:**
- Logs dentro de bucles (`for`, `while`, `forEach`)
- Logs en endpoints de health check (`/health`, `/ping`, `/status`)
- El mismo error loggeado en más de una capa
- Logs sin campo `event`
- Logs sin `trace_id` o `request_id`

---

### Paso 3. Definir el estándar mínimo común

Antes de escribir código, acordar y documentar las reglas que aplicarán a todos los servicios:

| Regla | Descripción |
|---|---|
| Formato | JSON estructurado siempre |
| Punto de entrada | Usar `SafeLogger`, nunca el logger directamente |
| Sanitización | Todo log pasa por el sanitizer antes de emitirse |
| Correlación | Incluir `trace_id`, `request_id` o `message_id` cuando aplique |
| Campo `event` | Obligatorio en todos los logs |
| Payloads | Prohibido loggear payload completo por defecto |
| Headers | Prohibido loggear headers completos |
| Errores | Una excepción se loggea una sola vez |
| DEBUG en prod | Desactivado por defecto |
| Truncado | Aplicar límites de tamaño a campos y eventos |

---

### Paso 4. Crear o adaptar el SafeLogger

El `SafeLogger` es el único punto de entrada al sistema de logging. Todos los servicios lo usan; nadie llama al logger directamente.

**Lo que hace el SafeLogger:**
1. Añade los campos comunes del envelope (timestamp, service, environment, version).
2. Inyecta `trace_id` / `request_id` / `message_id` desde el contexto.
3. Aplica el sanitizer completo sobre los campos.
4. Trunca valores que superen los límites.
5. Normaliza el formato JSON.
6. Aplica las reglas de nivel (no emite DEBUG en prod).
7. Rechaza objetos arbitrarios como contexto.

**API de uso esperada:**

```ts
// ❌ No
logger.info("Request received", request);

// ✅ Sí
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

Si ya existe un logger en `lib/logger.ts`, adaptarlo añadiendo las capas de sanitización, truncado y campos comunes. No reemplazarlo desde cero si puede evitarse.

---

### Paso 5. Implementar el sanitizer

El sanitizer es una función que recibe los campos de un log y devuelve una versión segura. Se llama internamente desde el SafeLogger; los servicios no la invocan directamente.

**Pipeline del sanitizer:**

```
input: campos del log
  ↓
1. Redactar por nombre de campo (lista de campos sensibles)
  ↓
2. Redactar por patrón en strings (regex para tokens, Bearer, etc.)
  ↓
3. Truncar strings que superen 2 KB
  ↓
4. Limitar arrays a 20 elementos
  ↓
5. Limitar profundidad de objetos a 4 niveles
  ↓
6. Verificar tamaño total del evento (máx. 8 KB)
     → Si excede: eliminar campos no esenciales, luego truncar message/stacktrace
     → Conservar siempre: event, level, service, trace_id, error_type
  ↓
output: evento seguro y dentro de límites
```

---

### Paso 6. Centralizar el handler de errores

Crear un único handler global de excepciones por microservicio. Este handler es el único punto que loggea errores no controlados. Las capas internas no loggean errores; los propagan hacia arriba.

**Fuentes que deben conectarse al handler global:**
1. Controladores HTTP (errores no capturados en endpoints)
2. Consumers RabbitMQ
3. Handlers de conexión con dispositivos
4. Jobs programados
5. Clientes hacia APIs externas

El log del error debe incluir: `event`, `error_type`, `error_code`, `error_message` (sin stacktrace completo en prod), y los identificadores de contexto disponibles (`trace_id`, `message_id`, etc.).

---

### Paso 7. Implementar los wrappers por tipo de entrada

Cada tipo de entrada tiene su propio wrapper que loggea en el boundary, liberando al código de negocio de responsabilidad de logging.

**Middleware HTTP:**
- Intercepta toda request entrante.
- Loggea al completarse: `method`, `route` (normalizada, sin IDs reales), `status_code`, `duration_ms`, `trace_id`.
- Los 2xx de alta frecuencia pueden samplearse o dejarse solo al gateway.

**Consumer wrapper RabbitMQ:**
- Envuelve cada consumer.
- Loggea al recibir y al completar: `queue`, `routing_key`, `message_id`, `duration_ms`, resultado (`ack`/`nack`).
- En fallo: loggea `error_type` y el contexto del mensaje.

**Device connection wrapper:**
- Envuelve el protocol adapter o connection handler.
- Loggea al recibir mensaje: `protocol`, `message_type`, `device_id_hash` (nunca el ID real), `payload_size_bytes`.

**Interceptor de clientes externos:**
- Envuelve las llamadas salientes a otras APIs.
- Loggea: `target_service`, `operation`, `status_code`, `duration_ms`, `retry_count`.
- Nunca incluir el payload de la llamada ni las cabeceras.

---

### Paso 8. Reducir logs excesivos

Con el inventario del Paso 1 y los wrappers del Paso 7, aplicar las acciones identificadas:

- **Mover a DEBUG**: logs de operaciones frecuentes y normales que no son errores ni eventos de negocio relevantes.
- **Eliminar**: logs de health checks, loops, operaciones triviales, duplicados de error en capas internas.
- **Samplear**: logs de operaciones de muy alto volumen donde loggear cada evento no aporta valor diferencial.
- **Consolidar**: si hay varios logs del mismo flujo, dejar solo el del boundary y eliminar los intermedios.

---

### Paso 9. Validar que la mejora funciona

Tras implementar los cambios, verificar:

- [ ] Ningún log en producción contiene tokens, passwords, cookies ni headers completos.
- [ ] Todos los logs tienen campo `event` con nombre del catálogo acordado.
- [ ] Todos los logs tienen `trace_id` o el identificador de correlación correspondiente.
- [ ] Los errores no aparecen duplicados en múltiples capas.
- [ ] No hay `console.log` ni `printStackTrace` activos en producción.
- [ ] El nivel `DEBUG` está desactivado en producción.
- [ ] Los logs de health checks no aparecen en el sistema de logs.

---

### Paso 10. Medir el antes y el después

Registrar las siguientes métricas antes de empezar y volver a medirlas tras la implementación:

| Métrica | Antes | Después |
|---|---|---|
| Volumen total de logs / día | | |
| % de logs con datos sensibles detectados | | |
| % de logs con campo `event` | | |
| % de logs con `trace_id` | | |
| Número de logs ERROR duplicados en el mismo flujo | | |
| Presencia de `console.log` en producción | | |
| Tamaño medio por evento (KB) | | |

---

## Promp

[[AKO 44 - Guia_Logs_Microservicios - promp]]


## Resumen de lo hecho (5 pasos)


### La arquitectura: 3 capas

>[!important] 

>`WinstonCloudwatch.ts` es el puente que ya implementamos. Sin él, los logs solo van a consola y desaparecen cuando el contenedor se reinicia. Con él, quedan persistidos en AWS


```
Tráfico real  →  Microservicio  →  logger.ts  →  CloudWatch / consola
                      ↕
              sanitizer.ts  ←  safeLogger.ts
```

**Capa 1 — `sanitizer.ts`** (el filtro de seguridad)
Antes de que cualquier dato salga al log, pasa por un pipeline automático de 7 pasos en orden:
1. **Campos sensibles por nombre** — si el campo se llama `password`, `token`, `authorization`, `api_key`, `secret`, `jwt`, `iban`, `cvv`, `imei`, etc. → se sustituye por `[REDACTED]`, sin importar el valor
2. **Patrones de credenciales en strings** — detecta `Bearer eyJ...`, `token=abc123`, `secret=xyz` aunque estén dentro de un texto largo → `[REDACTED]`
3. **URLs con credenciales embebidas** — `amqp://user:pass@host` → `[REDACTED_URL]`
4. **Números de tarjeta** — secuencias de 13–19 dígitos con separadores → `[REDACTED_CARD]`
5. **Emails** — `user@domain.com` → `[REDACTED_EMAIL]`
6. **IPs privadas** — rangos RFC 1918 (192.168.x.x, 10.x.x.x, 172.16-31.x.x) y loopback → `[REDACTED_IP]`
7. **Límites de tamaño** — strings > 2 KB se truncan, arrays > 20 elementos se cortan, objetos > 4 niveles de profundidad se capan. Nada vuela CloudWatch por un payload de 500 KB.

**Capa 2 — `safeLogger.ts`** (el emisor estructurado)
API limpia para cualquier microservicio nuevo:
```typescript
const log = new SafeLogger("cl-12830");

log.info("device_message_received", { method: "POST", serial: "123456" });
log.error("exception_unhandled", error, { context: "handleCLOutput" });

// Con traza de mensaje (para correlacionar en CloudWatch):
const tracedLog = log.withTrace(correlationId, messageId);
tracedLog.info("rabbit_message_processed", { type: "CMD" });
```
Cada llamada emite automáticamente un JSON con los campos obligatorios: `event`, `service`, `environment`, y opcionalmente `trace_id` / `message_id`. El `timestamp` y `level` los añade Winston.

**Capa 3 — `logger.ts`** (el transporte, sin cambios de API)
Respeta `LOG_LEVEL` por variable de entorno. En producción: `info` y superior. En desarrollo: `debug` activado. Enruta a consola y a CloudWatch si `AWS_CLOUDWATCH_ENABLED=true`.

---

### El catálogo de eventos — nombres estables para buscas en CloudWatch

En lugar de strings libres imposibles de buscar, todos los logs nuevos usan nombres del catálogo:

| Evento | Cuándo ocurre |
|---|---|
| `device_message_received` | CoAP llega un mensaje de un dispositivo físico |
| `device_message_rejected` | Mensaje rechazado por integridad/autenticación |
| `rabbit_message_received` | AMQP recibe un mensaje entrante |
| `rabbit_message_processed` | Mensaje procesado correctamente |
| `rabbit_message_processing_failed` | Parser devuelve null o lanza excepción |
| `rabbit_message_published` | Mensaje publicado a RabbitMQ (antes volcaba el payload entero) |
| `command_translated` | Translator convirtió un comando al formato del dispositivo |
| `command_sent_to_device` | Mensaje enviado al driver CoAP |
| `validation_passed` | Payload validado correctamente contra el schema |
| `validation_failed` | Payload inválido, campo faltante, firmware incompatible |
| `database_query` / `database_query_result` | Consulta a MongoDB/Redis (sin exponer el contenido) |
| `database_error` | Error en BD — solo el mensaje, nunca el objeto completo |
| `exception_unhandled` | Error capturado en catch — solo `error.message`, sin stack en producción |

---

### Qué se eliminó — logs problemáticos retirados

#### `cl-12830` — 28 logs problemáticos eliminados

| Archivo | Qué había | Qué hay ahora |
|---|---|---|
| `perte-coap/microservice.ts` | `JSON.stringify(request)` completo por cada mensaje CoAP + delimitadores `#####` | `device_message_received :: method=X url=Y serial=Z` |
| `perte-coap/microservice.ts` | 4× `JSON.stringify(error)` en catches | `exception_unhandled :: context=X error=message` |
| `authentication.ts` | `JSON.stringify(message.payload)` — el payload del dispositivo | `device_message_received :: serial=X action=Y` |
| `authentication.ts` | `JSON.stringify(deviceInfo.body)` — información completa del dispositivo | `integrity_check_passed :: serial=X` |
| `authentication.ts` | `console.log(body, body.existsInManufactured, message)` — objeto completo | `device_not_found :: serial=X existsInManufactured=bool` |
| `authentication.ts` | `JSON.stringify(deviceInfo)` en `_validateSerialNumber` | `_validateSerialNumber :: result=success/error` |
| `checkIntegrity.ts` | `console.log(decode(payload))` — el payload decodificado raw | `_decodePayload :: payload_bytes=N` |
| `checkIntegrity.ts` | `JSON.stringify(decodePayload)` en debug | solo keys del objeto, nunca valores |
| `checkIntegrity.ts` | `JSON.stringify(deviceInfo)` en UUID missing | `validation_failed :: reason=uuid_missing` |
| `microservice.abstract.ts` | 8× `console.log` convertidos a `this.log` | Todos migrados |
| `microservice.abstract.ts` | `JSON.stringify(copyData)` en **CADA publicación RabbitMQ** (crítico) | `rabbit_message_published :: exchange=X routingKey=Y` |
| `microservice.abstract.ts` | `JSON.stringify(data)` en RPC response | `rabbit_message_published :: type=RPC-RESPONSE` |
| `microservice.abstract.ts` | 8× `JSON.stringify(e/err)` en conexiones AMQP | `error.message` solamente |
| `repositories/ManufacturedRepository.ts` | `JSON.stringify(query)` + `JSON.stringify(response)` | `database_query :: key=X` + `found=true/false` |
| `repositories/IPRepository.ts` | `JSON.stringify(query)` + `JSON.stringify(response)` | `database_query :: key=X` + `found=true/false` |
| `adapters/data-access-manager.layer.ts` | `JSON.stringify(adapter)` + `JSON.stringify([query, sort, model])` + `JSON.stringify(queryObject)` | solo model + adapter index |
| `adapters/redis-database.adapter.ts` | `JSON.stringify(query)` + `JSON.stringify(queryKey)` | `database_query :: model=X` |

#### `translator-12830` — 22 logs problemáticos eliminados

| Archivo | Qué había | Qué hay ahora |
|---|---|---|
| `microservice.ts` | 4× `JSON.stringify(parsedMsg/message)` en entrada de handlers | `command_translated :: handler=X device=Y` |
| `microservice.ts` | 4× `JSON.stringify(toCLMessage)` justo antes de publicar | `command_sent_to_device :: handler=X device=Y connectivity=Z` |
| `tl-lib/parsers/Parser.ts` | 4× `JSON.stringify(parsedData)` por cada tipo de mensaje | `command_translated :: type=CMD/PARAM/... ok=true` |
| `tl-lib/parsers/implementations/SyncParser.ts` | 3× `JSON.stringify(d/singleD/bitMaps)` en errores de parseo | `validation_failed :: reason=X field=Y` |
| `tl-lib/parsers/implementations/LiveParser.ts` | 3× `JSON.stringify(bitMaps/bitMap/analogInputs)` | `validation_failed :: reason=X field=Y` |
| `tl-lib/services/InputProcessor.ts` | 3× `JSON.stringify(err/message)` | `exception_unhandled` + `validation_failed` con reason |
| `tl-lib/services/InputValidator.ts` | 3× `JSON.stringify(deviceInformation/payload)` | `validation_failed/passed :: type=X serial=Y` |
| `tl-lib/services/DeviceService.ts` | 2× `JSON.stringify(err)` en errores de Redis/BD | `database_error :: context=X device=Y error=message` |
| `tl-lib/services/ParamMsgService.ts` | `JSON.stringify(data)` al almacenar param | `rabbit_message_processed :: context=storeParam device=X` |

---

### Métricas — antes vs. después

| Métrica | Antes | Después |
|---|---|---|
| `console.log` activos en los 2 micros | ~12 | **0** |
| `JSON.stringify` dentro de llamadas a logger | ~50 | **0** |
| Payloads de dispositivo expuestos en logs | ✅ múltiples | **0** |
| Payload RabbitMQ publicado en log (crítico) | ✅ cada publicación | **0** |
| Logs con objeto de error completo (`JSON.stringify(err)`) | ~15 | **0** |
| Campos sensibles automáticamente redactados | 0 | **20 campos** |
| Patrones de credenciales en strings detectados | 0 | **6 patrones** |
| IPs/emails/tarjetas/URLs redactados | 0 | **4 tipos** |
| Eventos con nombre estable (buscables en CloudWatch) | 0 | **13 eventos** |
| Soporte `trace_id` / `message_id` en logs | ❌ | ✅ vía `withTrace()` |
| `DEBUG` desactivado en producción | ❌ | ✅ vía `LOG_LEVEL` env |

---

### Lo que queda disponible para el equipo

Para cualquier microservicio nuevo o existente, el uso correcto es:

```typescript
import { SafeLogger } from "../lib/safeLogger";

const log = new SafeLogger("mi-servicio");

// En el boundary de entrada (RabbitMQ, CoAP, HTTP):
log.info("rabbit_message_received", { type: "CMD", device: deviceId });

// Con correlación de traza:
const tracedLog = log.withTrace(msg.correlationId, msg.messageId);
tracedLog.info("rabbit_message_processed", { handler: "handleOutput" });

// Errores — solo el mensaje, nunca el objeto completo:
log.error("exception_unhandled", error, { context: "handleCLOutput" });
```
