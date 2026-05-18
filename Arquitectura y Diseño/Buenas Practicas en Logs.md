Sí. En microservicios, la mejora más rentable suele ser **dejar de tratar los logs como texto libre** y pasar a un modelo gobernado: **eventos estructurados, sanitizados, con niveles coherentes, filtrado en puntos comunes y reducción antes de ingestar**.

## 1. Objetivo técnico

La meta no es “loggear menos” sin criterio. Es:

1. **Conservar señal operativa y de seguridad.**
2. **Reducir ruido, coste e I/O.**
3. **Evitar fugas de datos sensibles.**
4. **Correlacionar logs entre microservicios.**
5. **Centralizar reglas para no tocar 40 servicios a mano.**


OWASP recomienda que el logging sea consistente dentro de la aplicación y también entre aplicaciones, y recalca que no conviene loggear ni demasiado ni demasiado poco; el volumen debe derivarse del propósito del log. ([cheatsheetseries.owasp.org](https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html "Logging - OWASP Cheat Sheet Series"))

---

# 2. Arquitectura recomendada

Piensa en 4 capas:

```text
Código del microservicio
   ↓
Librería / middleware común de logging
   ↓
Pipeline de logs: OpenTelemetry Collector / Fluent Bit / Vector / Logstash
   ↓
Backend: ELK / Loki / Datadog / Cloud Logging / Splunk / S3 / SIEM
```

La clave es que las reglas importantes vivan en **pocos puntos globales**, no en cada `logger.info(...)`.

## Puntos globales donde intervenir

### A. Librería compartida de logging

Crear un módulo común, por ejemplo:

```text
company-logging
company-observability
platform-logging
```

Debe encargarse de:

- Formato JSON.
- Campos obligatorios.
- Sanitización.
- Enriquecimiento con `trace_id`, `span_id`, `request_id`, `service`, `version`, `environment`.
    
- Política de niveles.
    
- Helpers para eventos de negocio y seguridad.
    
- Bloqueo o warning si alguien intenta loggear campos prohibidos.
    

OpenTelemetry recomienda integrar logs con el resto de la observabilidad y propagar contexto como `trace_id` y `span_id`, porque en sistemas distribuidos los logs aislados pierden mucho valor. ([OpenTelemetry](https://opentelemetry.io/docs/specs/otel/logs/ "OpenTelemetry Logging | OpenTelemetry"))

---

### B. Middleware HTTP global

Un único middleware por servicio para requests entrantes:

Debe generar o propagar:

```json
{
  "request_id": "...",
  "trace_id": "...",
  "method": "GET",
  "route": "/orders/{id}",
  "status": 200,
  "duration_ms": 43,
  "client_type": "internal",
  "user_id_hash": "..."
}
```

Importante: usar **ruta normalizada**, no URL completa con IDs o query params sensibles.

Mal:

```text
GET /users/123456789?token=abc
```

Bien:

```text
GET /users/{id}
```

---

### C. Handler global de excepciones

Regla: **una excepción se loggea una vez**, normalmente en el borde del servicio.

Evitar esto:

```text
Repository logs error
Service logs same error
Controller logs same error
Middleware logs same error
```

Eso multiplica volumen y ensucia el diagnóstico.

Mejor:

- Las capas internas lanzan excepción con contexto seguro.
    
- El handler global registra una entrada final.
    
- El log incluye causa, tipo de error, operación y correlación.
    
- Stacktrace solo en `ERROR`, no en todos los `WARN`.
    

---

### D. Interceptores de clientes externos

Centralizar logs de:

- HTTP clients.
    
- gRPC clients.
    
- Kafka/Rabbit consumers/producers.
    
- Jobs programados.
    
- Accesos a proveedores externos.
    

Ejemplo de evento útil:

```json
{
  "event": "external_call_completed",
  "target_service": "payment-provider",
  "operation": "authorize_payment",
  "status": 502,
  "duration_ms": 812,
  "retry_count": 2,
  "trace_id": "..."
}
```

No loggear payloads completos por defecto.

---

### E. Pipeline de ingesta

Aquí puedes filtrar, transformar, enriquecer, dropear, rutear y aplicar retención distinta. OpenTelemetry Collector, por ejemplo, permite usar procesadores para transformar, filtrar y enriquecer telemetría antes de enviarla a backends, lo cual se usa para calidad de datos, gobernanza, coste y seguridad. ([OpenTelemetry](https://opentelemetry.io/docs/collector/components/processor/ "Processors | OpenTelemetry"))

Este punto es muy potente porque permite aplicar reglas sin redeployar todos los servicios.

Ejemplos:

- Dropear `/health`, `/metrics`, `/ready`.
    
- Reducir logs `INFO` repetitivos.
    
- Enviar logs de auditoría al SIEM.
    
- Enviar logs DEBUG solo a almacenamiento barato.
    
- Enmascarar campos que escaparon de la app.
    
- Rutear `ERROR` a backend caliente y `INFO` a almacenamiento frío.
    

---

# 3. Formato: JSON estructurado, no texto libre

Evita logs como:

```text
User 123 bought product 456 with card 4111111111111111
```

Usa eventos estructurados:

```json
{
  "timestamp": "2026-05-15T10:15:30.123Z",
  "level": "INFO",
  "service.name": "orders-api",
  "service.version": "1.23.4",
  "environment": "prod",
  "event": "order_created",
  "order_id": "ord_123",
  "user_id_hash": "hmac_sha256:9f2a...",
  "amount": 49.90,
  "currency": "EUR",
  "trace_id": "abc...",
  "span_id": "def..."
}
```

Google Cloud explica que los logs estructurados permiten consultar campos JSON concretos e indexar campos específicos, a diferencia del texto libre. ([Google Cloud Documentation](https://docs.cloud.google.com/logging/docs/structured-logging "Structured logging  |  Cloud Logging  |  Google Cloud Documentation")) OpenTelemetry también define convenciones semánticas para usar nombres comunes de atributos en logs, métricas, trazas, perfiles y recursos. ([OpenTelemetry](https://opentelemetry.io/docs/concepts/semantic-conventions/ "Semantic Conventions | OpenTelemetry"))

---

# 4. Campos mínimos recomendados

Yo definiría un contrato base así:

```json
{
  "timestamp": "...",
  "level": "INFO|WARN|ERROR|DEBUG",
  "message": "...",
  "event": "business_or_technical_event_name",
  "service.name": "...",
  "service.version": "...",
  "environment": "dev|test|pre|prod",
  "trace_id": "...",
  "span_id": "...",
  "request_id": "...",
  "tenant_id": "...",
  "user_id_hash": "...",
  "route": "...",
  "method": "...",
  "status_code": 200,
  "duration_ms": 123,
  "error.type": "...",
  "error.code": "...",
  "security": false
}
```

OWASP recomienda que cada evento tenga suficiente información para el análisis posterior y resume la idea como registrar **cuándo, dónde, quién y qué**. También recomienda usar un identificador de interacción para vincular eventos relacionados con una misma interacción. ([cheatsheetseries.owasp.org](https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html "Logging - OWASP Cheat Sheet Series"))

---

# 5. Política de niveles

Una mala política de niveles es una causa enorme de coste.

## ERROR

Solo cuando:

- La operación ha fallado.
    
- Requiere atención o afecta al usuario/sistema.
    
- Hay excepción no controlada.
    
- Un proveedor externo falla después de retries.
    
- Hay pérdida de datos, inconsistencia o degradación fuerte.
    

Ejemplo:

```json
{
  "level": "ERROR",
  "event": "payment_authorization_failed",
  "error.type": "ExternalProviderTimeout",
  "payment_provider": "x",
  "retry_count": 3
}
```

## WARN

Para condiciones anómalas pero recuperables:

- Retry intermedio.
    
- Circuit breaker half-open.
    
- Latencia por encima de umbral.
    
- Fallback activado.
    
- Datos incompletos pero procesables.
    

No usar `WARN` para flujo normal.

## INFO

Para eventos relevantes de negocio o ciclo de vida:

- Pedido creado.
    
- Pago autorizado.
    
- Usuario autenticado.
    
- Job finalizado.
    
- Integración completada.
    
- Cambio de estado importante.
    

No usar `INFO` para cada línea interna del método.

## DEBUG

Solo para diagnóstico temporal.

Recomendación práctica: permitir DEBUG dinámico con TTL, por ejemplo 15, 30 o 60 minutos, pero no dejarlo fijo en producción.

## TRACE

Deshabilitado en producción salvo casos muy controlados.

---

# 6. Qué NO loggear

OWASP recomienda no registrar directamente datos como identificadores de sesión, access tokens, PII sensible, contraseñas, connection strings, claves de cifrado, secretos, datos bancarios o de tarjetas, y sugiere eliminar, enmascarar, sanitizar, hashear o cifrar cuando aplique. ([cheatsheetseries.owasp.org](https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html "Logging - OWASP Cheat Sheet Series"))

Lista práctica de campos prohibidos:

```text
password
passwd
pwd
secret
token
access_token
refresh_token
id_token
authorization
cookie
set-cookie
api_key
apikey
private_key
session_id
jwt
credit_card
card_number
cvv
iban
ssn
dni
passport
connection_string
database_url
```

También evitar por defecto:

- Request body completo.
    
- Response body completo.
    
- Headers completos.
    
- Query string completa.
    
- Payloads de eventos.
    
- SQL con parámetros reales.
    
- Excepciones que incluyan datos de usuario.
    
- Objetos serializados completos.
    

---

# 7. Sanitización: estrategia correcta

No confiaría solo en regex al final. Haría defensa en capas.

## Capa 1: allowlist

El mejor enfoque: solo permitir campos conocidos.

Ejemplo:

```ts
logger.info("order_created", {
  order_id,
  amount,
  currency,
  user_id_hash
});
```

En vez de:

```ts
logger.info("order_created", orderObject);
```

## Capa 2: redacción por nombre de campo

Cualquier campo cuyo nombre coincida con la denylist se reemplaza:

```json
{
  "authorization": "[REDACTED]",
  "access_token": "[REDACTED]",
  "password": "[REDACTED]"
}
```

## Capa 3: redacción por patrón

Última línea de defensa:

- JWT.
    
- Bearer tokens.
    
- Emails, según caso.
    
- Tarjetas.
    
- IBAN.
    
- API keys.
    
- Cookies.
    
- UUIDs si son sensibles.
    
- Números largos.
    

## Capa 4: hashing para correlación

Cuando necesitas correlacionar sin exponer:

```text
user_id -> HMAC-SHA256(user_id, secret)
session_id -> HMAC-SHA256(session_id, secret)
email -> HMAC-SHA256(lowercase(email), secret)
```

Mejor HMAC que hash simple, porque un hash simple de emails o IDs puede ser vulnerable a diccionario.

## Capa 5: límites de tamaño

Imponer:

```text
max message length
max field length
max number of fields
max stacktrace length
max nested depth
```

Esto evita explosiones de volumen y también reduce riesgo de DoS vía logs. OWASP recomienda probar que el logging no pueda agotar recursos como disco o espacio transaccional, y probar fallos del mecanismo de logging. ([cheatsheetseries.owasp.org](https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html "Logging - OWASP Cheat Sheet Series"))

---

# 8. Reducción de volumen

Aquí está probablemente el mayor ahorro.

## A. Eliminar logs repetidos

Patrones típicos a eliminar:

```text
Entering method X
Leaving method X
Calling repository
Repository returned result
Mapping DTO
Validation passed
```

Eso debería estar en traces, métricas o no existir.

## B. No loggear health checks

Dropear o muestrear agresivamente:

```text
/health
/ready
/live
/metrics
/favicon.ico
```

## C. No loggear 2xx de bajo valor en todos los servicios

En microservicios, un request puede pasar por 8 servicios. Si cada uno escribe un access log `INFO`, tienes multiplicación inmediata.

Opciones:

- Access log completo solo en gateway/borde.
    
- Resumen por servicio solo si hay error, latencia alta o sampling.
    
- Métricas para conteos normales.
    
- Tracing para caminos distribuidos.
    

## D. Sampling

Regla inicial razonable:

```text
ERROR: 100%
WARN: 100% al inicio, luego revisar
Security/audit: 100%
INFO negocio crítico: 100%
INFO técnico repetitivo: 1-10%
DEBUG: 0% salvo activación temporal
Health checks: 0% o casi 0%
```

Usar sampling determinístico por `trace_id`, `request_id` o `user_id_hash`, no completamente aleatorio, para conservar trazabilidad de una misma interacción.

Datadog describe la optimización de volumen como filtrar, samplear, enriquecer y rutear logs antes de almacenamiento/análisis para reducir coste y ruido sin perder visibilidad relevante. ([Datadog](https://www.datadoghq.com/knowledge-center/log-optimization/ "How to Optimize Log Volume and Reduce Noise at Scale | Datadog"))

## E. Rate limiting de logs

Para errores repetidos:

```text
Máximo 10 eventos iguales por minuto por servicio/ruta/error_type
Luego emitir resumen:
"same error suppressed 1243 times in last 60s"
```

Esto evita tormentas de logs cuando un proveedor cae.

## F. Deduplicación de stacktraces

Si el mismo error ocurre 10.000 veces, no necesitas 10.000 stacktraces completos.

Guardar:

- Primer stacktrace completo.
    
- Siguientes eventos resumidos.
    
- Contador agregado.
    

## G. Convertir logs en métricas

Muchos logs existen porque no hay métricas.

En vez de:

```text
INFO stock checked for product X
```

Usar métrica:

```text
stock_check_total{result="success"}
stock_check_duration_ms
```

Logs para eventos significativos; métricas para conteos de alta frecuencia.

---

# 9. Clasificación de logs por tipo

Separaría los logs en categorías:

## 1. Operacionales

Para debugging y estabilidad:

```text
service_started
external_call_failed
job_completed
cache_unavailable
circuit_breaker_opened
```

## 2. Seguridad

OWASP recomienda loggear eventos como fallos de validación, autenticación, autorización, gestión de sesión, errores de aplicación, arranques/paradas, uso de funciones de alto riesgo y actividades sospechosas de lógica de negocio. ([cheatsheetseries.owasp.org](https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html "Logging - OWASP Cheat Sheet Series"))

Ejemplos:

```text
authentication_failed
authorization_denied
jwt_validation_failed
rate_limit_exceeded
suspicious_business_flow
admin_privilege_used
```

## 3. Auditoría

No son para debugging, son para trazabilidad formal:

```text
user_created
role_changed
payment_refunded
invoice_exported
data_deleted
configuration_changed
```

Estos normalmente no se samplean.

## 4. Negocio

Eventos útiles para soporte o análisis:

```text
order_created
order_cancelled
payment_authorized
shipment_delayed
```

## 5. Diagnóstico temporal

DEBUG/TRACE activable por:

```text
service
tenant
correlation_id
user hash
endpoint
time window
```

---

# 10. Retención diferenciada

No todos los logs merecen el mismo almacenamiento.

Ejemplo:

```text
ERROR/WARN: 30-90 días en backend caliente
Security/audit: 180-365+ días según compliance
INFO negocio: 30-90 días
INFO técnico: 7-15 días
DEBUG temporal: 1-3 días
Raw logs baratos: S3/blob storage si hace falta
```

OWASP también advierte que los logs no deben conservarse menos de lo requerido ni más allá del periodo necesario, porque obligaciones legales, regulatorias o contractuales pueden afectar esos plazos. ([cheatsheetseries.owasp.org](https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html "Logging - OWASP Cheat Sheet Series"))

---

# 11. Ejemplo de política concreta

Puedes definir algo así para todos los equipos:

```text
1. Todo log debe ser JSON.
2. Todo log debe tener event, level, service.name, environment, trace_id/request_id.
3. Prohibido loggear bodies completos salvo allowlist explícita.
4. Prohibido loggear headers completos.
5. Prohibido loggear tokens, passwords, cookies, secrets o connection strings.
6. Los logs de auditoría no se samplean.
7. Los logs de seguridad no se samplean.
8. Los access logs 2xx internos se samplean o se emiten solo en gateway.
9. Los health checks se dropean.
10. DEBUG en producción requiere TTL.
11. Cada excepción se loggea una sola vez.
12. Todo campo desconocido en eventos sensibles se descarta o redacted.
13. Todo cambio de nivel en producción queda auditado.
```

---

# 12. Ejemplo de wrapper conceptual

```ts
const SENSITIVE_KEYS = [
  "password",
  "token",
  "access_token",
  "refresh_token",
  "authorization",
  "cookie",
  "secret",
  "api_key",
  "session_id",
  "connection_string"
];

function sanitize(value: unknown): unknown {
  if (value === null || value === undefined) return value;

  if (typeof value === "string") {
    return value
      .replace(/Bearer\s+[A-Za-z0-9._-]+/g, "Bearer [REDACTED]")
      .replace(/\r|\n/g, " ")
      .slice(0, 4000);
  }

  if (Array.isArray(value)) {
    return value.slice(0, 50).map(sanitize);
  }

  if (typeof value === "object") {
    const result: Record<string, unknown> = {};

    for (const [key, raw] of Object.entries(value)) {
      const normalized = key.toLowerCase();

      if (SENSITIVE_KEYS.some(s => normalized.includes(s))) {
        result[key] = "[REDACTED]";
      } else {
        result[key] = sanitize(raw);
      }
    }

    return result;
  }

  return value;
}

function logInfo(event: string, fields: Record<string, unknown>) {
  baseLogger.info({
    level: "INFO",
    event,
    ...sanitize(fields),
    service: process.env.SERVICE_NAME,
    environment: process.env.ENVIRONMENT
  });
}
```

Pero esto debería ser **última defensa**, no permiso para loggear objetos enteros.

---

# 13. Ejemplo de log malo vs bueno

## Malo

```text
INFO Payment request received: {"card":"4111111111111111","cvv":"123","token":"abc..."}
```

Problemas:

- Tiene tarjeta.
    
- Tiene CVV.
    
- Tiene token.
    
- Es texto libre.
    
- Difícil de consultar.
    
- Riesgo legal y de seguridad.
    

## Bueno

```json
{
  "level": "INFO",
  "event": "payment_authorization_requested",
  "service.name": "payments-api",
  "payment_method": "card",
  "card_brand": "visa",
  "card_last4": "1111",
  "amount": 49.90,
  "currency": "EUR",
  "user_id_hash": "hmac_sha256:...",
  "trace_id": "..."
}
```

---

# 14. Configuración central: qué mover fuera del código

Idealmente configurable sin redeploy:

```yaml
logging:
  level:
    root: INFO
    services:
      payments-api: INFO
      orders-api: INFO

  debug_overrides:
    enabled: true
    ttl_minutes: 30
    allowed_scopes:
      - trace_id
      - user_id_hash
      - route
      - tenant_id

  sampling:
    http_2xx_internal: 0.05
    health_checks: 0.0
    debug: 0.0
    errors: 1.0
    audit: 1.0
    security: 1.0

  redaction:
    mode: strict
    unknown_fields_in_sensitive_events: drop
```

---

# 15. Pipeline de ingesta: reglas útiles

En el collector/pipeline:

```text
1. Parsear JSON.
2. Validar schema mínimo.
3. Añadir metadata: env, region, host, service.
4. Redactar campos sensibles.
5. Dropear ruido conocido.
6. Samplear eventos de bajo valor.
7. Rutear:
   - ERROR/WARN → backend caliente
   - audit/security → SIEM
   - INFO bajo valor → storage barato o drop
8. Medir:
   - logs recibidos
   - logs dropeados
   - % reducción
   - errores de parsing
   - top servicios por volumen
```

Datadog recomienda medir reducción, volumen dropeado y ahorro downstream para demostrar impacto de la optimización. ([Datadog](https://www.datadoghq.com/knowledge-center/log-optimization/ "How to Optimize Log Volume and Reduce Noise at Scale | Datadog"))

---

# 16. Métricas que deberías crear para controlar la mejora

Antes de cambiar, mide baseline:

```text
logs_por_servicio_por_día
logs_por_nivel
logs_por_event
GB/día ingeridos
coste/día
top 20 endpoints por volumen
top 20 mensajes repetidos
% logs sin trace_id
% logs no JSON
% logs con campos sensibles detectados
errores de parsing
ratio ERROR/INFO/WARN
```

Después de la mejora:

```text
reducción de GB/día
reducción de eventos/día
coste evitado
servicios migrados al contrato común
eventos bloqueados por sanitización
eventos dropeados por sampling
tiempo medio de búsqueda/debug
```

---

# 17. Plan de implementación recomendado

## Fase 1 — Diagnóstico rápido

Extraer top offenders:

```text
Top servicios por volumen
Top mensajes repetidos
Top endpoints ruidosos
Top logs con payloads grandes
Top errores duplicados
Campos sensibles detectados
```

Resultado esperado: probablemente 10-20 patrones generan 70-80% del volumen.

---

## Fase 2 — Definir estándar

Crear un documento corto:

```text
Logging Standard v1
- formato
- campos obligatorios
- niveles
- campos prohibidos
- eventos audit/security
- sampling
- retención
- ejemplos
```

---

## Fase 3 — Librería compartida

Implementar:

```text
logger seguro
sanitizer
enriquecimiento trace/request
helpers de audit/security/business events
limitador de tamaño
tests unitarios
```

---

## Fase 4 — Middleware global

Añadir:

```text
HTTP incoming middleware
exception handler
HTTP/gRPC client interceptor
message consumer wrapper
job wrapper
```

---

## Fase 5 — Pipeline central

Aplicar:

```text
drop health checks
redaction final
sampling bajo valor
routing por tipo
deduplicación
retención diferenciada
```

---

## Fase 6 — Guardrails

Añadir controles en CI:

```text
detectar logger.info(object)
detectar Authorization/Cookie en logs
detectar request.body en logs
detectar console.log
detectar printStackTrace
detectar logs sin event
```

También tests:

```text
"no secret appears in logs"
"tokens are redacted"
"large payload is truncated"
"exception logs contain trace_id"
```

OWASP recomienda verificar que el logging funcione como se especificó, que los eventos se clasifiquen de forma consistente, que no haya inyección en logs y que los fallos del mecanismo de logging no rompan la aplicación. ([cheatsheetseries.owasp.org](https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html "Logging - OWASP Cheat Sheet Series"))

---

# 18. Antipatrones a eliminar

```text
logger.info("request: {}", request)
logger.info("response: {}", response)
logger.error("error", ex) en 4 capas distintas
logger.warn para flujo normal
logger.info dentro de loops masivos
logger.debug permanente en producción
loggear payloads de Kafka completos
loggear SQL con parámetros sensibles
loggear headers completos
loggear tokens para "debug rápido"
usar string concatenation en vez de campos estructurados
```

---

# 19. Recomendación práctica inicial

Yo empezaría con este backlog:

1. **Crear contrato JSON mínimo.**
    
2. **Definir campos prohibidos.**
    
3. **Implementar sanitizer común.**
    
4. **Meter middleware HTTP + exception handler.**
    
5. **Propagar `trace_id` / `request_id`.**
    
6. **Dropear health checks y access logs 2xx internos de bajo valor.**
    
7. **Eliminar logs duplicados de excepciones.**
    
8. **Activar sampling para INFO técnico repetitivo.**
    
9. **Separar audit/security de logs normales.**
    
10. **Añadir dashboard de volumen por servicio/evento.**
    

Con eso normalmente ya se consigue una reducción fuerte sin perder observabilidad.