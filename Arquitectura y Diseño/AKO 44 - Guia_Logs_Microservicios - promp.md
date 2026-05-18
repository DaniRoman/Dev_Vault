## Bloque 3 — Prompt para la IA encargada del módulo de logs

```
## Contexto del proyecto

Trabajas sobre un sistema de microservicios en TypeScript/Node.js.
Los servicios piloto son [NOMBRE_SERVICIO_A] y [NOMBRE_SERVICIO_B].
El logger actual se encuentra en: `src/lib/logger.ts` (o la ruta equivalente de cada servicio).
La arquitectura recibe tráfico por tres vías: HTTP, RabbitMQ y conexiones directas con dispositivos.

---

## Objetivo

Mejorar el sistema de logging de los dos microservicios indicados para:
1. Eliminar exposición de datos sensibles.
2. Reducir ruido de logs innecesarios.
3. Estandarizar formato y campos mínimos en todos los logs.
4. Centralizar sanitización en un punto común (SafeLogger).
5. Dejar un patrón reutilizable para otros servicios.

---

## Reglas de logging que debes aplicar

- Todos los logs deben ser JSON estructurado.
- Todo log debe tener el campo `event` con un nombre estable (ver catálogo más abajo).
- Todo log debe incluir: `timestamp`, `level`, `event`, `service`, `environment`.
- Incluir `trace_id` / `request_id` / `message_id` según el tipo de entrada.
- Prohibido loggear objetos completos: request, response, body, headers, user.
- Prohibido loggear payloads completos.
- Una excepción se loggea una sola vez, en el boundary. Las capas internas la propagan sin loggear.
- `DEBUG` desactivado en producción (`LOG_LEVEL=INFO`).
- Los logs de health checks (`/health`, `/ping`, `/status`) deben eliminarse o excluirse.
- Los logs dentro de bucles deben eliminarse o moverse a `DEBUG` con sampling.

Niveles permitidos:
- `DEBUG`: diagnóstico temporal, solo desarrollo.
- `INFO`: eventos importantes y esperados.
- `WARN`: anomalías recuperables (retry, timeout, fallback).
- `ERROR`: fallos reales de operación no recuperados.

Catálogo de nombres de evento permitidos:

http_request_completed, http_request_failed,
rabbit_message_received, rabbit_message_processed, rabbit_message_processing_failed, rabbit_message_published,
device_connected, device_disconnected, device_message_received, device_message_rejected,
external_http_call_completed, external_http_call_failed,
validation_failed, command_translated, command_sent_to_device
```

---

## Reglas de sanitización que debes aplicar

Implementar un sanitizer con el siguiente pipeline:
1. Redactar por nombre de campo: si el campo pertenece a la lista de campos sensibles, reemplazar su valor por `"[REDACTED]"`.
2. Redactar por patrón: aplicar regex sobre strings para detectar tokens, Bearer, passwords, etc., y reemplazar por `"[REDACTED]"`.
3. Truncar strings que superen 2 KB.
4. Limitar arrays a 20 elementos.
5. Limitar profundidad de objetos a 4 niveles.
6. Si el evento completo supera 8 KB: eliminar campos no esenciales, luego truncar `message` y `stacktrace`. Conservar siempre: `event`, `level`, `service`, `trace_id`, `error_type`.

Campos que siempre se redactan:
`password`, `token`, `access_token`, `refresh_token`, `id_token`, `authorization`, `cookie`, `set-cookie`, `api_key`, `secret`, `private_key`, `jwt`, `session_id`, `connection_string`, `database_url`, `card_number`, `cvv`, `iban`, `dni`, `passport`

---

## Qué archivos y patrones debes buscar

Busca en el código fuente de los dos servicios:

Patrones de código problemáticos:
- `logger.info(request)`, `logger.info(response)`, `logger.info(body)`, `logger.info(headers)`
- `logger.error(e)` repetido en múltiples capas del mismo flujo
- `console.log(`, `console.error(`, `console.warn(`
- `printStackTrace`
- Logs dentro de bucles (`for`, `forEach`, `while`, etc.)
- Logs en endpoints de health check

Palabras clave de riesgo en el contenido:
- `Authorization`, `Bearer`, `password`, `token`, `secret`, `Cookie`, `body`, `payload`, `headers`

Archivos donde habitualmente se concentran estos problemas:
- Controladores HTTP
- Consumers de RabbitMQ
- Handlers de conexión con dispositivos
- Clientes HTTP hacia servicios externos
- Middleware de errores

---

## Qué cambios debes proponer

Para cada log problemático encontrado, proponer:
1. Si debe eliminarse, moverse a `DEBUG`, mantenerse o consolidarse con otro log.
2. El nombre de evento estructurado que debería tener (del catálogo).
3. Los campos específicos que debe incluir (solo los seguros y relevantes).
4. El nivel correcto para ese evento.

---

## Qué debes implementar

1. **Sanitizer**: función `sanitize(fields: object): object` que aplique el pipeline completo descrito arriba.
2. **SafeLogger**: clase o módulo que envuelva al logger actual y aplique automáticamente: campos comunes del envelope, sanitizer, truncado, reglas de nivel. API: `safeLogger.info(event, fields)`, `safeLogger.warn(event, fields)`, `safeLogger.error(event, error, fields)`, `safeLogger.debug(event, fields)`.
3. **Middleware HTTP**: interceptor que loggee `http_request_completed` o `http_request_failed` en el boundary, capturando solo los campos permitidos.
4. **Consumer wrapper RabbitMQ**: wrapper que loggee `rabbit_message_processed` o `rabbit_message_processing_failed` en el boundary.
5. **Device connection wrapper**: wrapper que loggee `device_message_received` en el boundary, con `device_id_hash` en lugar del ID real.
6. **Handler global de excepciones**: punto único donde se loggean los errores no controlados de cada servicio.
7. **Reemplazar logs problemáticos**: sustituir en el código todos los patrones detectados por llamadas a `safeLogger` con los campos correctos.

---

## Qué debes evitar

- No loggear objetos completos como contexto.
- No loggear headers completos en ningún caso.
- No loggear el mismo error en más de una capa del mismo flujo.
- No usar `console.log` ni `printStackTrace`.
- No crear logs dentro de bucles sin sampling explícito.
- No inventar nombres de evento fuera del catálogo sin justificación.
- No incluir `device_id` real; usar siempre `device_id_hash`.
- No incluir stacktraces completos en producción en el campo `error_message`; usar `error_type` y `error_code`.
- No romper el logger existente si puede adaptarse; preferir extensión a reescritura completa.

---

Primero verifico si hay más loggers activos además de `LoggerHelper`:

`WinstonLog` devuelve vacío — cero instancias activas en los dos micros. Confirmado: **`LoggerHelper` de `src/micros/lib/logger.ts` es el único logger activo** en `translator-12830` y `cl-12830`.

Ahora hago el inventario completo de problemas antes de tocar nada:

Tengo el mapa completo. Confirmado: **`LoggerHelper` de `src/micros/lib/logger.ts` es el único logger activo** en los dos micros. Los archivos de `cl-12830/cl-lib/logger/` (`WinstonLog`, `ConsoleLogger`) no se instancian en ningún sitio de los dos micros objetivo.

---

## Inventario de problemas encontrados

**`console.log` activos (12 ocurrencias):**
- `authentication.ts:137` — loggea `body + message` completo → **crítico**
- `checkIntegrity.ts:89` — loggea payload decodificado → **crítico**
- `microservice.abstract.ts:97,99,104,106,111,113,132,482` — conexión AMQP sin usar `this.log`
- `perte-coap/microservice.ts:401` — ruido puro

**`JSON.stringify` de objetos completos prohibidos:**
- `authentication.ts:54,66,234` — `message.payload`, `deviceInfo.body`, `deviceInfo` completo
- `perte-coap/microservice.ts:244` — `JSON.stringify(request)` CoAP completo → **crítico**
- `translator-12830/microservice.ts` — 10 ocurrencias de `parsedMsg/toCLMessage/message` completos
- `InputProcessor.ts:43,66,87,108,129` — `parsedMessage.parsedData` completo en cada mensaje

**Ruido y noise:**
- `perte-coap/microservice.ts:243,245` — delimitadores `#########################`
- Logs en parsers que se ejecutan por cada mensaje procesado

---

## Plan de implementación (paso a paso)

```
Paso 1 → Mejorar logger.ts (nivel configurable, base limpia)
Paso 2 → Crear sanitizer.ts
Paso 3 → Crear safeLogger.ts  
Paso 4 → Limpiar cl-12830 (abstract, perte-coap, authentication, checkIntegrity)
Paso 5 → Limpiar translator-12830 (microservice.ts, InputProcessor, InputValidator)
```

Empezamos con el **Paso 1**.

**Paso 1 — Mejorar `logger.ts`:**

**Paso 2 — `sanitizer.ts`** (sobreescribimos el que se creó antes en limpio):

**Paso 3 — `safeLogger.ts`:**

**Paso 4 — Limpiar `cl-12830`.** Empiezo por `microservice.abstract.ts` (los 8 `console.log` de conexión AMQP):

Ahora `perte-coap/microservice.ts` — el `JSON.stringify(request)` completo (crítico) y los delimitadores:

Ahora `authentication.ts` — el `console.log(body + message)` crítico y los `JSON.stringify` de payload/deviceInfo:

Ahora `checkIntegrity.ts` — el `console.log(decode(payload))` crítico:

**Paso 5 — Limpiar `translator-12830`.** Primero el `microservice.ts` (10 `JSON.stringify` de payloads completos):

Ahora `InputProcessor.ts` — los `JSON.stringify(parsedMessage.parsedData)` que se ejecutan en cada mensaje:

Ahora `InputValidator.ts` — los `JSON.stringify(deviceInformation/payload)` que loggean datos sensibles:

Verificación final de los dos micros:

`console.log` activos: **cero** en los dos micros. Sin errores TypeScript.

---