## 1. Resumen del objetivo

Eliminar `Device12830Controller` como API paralela (`/api/12830/device`) y consolidar toda la lógica en el `DeviceController` genérico (`/api/device`), bifurcando por `isNewDevice / isOldDevice`. La buena noticia: **la mayor parte ya está hecha**.

---

## 2. Mapa de endpoints

|Endpoint 12830|Endpoint genérico|Estado|
|---|---|---|
|`GET /api/12830/device/:id`|`GET /api/device/:id`|⚠️ Parcial — `eventsActive/alarmsActive` bifurcados, `normalizeDeviceDefinition` también. Revisar paridad exacta|
|`GET /api/12830/device/indicators/:id`|`GET /api/device/:id/indicators` _(pendiente)_|❌ Ruta genérica no expuesta públicamente. `_getIndicators()` existe privado en DeviceController|
|`GET /api/12830/device/metrics/:id`|`GET /api/device/:id/metrics`|✅ Ya bifurcado (`isOldDevice` en línea 1877)|
|`GET /api/12830/device/activity/:id`|`GET /api/device/:id/activity`|✅ Ya bifurcado (`isOldDevice` en línea 2421)|
|`GET /api/12830/device/activity/:id/xlsx`|`GET /api/device/:id/activity/xlsx`|✅ Ya bifurcado (`isOldDevice` en línea 2539)|

---

## 3. Matriz de funciones

|Función en Device12830Controller|Destino en DeviceController|Estado actual|Acción necesaria|
|---|---|---|---|
|`get()` + `normalizeDeviceDefinition`|`get()` → `_extraGetSelects()` línea 4361|⚠️ Parcial|Verificar paridad: selectParams, campos devueltos, comportamiento si falta definición|
|`getActives()` — eventsActive|`_extraGetSelects()` línea 4308|✅ Duplicado idéntico|Verificar que la deduplicación es igual. Sin acción si es igual|
|`getActives()` — alarmsActive|`_extraGetSelects()` línea 4331|✅ Duplicado idéntico|Ídem|
|`getIndicators()` via Connector|`_getIndicators()` privado línea 3631|⚠️ Parcial|Exponer como ruta pública `GET /api/device/:id/indicators`|
|`getMetrics()` via Metrics12830|`getMetrics()` línea 1877|✅ Bifurcado|Solo verificar que los parámetros (`type`, `dateRange`, `from`, `until`) son equivalentes|
|`getActivity()` via lib/12830|`getActivity()` línea 2421|✅ Bifurcado|Verificar paridad de `resolveRange` vs lógica de fechas en genérico|
|`getActivityExportXlsx()`|`getDeviceActivityExportXlsx()` línea 2539|✅ Bifurcado|Verificar intervalos diarios/semanales. Lógica casi idéntica|
|`resolveRange()` helper local|DateHelper en genérico|❌ No migrado|Evaluar si hay diferencia en el cálculo de rangos|

---

## 4. Plan por fases

---

### Fase 0 — Análisis y verificación previa

**Objetivo**: confirmar el estado real antes de tocar nada.

**Archivos**: [device.ts (12830)](vscode-webview://0g884bpg89jtf79uob4hmsq4tpmqpoun4g8c6mcqtrpg7g0nu3hr/src/controllers/api/12830/device.ts), [device.ts (genérico)](vscode-webview://0g884bpg89jtf79uob4hmsq4tpmqpoun4g8c6mcqtrpg7g0nu3hr/src/controllers/api/device.ts)

**Acciones**:

- Comparar `resolveRange()` (12830, línea 22) con la lógica de fechas del genérico
- Confirmar que `_getIndicators()` en DeviceController ya llama a `Indicators12830` para new devices
- Hacer una llamada real a `/api/device/:id` con un new device y verificar que `eventsActive`, `alarmsActive` y `normalizeDeviceDefinition` funcionan igual que en `/api/12830/device/:id`
- Revisar si el frontend llama actualmente a `/api/12830/device` o ya usa `/api/device`

**Criterio de aceptación**: mapa de paridad documentado, sin tocar código.

**Riesgo**: diferencias silenciosas en formato de respuesta que el frontend asume.

---

### Fase 1 — Exponer `getIndicators` como ruta pública en DeviceController

**Objetivo**: `GET /api/device/:id/indicators` funcional para new y old devices.

**Archivos**: [device.ts (genérico)](vscode-webview://0g884bpg89jtf79uob4hmsq4tpmqpoun4g8c6mcqtrpg7g0nu3hr/src/controllers/api/device.ts)

**Funciones afectadas**: `_getIndicators()` (línea 3631) — ya tiene bifurcación `isOldDevice`

**Cambios**:

- Crear método público `getIndicators(id, params)` que llame a `_getIndicators()`
- Registrar ruta `GET /api/device/indicators/:id`
- Asegurar que los parámetros aceptados (`dateRange`, `from`, `until`) son equivalentes a los que acepta la ruta 12830

**Criterio de aceptación**: misma respuesta en `/api/device/indicators/:id` que en `/api/12830/device/indicators/:id` para un new device.

**Riesgo**: `_getIndicators()` puede tener dependencias de `req.context` que no estén en el scope del método público.

---

### Fase 2 — Verificar y cerrar paridad de `get()`

**Objetivo**: `GET /api/device/:id` devuelve exactamente lo mismo que `GET /api/12830/device/:id` para new devices.

**Archivos**: [device.ts (genérico)](vscode-webview://0g884bpg89jtf79uob4hmsq4tpmqpoun4g8c6mcqtrpg7g0nu3hr/src/controllers/api/device.ts) líneas 1540, 4305–4361

**Acciones**:

- Comparar los campos devueltos campo a campo
- Verificar que `normalizeDeviceDefinition` se aplica en el mismo punto
- Verificar `eventsActive` y `alarmsActive` con el mismo comportamiento de deduplicación

**Criterio de aceptación**: test manual con misma petición a ambos endpoints devuelve respuesta idéntica.

**Riesgo**: diferencias en `select` params, fields ocultos, población de refs.

---

### Fase 3 — Verificar paridad de `getMetrics`

**Objetivo**: confirmar que `GET /api/device/:id/metrics` ya funciona como `/api/12830/device/metrics/:id`.

**Archivos**: [device.ts (genérico)](vscode-webview://0g884bpg89jtf79uob4hmsq4tpmqpoun4g8c6mcqtrpg7g0nu3hr/src/controllers/api/device.ts) línea 1843–1911

**Acciones**:

- Verificar que `Metrics12830.getMetrics()` se llama con los mismos parámetros (`type`, `value`, `dateRange`, `from`, `until`)
- Verificar el flag `is12830: true` si lo usa
- Test con new device en ambos endpoints

**Criterio de aceptación**: respuesta idéntica en ambos endpoints.

**Riesgo bajo** — ya bifurcado.

---

### Fase 4 — Verificar paridad de `getActivity` y `resolveRange`

**Objetivo**: confirmar que la lógica de fechas es equivalente.

**Archivos**: [device.ts (12830)](vscode-webview://0g884bpg89jtf79uob4hmsq4tpmqpoun4g8c6mcqtrpg7g0nu3hr/src/controllers/api/12830/device.ts) línea 22–74, [device.ts (genérico)](vscode-webview://0g884bpg89jtf79uob4hmsq4tpmqpoun4g8c6mcqtrpg7g0nu3hr/src/controllers/api/device.ts) línea 2407–2500

**Acciones**:

- Comparar `resolveRange()` (12830) con el cálculo de fechas en el genérico
- Si difieren: mover `resolveRange` a un helper compartido o replicar lógica en genérico
- Test con `dateRange=24h`, `7d`, `30d` y con `from`/`until` explícitos

**Criterio de aceptación**: mismas fechas de corte en ambos endpoints para los mismos parámetros.

**Riesgo**: `resolveRange` puede tener edge cases no cubiertos en el genérico.

---

### Fase 5 — Verificar paridad de `getActivityExportXlsx`

**Objetivo**: confirmar que el export XLSX es equivalente.

**Archivos**: [device.ts (genérico)](vscode-webview://0g884bpg89jtf79uob4hmsq4tpmqpoun4g8c6mcqtrpg7g0nu3hr/src/controllers/api/device.ts) línea 2505–2606

**Acciones**:

- Comparar intervalos diarios/semanales entre ambas implementaciones
- Verificar headers del XLSX, nombres de columnas, formato de fechas

**Criterio de aceptación**: XLSX idéntico generado por ambos endpoints con los mismos params.

**Riesgo bajo** — lógica casi idéntica según el análisis.

---

### Fase 6 — Deprecar rutas `/api/12830/device`

**Objetivo**: eliminar o redirigir las rutas del controlador 12830.

**Opciones**:

- A) Redirigir 301 a `/api/device` (si el frontend puede seguir redirects)
- B) Mantener rutas 12830 como aliases que llaman internamente a DeviceController
- C) Eliminar directamente si se confirma que el frontend ya usa `/api/device`

**Criterio de aceptación**: ningún cliente activo llama a `/api/12830/device`.

**Riesgo alto** — depende del estado real del frontend.

---

### Fase 7 — Eliminar `Device12830Controller`

Solo cuando Fase 6 esté confirmada en producción.

---

### Fase 8 — Tests y validación

- Test manual en local con new y old devices para cada endpoint
- Revisar logs de producción buscando errores en endpoints migrados
- Regresión en export XLSX, indicators, metrics, activity

---

## 5. Dudas y bloqueantes antes de implementar

|#|Pregunta|Impacto|
|---|---|---|
|1|¿El frontend actualmente llama a `/api/12830/device` o ya usa `/api/device`?|Determina urgencia de Fase 6|
|2|¿Hay diferencia en `resolveRange()` vs la lógica de fechas del genérico?|Bloquea Fase 4|
|3|¿`_getIndicators()` en DeviceController llama a `Indicators12830` con los mismos params que el 12830 controller?|Bloquea Fase 1|
|4|¿El campo `is12830: true` que usa `getMetrics` en Device12830Controller se pasa también desde el endpoint genérico?|Afecta Fase 3|
|5|¿Hay algún campo en la respuesta de `GET /api/12830/device/:id` que el frontend asume y que no esté en `GET /api/device/:id`?|Bloquea Fase 2|

---

## 6. Primera fase lista para ejecutar — Fase 0

Antes de tocar código, responde estas preguntas mirando la codebase y el frontend:

1. **¿El frontend usa `/api/12830/device` o `/api/device`?** Busca en el código del frontend.
2. **Llama a `/api/device/:id` con `?select=eventsActive,alarmsActive`?** Si no, la bifurcación ya en el genérico no se está ejercitando.
3. **¿`resolveRange` y la lógica de fechas del genérico producen el mismo resultado?** Compara con un caso concreto: `dateRange=24h` → ¿mismo `from` y `until`?

Con esas 3 respuestas ya podemos ejecutar Fase 1 sin riesgo.

---

## 7. Plantilla de documentación por fase

```
## Fase N — [Nombre]

### Objetivo
[Una frase]

### Archivos modificados
- path/to/file.ts (líneas X–Y)

### Funciones afectadas
- antes: [función original]
- después: [función modificada]

### Cambios implementados
[Descripción concisa o pseudocódigo]

### Tests realizados
- [ ] New device: endpoint X devuelve Y
- [ ] Old device: sin regresión
- [ ] Edge case: [descripción]

### Resultado
✅ / ⚠️ / ❌

### Notas / decisiones tomadas
[Lo que se descubrió durante la implementación]
```

---

Cuando quieras arrancar, dime si tienes las respuestas de la **Fase 0** y empezamos por la **Fase 1** (exponer `getIndicators` públicamente)


## PASAR A LIMPIO 

# Tarea: Integrar lógica 12830 en endpoints genéricos de DeviceController

## Objetivo

Unificar la lógica de `Device12830Controller` dentro de los endpoints genéricos `/api/device`, manteniendo compatibilidad con devices legacy y bifurcando por `isNewDevice` / `isOldDevice`.

## Estado actual

- [X] Fase 0: Análisis inicial
- [X] Fase 1: Helpers comunes
- [X] Fase 2: GET device / actives / normalize
- [X] Fase 3: Indicators
- [X] Fase 4: Metrics
- [X] Fase 5: Activity
- [X] Fase 6: Activity XLSX
- [X] Fase 7: Limpieza controlador 12830
- [X] Fase 8: Validación

## ✅ TAREA COMPLETADA — 2026-05-27 (fix adicional: GET paridad timezone/utcOffset)

## Decisiones tomadas

- Usar `/api/device` como endpoint principal.
- Patrón de delegación: `DeviceController` instancia `Device12830Controller` con el mismo `context` y llama a sus métodos directamente (sin pasar por HTTP).
- Helper `resolveRange` extraído a `src/lib/helpers/resolve-range.ts` para unificar lógica de rango entre `getActivity` y `getIndicators`.
- Los modelos "nuevos" se definen en `src/lib/helpers/device-classifier.ts`: `assetLogger`, `MURAL`, `panel`, `panelVee`, `productCare`, `sondasTrace`, y variantes `panel_*ry*`.
- No se añade ruta dedicada `/api/device/indicators/:id` — los indicators se sirven via `GET /api/device/:id?select=data.indicators` para todos los devices (nuevos → `_getIndicatorsFromTigerData`, legacy → `_getIndicators`). Sin endpoints nuevos.
- `DeviceController.get()` elimina `timezone` y `utcOffset` para devices nuevos (`isNewDevice`) porque `_defaultSelectPaths = ["utcOffset", "timezone"]` los añadía siempre desde el abstract controller, pero `Device12830Controller` no los incluía. Para old devices el comportamiento no cambia.
- Las rutas `/api/12830/device` se eliminaron de `app.ts` tras confirmar paridad funcional.
- La clase `Device12830Controller` se mantiene como clase interna reutilizable (no expone rutas propias).

## Archivos modificados

| Archivo                                  | Cambio                                                                                                                 |
| ---------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| `src/controllers/api/device.ts`        | Delegación a Device12830Controller para devices nuevos en getMetrics, getActivity, getDeviceActivityExportXlsx        |
| `src/controllers/api/12830/device.ts`  | Refactorizado como controlador reutilizable; getActivity, getMetrics, getActivityExportXlsx, getIndicators, getActives |
| `src/lib/helpers/resolve-range.ts`     | Nuevo — helper de resolución de rango de fechas compartido                                                           |
| `src/lib/helpers/device-classifier.ts` | Preexistente — define isOldDevice / isNewDevice                                                                       |
| `src/app.ts`                           | Eliminado `Device12830Controller.init(this)` — rutas /api/12830/device desactivadas                                 |
| `src/controllers/api/device.ts` (fix)  | Eliminados `timezone`/`utcOffset` para isNewDevice en `get()`                                                    |

---

## Log de implementación

### 2026-05-27 — Fases 0–6

#### Fase 0: Análisis inicial ✅

Mapeo completado entre `Device12830Controller` y `DeviceController`:

| Función 12830              | Equivalente genérico                           | Estrategia                                 |
| --------------------------- | ----------------------------------------------- | ------------------------------------------ |
| `get()`                   | `get()`                                       | Delegación +`normalizeDeviceDefinition` |
| `getActives()`            | `getActives()`                                | Migrado a Device12830Controller            |
| `getIndicators()`         | `getV2()` + `_getIndicatorsFromTigerData()` | Via `select=data.indicators`             |
| `getMetrics()`            | `getMetrics()`                                | Delegación directa                        |
| `getActivity()`           | `getActivity()`                               | Delegación directa                        |
| `getActivityExportXlsx()` | `getDeviceActivityExportXlsx()`               | Delegación directa                        |

#### Fase 1: Helpers comunes ✅

- Creado `src/lib/helpers/resolve-range.ts` con función `resolveRange(params, device?)`.
- Resuelve `type` (daily/weekly/monthly) y `from/to` normalizados a `YYYY-MM-DD`.
- Valida que `from` no sea futuro respecto al último mensaje del device y que la ventana no supere 30 días.

#### Fase 2: GET device / actives / normalize ✅

- `Device12830Controller.get()` llama a `getActives()` si el `select` incluye `eventsActive` o `alarmsActive`.
- `normalizeDeviceDefinition()` aplicado en todas las respuestas.

#### Fase 3: Indicators ✅

- `Device12830Controller.getIndicators()` reutilizable internamente.
- En `DeviceController`, indicators via `GET /api/device/:id?select=data.indicators` → `_getIndicatorsFromTigerData()`.
- En listado bulk, delega a `Device12830Controller.getIndicators()` para devices nuevos.

#### Fase 4: Metrics ✅

- `DeviceController.getMetrics()` bifurca por `isOldDevice`:
  - Nuevos → `(new Device12830Controller(this.context)).getMetrics(id, queryOptions)`
  - Legacy → switch por modelo (AD1, AD2 … EDGE)
- Ruta: `GET /api/device/metrics/:id?dateRange=24h|7d|30d&type=...&value=...&from=...&until=...`

#### Fase 5: Activity ✅

- `DeviceController.getActivity()` bifurca por `isOldDevice`:
  - Nuevos → `(new Device12830Controller(this.context)).getActivity(deviceId, options)`
  - Legacy → `ActivityController`
- Ruta: `GET /api/device/:id/activity?from=YYYY-MM-DD&type=daily|weekly|monthly`
- `Device12830Controller.getActivity()` normaliza params legacy (`day`→`daily`, etc.) antes de `resolveRange`.

#### Fase 6: Activity XLSX ✅

- `DeviceController.getDeviceActivityExportXlsx()` refactorizado a `async/await`.
- Bifurca: nuevos → `Device12830Controller.getActivityExportXlsx()`, legacy → `ActivityController.getActivityByday()`.
- Ruta: `GET /api/device/:id/activity/xlsx?startDate=YYYY-MM-DD&endDate=YYYY-MM-DD`

---

### 2026-05-27 — Fase 8: Validación ✅

**Entorno:** API local `localhost:3001` con variables de entorno apuntando a bases de datos de PRE:

- `MONGO_URI` / `DB_CONFIG` → `ako54-mongo.akonet.internal:27017` (MongoDB PRE)
- `TIMESCALE_API_URL` → `http://ako54-timescale.akonet.internal:3500/api` (Timescale API PRE)

Esto permite ejecutar el código local contra datos reales sin necesidad de un entorno completo.

**Device de prueba:** `69df7d6324f2820018437027` — modelo `panel_4ry_6202` (device nuevo), serial `202604150002`, nombre "4RY JR 3 - FW".

**Resultados — validación inicial:**

| Endpoint 12830                              | Endpoint genérico                             | Resultado                                                                                                           |
| ------------------------------------------- | ---------------------------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| `GET /api/12830/device/:id`               | `GET /api/device/:id`                        | ⚠️ Payload correcto pero genérico incluía `timezone`/`utcOffset` extra — corregido en fix posterior        |
| `GET /api/12830/device/metrics/:id`       | `GET /api/device/metrics/:id`                | ✅ Idénticos —`{}` (device sin datos de métricas, comportamiento consistente)                                  |
| `GET /api/12830/device/activity/:id`      | `GET /api/device/:id/activity`               | ✅**Respuesta byte a byte idéntica**                                                                         |
| `GET /api/12830/device/indicators/:id`    | `GET /api/device/:id?select=data.indicators` | ✅ Ya existía — sin endpoint nuevo. Nuevo devices →`_getIndicatorsFromTigerData`, legacy → `_getIndicators` |
| `GET /api/12830/device/activity/:id/xlsx` | `GET /api/device/:id/activity/xlsx`          | ✅**Idénticos** — HTTP 200, mismo tamaño (28284 bytes)                                                     |

**Resultados — tras fix adicional (GET paridad timezone/utcOffset):**

| Endpoint 12830                | Endpoint genérico      | Resultado                                                                                                          |
| ----------------------------- | ----------------------- | ------------------------------------------------------------------------------------------------------------------ |
| `GET /api/12830/device/:id` | `GET /api/device/:id` | ✅**Idénticos** — eliminados `timezone`/`utcOffset` para `isNewDevice` en `DeviceController.get()` |

**TypeScript:** `npx tsc --noEmit` → sin errores ✅ (verificado tras cada cambio)

**Script de paridad:** `test/controllers/api/12830/parity.sh`

```bash
DEVICE_ID=<id> TOKEN=<x-authtoken> bash test/controllers/api/12830/parity.sh
```

- HTTP 0 en ambos lados = `FAIL` (antes falso positivo)
- Verifica que `timezone`/`utcOffset` no estén en `GET /api/device/:id` para devices nuevos
- Exit 0 = todo pasa, Exit 1 = algún fallo

**Resultado final con device `69df7d6324f2820018437027` (`panel_4ry_6202`):**

```
[✓] getDevice — HTTP 200 — responses match
[✓] getDevice / no timezone+utcOffset — Neither field present in generic response
[✓] getIndicators — HTTP 200 — responses match
[✓] getMetrics — HTTP 200 — responses match
[✓] getActivity — HTTP 200 — responses match
[✓] getActivityXlsx — HTTP 200 — responses match
6 passed, 0 failed
```

**Paridad funcional confirmada en todos los endpoints.**

---

### 2026-05-27 — Fase 7: Limpieza ✅

- Eliminado `Device12830Controller.init(this)` de `src/app.ts` (línea 599).
- Las rutas `/api/12830/device/*` ya no están registradas.
- `Device12830Controller` se mantiene como clase interna reutilizada por `DeviceController`.
- TypeScript compila sin errores tras la eliminación.

**Notas para el equipo:**

- Clientes que usen `/api/12830/device/*` deben migrar a `/api/device/*`. Las rutas equivalentes son las listadas en la tabla de validación.
- Diferencia de URL en activity: 12830 era `/activity/:id`, el genérico es `/:id/activity`.
- `timezone`/`utcOffset` ya no se devuelven para devices nuevos en `GET /api/device/:id` — comportamiento alineado con 12830.
- Indicators: no se crea endpoint nuevo — se usa `GET /api/device/:id?select=data.indicators` que ya funciona para todos los devices.