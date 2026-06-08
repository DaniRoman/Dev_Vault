# Task: KNT-2373 — status-12830/device: initializeCache y alarmCount

---

## 1. Metadata

| Field | Value |
|---|---|
| Task name | Verificar comportamiento de initializeCache en status-12830 |
| Owner | daniel.roman@ako.com |
| Created | 2026-06-04 |
| Last updated | 2026-06-04 |
| Status | In progress |
| Current phase | Phase 0 — Analysis |
| Repository area | `src/micros/status-12830/`, `src/util/mongooseCache.ts` |
| Base branch | master |
| Related ticket | KNT-2373 |

---

## 2. Objective

`status-12830/device` llama a `initializeCache('micros-status-device-12830')`. Determinar qué colecciones cachea y con qué TTL, y si el campo `alarmCount` del device en MongoDB podría leerse desactualizado.

---

## 4. Background

- **Current behavior:** `handleInputStatus` llama a `alarmRepo.countActiveByDevice(deviceId)` antes de actualizar el device. Esa query usa `model.count().exec()`.
- **Riesgo planteado:** Si `initializeCache` cachea la colección `alarm12830`, el `alarmCount` podría ser stale durante el TTL del caché.
- **Conclusión:** No hay riesgo. Ver Phase 0.

---

## 5. Source of truth

| Type | Path | Why it matters |
|---|---|---|
| Cache impl. | [src/util/mongooseCache.ts](../../src/util/mongooseCache.ts) | Define qué operaciones y modelos se cachean |
| AlarmRepository | [src/micros/status-12830/st-lib/repositories/AlarmRepository.ts](../../src/micros/status-12830/st-lib/repositories/AlarmRepository.ts) | Ejecuta la query `count` sobre `alarm12830` |
| Status microservice | [src/micros/status-12830/device/microservice.ts](../../src/micros/status-12830/device/microservice.ts) | Llama a `initializeCache` y usa `alarmCount` |

---

## 8. Phases

| Phase | Name | Goal | Status |
|---|---|---|---|
| 0 | Analysis | Verificar si alarmCount puede ser stale por caché | Done |

---

## 9. Decisions

| Date | Decision | Reason |
|---|---|---|
| 2026-06-04 | `alarmCount` no se cachea | `count()` genera op `"count"`. `REDIS_CACHE_OPS` default es `"find,findOne"` (hardcodeado en `mongooseCache.ts:65`). `count` no está → siempre va a MongoDB. |

---

## 10. Implementation log

### 2026-06-04 — Phase 0: Analysis

#### Summary

- `initializeCache` solo inicializa un logger — no configura qué se cachea.
- La lógica real está en `mongoose.Query.prototype.exec` (monkey-patch en `mongooseCache.ts:51`).
- El caché solo actúa si `USE_REDIS_CACHE=true` **y** el modelo está en `REDIS_CACHE_MODELS` **y** la operación está en `REDIS_CACHE_OPS`.
- `REDIS_CACHE_OPS` default hardcodeado: `"find,findOne"` (`mongooseCache.ts:65`).
- `AlarmRepository.countActiveByDevice` usa `model.count()` → op `"count"` → **no cacheable por defecto**.
- Conclusión: `alarmCount` siempre se lee directo de MongoDB. No hay riesgo de valor desactualizado.

#### Files inspected

- `src/util/mongooseCache.ts`
- `src/micros/status-12830/st-lib/repositories/AlarmRepository.ts`
- `src/micros/status-12830/device/microservice.ts`

#### Tests añadidos

**Archivo:** `tests/micros/status-12830/st-lib/repositories/AlarmRepository.test.ts`

**Qué prueban:**
- **Test 1:** Dos llamadas consecutivas devuelven valores distintos (3 → 4). Si el caché interviniera, la segunda devolvería 3 (el valor guardado). El hecho de que devuelva 4 demuestra que siempre va al modelo.
- **Test 2:** El método `count().exec()` del modelo es invocado exactamente una vez por cada llamada a `countActiveByDevice`. Nunca se salta ninguna ejecución.
- **Test 3:** Si la query falla, el repositorio devuelve 0 sin lanzar excepción — comportamiento defensivo correcto.

**Qué necesitan:**
- Nada externo. Solo el código del repositorio y un mock JS plano del modelo.

**Qué NO necesitan — y por qué:**
- **Redis real:** no hace falta. El mock es un objeto JS plano `{ count: () => ({ exec: () => ... }) }`, no un modelo Mongoose real. El monkey-patch de `mongooseCache.ts` actúa sobre `mongoose.Query.prototype.exec`, que no interviene en objetos planos. Por tanto, Redis no llega a ser consultado en ningún momento del test.
- **MongoDB real:** tampoco. El modelo está mockeado, no hay conexión a base de datos.

**Limitación conocida:** precisamente porque el mock bypasea Mongoose, estos tests no ejercen el layer de caché Redis directamente. La prueba de que `alarm12830` no se cachea viene del análisis estático del código (`REDIS_CACHE_MODELS` no incluye `alarm12830` y `REDIS_CACHE_OPS` no incluye `count`). Los tests validan el comportamiento del repositorio, no el caché.

**Resultado:** `npx jest AlarmRepository.test.ts --no-coverage` → ✅ 3/3 PASSED

#### Test de integración añadido

**Archivo:** `tests/micros/status-12830/st-lib/repositories/AlarmRepository.integration.test.ts`

**Qué prueba:**
- **Test 1:** Inserta 3 alarmas → cuenta 3. Inserta 1 más directo en MongoDB → cuenta 4. Si el caché hubiera intervenido devolvería 3 (stale). El 4 confirma que siempre va a MongoDB.
- **Test 2:** El mock de `redis.get` y `redis.setex` nunca son llamados. Prueba que alarm12830 no toca Redis en ningún momento.
- **Test 3:** Solo cuenta alarmas `active=true` con `priority != unassigned`.
- **Test 4:** Devuelve 0 cuando no hay alarmas.

**Qué necesita:**
- MongoDB corriendo en `localhost:27017` (el contenedor de desarrollo). Usa la base de datos `test-alarm-integration`, que se limpia al terminar.
- El monkey-patch de `mongooseCache` activo (se importa en el test para aplicarlo).

**Qué NO necesita:**
- Redis real: `alarm12830` no está en `REDIS_CACHE_MODELS`, así que nunca llega a Redis. Se mockea solo para verificar que no se llama.

**Evidencia en los logs:**
El monkey-patch imprime `[CACHE SKIP] Model: alarm12830, Op: count - Not in allowed models or operations` en cada llamada — confirmación en runtime de que el caché lo salta.

**Resultado:** `npx jest AlarmRepository.integration.test.ts --no-coverage` → ✅ 4/4 PASSED

#### Test de integración con Redis real añadido

**Archivo:** `tests/micros/status-12830/st-lib/repositories/AlarmRepository.integration-redis.test.ts`

**Qué prueba:**
- **Test 1:** Igual que el test anterior — 3 alarmas → count 3, inserta 1 más → count 4. Con Redis real activo.
- **Test 2:** Tras llamar a `countActiveByDevice`, se consulta Redis directamente con `keys("alarm12830-*")`. Resultado: array vacío — nunca se escribió ninguna clave.
- **Test 3:** Filtros activos/unassigned correctos.

**Qué necesita:**
- MongoDB en `localhost:27017` y Redis en `localhost:6379` (contenedores de desarrollo).
- Sin mocks — conexiones reales a ambos servicios.

**Evidencia en los logs:**
- `[Redis] Connection established` — Redis real conectado.
- `[CACHE SKIP] Model: alarm12830, Op: count` — el monkey-patch salta la query en cada llamada.
- `keys("alarm12830-*")` devuelve `[]` — Redis nunca almacenó nada para alarm12830.

**Resultado:** `npx jest AlarmRepository.integration-redis.test.ts --no-coverage --verbose` → ✅ 3/3 PASSED — salida limpia, sin conexiones abiertas.

#### Next step

Pendiente de instrucciones del usuario para continuar.
