
# Tarea: Investigar por qué devices en `inactive` no vuelven a `active` al transmitir

## 1. Metadata

| Campo            | Valor                                              |
|------------------|----------------------------------------------------|
| Task name        | errcomm — recovery desde estado inactive           |
| Owner            | daniel.roman@ako.com                               |
| Created          | 2026-05-28                                         |
| Last updated     | 2026-06-01                                         |
| Status           | ✅ Cerrada — ambos bugs validados como resueltos   |
| Current phase    | Phase 3 completada — Phase 4 no necesaria          |
| Repository area  | `src/errocomtest/`                                 |
| Main branch/base | main                                               |

---

## 2. Problema y resolución

Los devices en estado `inactive` presentaban dos bugs en el micro errcomm, ambos causados por el mismo fix aplicado el 29/05:

**Bug A** ✅ RESUELTO — No pasaban a `online` al transmitir.
**Bug B** ✅ RESUELTO — No pasaban a `error` al expirar la TXE key.

### Causa raíz

El nombre de la variable TXE en la definición del micro errcomm no coincidía con el nombre real de la clave en Redis. El pipeline buscaba la clave con el nombre incorrecto, por lo que nunca la encontraba y no ejecutaba ninguna transición de estado.

**Fix aplicado:** corregir el nombre de la variable TXE en el micro errcomm para que coincida con la clave real en Redis (`device:errcom:<id>`).

### Flujo tras el fix

```
device transmite → backend renueva clave TXE en Redis → pipeline errcomm detecta heartbeat → device vuelve a online  ✓
TXE key expira  → pipeline errcomm detecta ausencia   → device pasa a error                                          ✓
```

### Evidencia de los bugs (pre-fix, 2026-05-28)

| Device | Observación |
|---|---|
| Cámara 1 (`2612099999111`) | Transmitió a las 08:01 (TXE key renovada). Status siguió `inactive`. Evidencia directa del Bug A. |
| 4RY_TEST_2, 4RY_TEST_3, 4RY SW DEMO, 202605050003 | TXE expirada, status sigue `inactive` en vez de `error`. Evidencia del Bug B. |

### Validación del fix (post-fix, 2026-05-29)

| Watch | Device | Veredicto | Detalle |
|---|---|---|---|
| 09:54 UTC | Cámara 1 | **RECOVERED** ✅ | Transmitió → volvió a `online` en 280s |
| 12:45 UTC | 4RY JR 3 - FW | **TXE_OK** ✅ | TXE expiró a las 12:32 UTC → pasó a `error` correctamente |

---

## 3. Source of truth

| Tipo           | Path                                                                          |
|----------------|-------------------------------------------------------------------------------|
| Runner watch   | `src/errocomtest/cases/error-com/test-errcom-inactive-recovery.ts`           |
| Test activo    | `src/errocomtest/cases/error-com/test-error-comm-real.ts`                    |
| Base class     | `src/errocomtest/base/BaseComTestCase.ts`                                     |
| Env prod       | `.env.prod`                                                                   |

---

## 4. Runner: `test-errcom-inactive-recovery.ts`

### Uso

```bash
npm run test:errcom:inactive-recovery

# Ajustar ventana y polling:
WATCH_MINUTES=10 POLL_INTERVAL_SECONDS=30 npm run test:errcom:inactive-recovery
```

### Variables de entorno (`.env.prod`)

| Variable | Default | Descripción |
|---|---|---|
| `API_URL` | — | URL base de la API de producción |
| `API_KEY_TOKEN` | — | Token de autenticación |
| `REDIS_HOST` | `127.0.0.1` | Host Redis |
| `REDIS_PORT` | `6379` | Puerto Redis |
| `WATCH_MINUTES` | `20` | Duración del watch |
| `POLL_INTERVAL_SECONDS` | `30` | Intervalo de polling |
| `DEVICE_LIMIT` | `200` | Máximo de devices a consultar |

### Flujo del runner

```
FASE 1 — Snapshot A (estado actual)
  ├── Conecta Redis + API
  ├── Fetcha todos los devices, filtra nombre no-SIM (case-insensitive)
  ├── Para cada device inactive:
  │     status, lastMsgTimestamp, TXE TTL, L2 TTL, alarmas L2/TXE
  │     tiempo restante hasta error (TXE TTL)
  └── Imprime tabla resumen en consola

FASE 2 — Watch (polling)
  ├── Cada POLL_INTERVAL_SECONDS relee lastMsgTimestamp + status de cada device
  ├── Cuando lastMsgTimestamp cambia → device transmitió
  │     → captura Snapshot B completo
  │     → clasifica:
  │           RECOVERED  — status = online ✓
  │           BUG        — status sigue inactive o error ✗
  │     → imprime resultado inmediato en consola
  └── Para cuando todos los devices transmitieron o se agota WATCH_MINUTES

Genera HTML en src/errocomtest/reports/ERROR_COMM_INACTIVE_WATCH_<ts>/watch-report.html
```

### Clasificaciones de resultado por device

| Veredicto | Significado |
|---|---|
| `RECOVERED` | Transmitió y volvió a `online` — recovery funciona |
| `BUG` | Transmitió pero sigue `inactive` o `error` — recovery rota |
| `TIMEOUT` | No transmitió durante el watch — no concluyente (device apagado o sin red) |

---

## 5. Runners disponibles

| Runner | Script | Propósito |
|---|---|---|
| `test-error-comm-real.ts` | `npm run test:errcom` | Test activo — 1 device concreto, tú fuerzas la transmisión a mano |
| `test-errcom-inactive-recovery.ts` | `npm run test:errcom:inactive-recovery` | Watch pasivo — observa todos los devices inactive y detecta si pasan a `online` o `error` |

---

## 6. Resumen ejecutivo — qué se hizo y qué se encontró

### Qué se construyó

Se construyeron dos herramientas de observación pasiva sobre producción (no modifican nada):

**`test-error-comm-health.ts`** — Foto instantánea del sistema.
Lee el estado actual de todos los devices (API + Redis + alarmas) y valida si es consistente.
Termina en segundos. Sirve para responder: *¿cómo está el sistema ahora mismo?*

**`test-errcom-inactive-recovery.ts`** — Watch en tiempo real.
Se queda observando los devices en `inactive` y detecta dos eventos:
- Un device transmite → ¿volvió a `online`? → veredicto `RECOVERED` o `BUG`
- La TXE key expira → ¿pasó a `error`? → veredicto `TXE_OK` o `TXE_BUG`
Sirve para responder: *¿el backend procesa bien las transiciones de estado?*

### Qué se observó

**Pre-fix (28/05):** Cámara 1 transmitió dos veces y el status no cambió. 5 devices quedaron con TXE expirada sin moverse a `error`. Ambos bugs confirmados.

**Post-fix (29/05):** El micro errcomm fue corregido (nombre de variable TXE). Resultado de los watches del mismo día:

| Hora (UTC) | Duración | Resultado clave |
|---|---|---|
| 2026-05-28 08:24 | Diagnóstico | Evidencia Bug A en Cámara 1 |
| 2026-05-28 14:04 | 20 min | 6 TIMEOUT |
| 2026-05-28 15:03 | 20 min | 6 TIMEOUT |
| 2026-05-29 07:14 | 20 min | 6 TIMEOUT |
| 2026-05-29 09:54 | 20 min | **Cámara 1 → RECOVERED** ✅ Bug A resuelto |
| 2026-05-29 11:39 | 20 min | 3 TIMEOUT (ventana corta) |
| 2026-05-29 12:45 | 60 min | **4RY JR 3 - FW → TXE_OK** ✅ Bug B resuelto |

---

## 6b. Fases

| Fase | Nombre | Estado |
|------|--------|--------|
| 1 | Diseño del runner | ✅ Hecho |
| 2 | Implementación (Snapshot A + Watch) | ✅ Hecho |
| 2b | Optimización del runner | ✅ Hecho (29/05) |
| 3 | Validación en producción | ✅ Hecho (29/05) — Bug A y Bug B confirmados resueltos |
| 4 | Análisis de causa raíz | ✅ No necesaria — causa identificada por el equipo (nombre variable TXE) |

---

## 7. Device anómalo — Cámara 1 (`2612099999111`)

### Estado observado (2026-05-28 ~08:24)

| Campo | Valor | ¿Normal? |
|---|---|---|
| API Status | `inactive` | — |
| L2 key TTL | — (expirada) | ✓ correcto |
| TXE key TTL | 174995s ≈ 48h | ✗ demasiado reciente |
| L2 alarm | — (no activa) | ✗ debería estar activa |
| TXE alarm | YES (activa) | ✗ solo debería estar en `error` |
| Last Msg | 2026-05-28T08:01:00Z | device transmitió esta mañana |

### Diagnóstico

El device pasó por el ciclo completo: `online → inactive → error → transmitió`. Al transmitir, la TXE key se renovó (TTL fresco de ~48h) pero:
- El **status no volvió a `online`** — se quedó en `inactive`
- La **alarma TXE no se limpió** tras la transmisión
- La **alarma L2 no se restauró** para el nuevo ciclo

Es la evidencia más clara del bug doble en el micro errcomm:
1. Recovery de estado no ejecuta correctamente (inactive en vez de online)
2. Limpieza de alarmas no ocurre tras la transmisión de recovery

### Cuándo volver a mirarlo

El watch del 2026-05-28 14:04 puso a Cámara 1 como **TIMEOUT** (no transmitió en la ventana de 20 min).

**Próxima revisión:** 2026-05-30 — la TXE key de Cámara 1 expira en ~48h desde el 2026-05-28 08:01. Si el micro errcomm no está fixado para entonces, el device pasará a `error` y se disparará de nuevo la alarma TXE. Antes de esa fecha conviene:
1. Confirmar si el fix del micro se ha desplegado a producción.
2. Ejecutar `npm run test:errcom:inactive-recovery` con `WATCH_MINUTES=15` para ver si Cámara 1 ahora recupera correctamente al transmitir.

---

## 8. Log de implementación

### 2026-05-28 — Análisis inicial y primer snapshot

- Primer snapshot (08:24) detectó **Cámara 1** como evidencia directa del bug:
  - Transmitió a las 08:01 (TXE key renovada con TTL fresco ~174995s).
  - Status sigue `inactive` — la recovery no ejecutó.
  - TXE alarm activa (debería haberse limpiado en recovery).
- Hipótesis confirmada parcialmente: el mensaje llega al backend (key se renueva) pero el cambio de estado no ocurre.

### 2026-05-28 — Phase 2: Implementación ✅

**Archivo:** `src/errocomtest/cases/error-com/test-errcom-inactive-recovery.ts`

**Script npm:** `test:errcom:inactive-recovery`

**Fase 1 (Snapshot A):**
- Conecta Redis + API.
- Fetcha devices, filtra nombre no-SIM.
- Por cada device `inactive`: 5 llamadas en paralelo (TXE TTL, L2 TTL, status + lastMsgTimestamp, alarma L2, alarma TXE).
- Muestra tiempo restante hasta error (TXE TTL).

**Fase 2 (Watch):**
- Loop de polling cada `POLL_INTERVAL_SECONDS`.
- Detecta cambio de `lastMsgTimestamp` → Snapshot B inmediato.
- Clasifica: RECOVERED / BUG / TIMEOUT.
- Imprime resultado en consola en tiempo real.
- Genera HTML con dos tablas: Snapshot A y resultados del watch.

**Validación:** `npx tsc --noEmit` — sin errores nuevos.

---

### 2026-05-29 — Optimización del runner

**Archivo:** `src/errocomtest/cases/error-com/test-errcom-inactive-recovery.ts`

**Cambios:**
1. **Redis error listener** — añadido `redis.on('error', ...)`. Antes el ETIMEDOUT aparecía mezclado en el output y las lecturas fallaban en silencio.
2. **Poll loop paralelo** — los devices se leen todos a la vez por poll con `Promise.allSettled`. Antes eran secuenciales, lo que hacía que el watch consumiera tiempo real y llegara tarde a los 40 polls.
3. **Nuevos veredictos TXE_BUG / TXE_OK** — el runner ahora detecta también si la TXE key expira durante el watch y el device no pasa a `error`. Antes solo detectaba transmisiones.
4. **allOk corregido** — el HTML ya no muestra "OVERALL PASS" cuando todos son TIMEOUT. Solo es PASS si hay al menos un RECOVERED o TXE_OK y cero bugs.

**Dos escenarios a validar en Phase 3:**

| Escenario | Qué probar | Cuándo lanzar |
|---|---|---|
| A — recovery tras transmisión | ¿El device vuelve a `online` al transmitir? | ~17:30 UTC (los 5 devices transmitieron ayer ~18:00) |
| B — expiración TXE | ¿El device pasa a `error` al expirar TXE? | ~12:20 UTC (4RY JR 3 - FW expira ~12:52 UTC) |

**Validación:** `npx tsc --noEmit` — sin errores nuevos en el archivo modificado.

---

### 2026-05-29 — Phase 3: Validación en producción ✅

**Watch 09:54 UTC (20 min):**
- Cámara 1 transmitió → `RECOVERED` en 280s → **Bug A resuelto** ✅
- 4RY_TEST_2, 4RY_TEST_3, 4RY SW DEMO, 202605050003 → ya en `online` al inicio del snapshot (recuperaron antes del watch)

**Watch 12:45 UTC (60 min):**
- 4RY JR 3 - FW: TXE expiró a las 12:32 UTC → pasó a `error` → `TXE_OK` → **Bug B resuelto** ✅
- 25191442520010 - AD20: TIMEOUT (no transmitió en 60 min — no concluyente)

**Causa raíz identificada:** nombre de variable TXE incorrecto en el micro errcomm. El pipeline buscaba la clave Redis con el nombre equivocado y nunca la encontraba, por lo que no ejecutaba ninguna transición de estado.

**Fix:** corregido el nombre de la variable TXE en el micro errcomm. Desplegado a producción el 29/05 entre las 07:14 y las 09:54 UTC.
