# Guía: Alarmas de temperatura MAX/MIN en cloud — panel_0ry

## Contexto

Las alarmas de cloud se definen en el campo `alarms.cloud` de la definición de dispositivo
(p. ej. `src/schemas/device-definitions/panel_0ry_7302.json`).

A diferencia de las alarmas de dispositivo (`alarms.device`), las alarmas de cloud **no las
evalúa el firmware** sino el microservicio dedicado de timeseries. Este servicio lee la
definición expuesta por `akocloud-api`, evalúa la condición con los valores en tiempo real, y
escribe el estado de la alarma en `device.lastStatus.alarms[ref]`.

`akocloud-api` en sí solo:
- Expone la definición con la sección `alarms.cloud` completa.
- Lee `device.lastStatus.alarms[ref]` para cada alarma definida (ver `device.ts:3328-3337`).
- No evalúa `compareType`, `valueRef` ni `threshold` — eso es responsabilidad del microservicio.

---

## Esquema de un objeto `alarms.cloud`

```json
{
  "ref":         "<identificador único>",
  "description": "<descripción legible>",
  "priority":    "<ref a un param conf>",
  "threshold":   "<ref a un param conf>",
  "valueRef":    "<ref a un analogInput o digitalInput>",
  "compareType": "above" | "below"
}
```

### Atributos en detalle

| Atributo | Tipo | Obligatorio | Descripción |
|---|---|---|---|
| `ref` | string | ✅ | Identificador único de la alarma. Debe coincidir con el `code` que registra el microservicio de timeseries. |
| `description` | string | ✅ | Texto legible para UI y logs. |
| `priority` | string → `conf.ref` | ✅ | Referencia a un param `conf` que el usuario configura para asignar el grupo de severidad (sin asignar / crítica / moderada / informativa). |
| `threshold` | string → `conf.ref` | ✅ | Referencia a un param `conf` que contiene el valor numérico del umbral. |
| `valueRef` | string → `analogInputs.ref` o `digitalInputs.ref` | Cuando hay comparación | Registro que proporciona el valor en tiempo real a comparar con el umbral. Si no hay comparación (p. ej. alarma de inactividad) se omite. |
| `compareType` | `"above"` \| `"below"` | Cuando hay `valueRef` | Dirección de la comparación: `"above"` → alarma si `valueRef > threshold`; `"below"` → alarma si `valueRef < threshold`. |

### Dónde encontrar los valores correctos para `threshold`, `priority` y `valueRef`

#### `threshold` y `priority` — el puente es `conf[].code`

El CSV de alarmas te da un **código corto** en las columnas `Umbral` y `Priority` (p. ej. `A1`, `c_AG1`).
Ese código corto aparece como sufijo del campo `code` en `conf[]`. Con eso localizas el `ref`:

| CSV corto | `conf[].code` | `conf[].ref` | Uso |
|---|---|---|---|
| `A1` (Umbral) | `al.A1` | `param_A1_alarm_prb1_max` | → `threshold` |
| `A2` (Umbral) | `al.A2` | `param_A2_alarm_prb1_min` | → `threshold` |
| `c_AG1` (Priority) | `al.c_AG1` | `param_c_ag1_max_s1` | → `priority` |
| `c_AG2` (Priority) | `al.c_AG2` | `param_c_ag2_min_s1` | → `priority` |

**Proceso:**
1. Anota el valor de `Priority` y `Umbral` del CSV de alarmas.
2. Busca en `conf[]` el objeto cuyo `code` contenga ese valor.
3. El `ref` de ese objeto es lo que pones en `priority` o `threshold`.

Características de los params de umbral (`threshold`): `group: "al"`, `origin: "device"`, `type: "input"` — son configurables por el instalador desde la UI.

Características de los params de prioridad (`priority`): `group: "al"`, `origin: "cloud"`, `type: "selector"` — el usuario elige la severidad del grupo de alarma.

#### `valueRef` — registro en `analogInputs.device[]`

No viene del CSV de alarmas. Busca en `analogInputs.device[]` el registro del sensor cuya lectura quieres vigilar:

```json
// analogInputs.device[0]
{ "ref": "reg_amv_analog_prb1", "code": 0, "name": "Sonda S1", "unit": "ºC", "convert": 100 }
```

El microservicio divide el valor raw Modbus por `convert` (÷100) para obtener ºC y lo compara contra el umbral.

#### `values[]` vs `analogInputs[]` — no confundir

El mismo `ref` (`reg_amv_analog_prb1`) aparece en tres sitios con roles distintos:

| Sección | Rol |
|---|---|
| `analogInputs.device[]` | Define el registro a bajo nivel: código Modbus y factor de conversión. **Fuente del dato real.** Lo usa el microservicio para la comparación. |
| `values[]` | Declara qué registros se exponen en la API con nombre y unidad (`lastStatus.values`, `device.values`). **Metadata para la UI.** No lo usa la alarma. |
| `valueRef` en `alarms.cloud` | Apunta al `ref` de `analogInputs` — indica al microservicio qué valor comparar. |

Para saber qué valor vigilar → mira `analogInputs.device[]`. `values[]` solo confirma que ese registro está publicado como valor de tiempo real.

### Mapeo columnas CSV → atributos JSON

| Columna CSV (alarmas) | Atributo JSON |
|---|---|
| `parametro` | `ref` |
| `Variable` | `description` |
| `Priority` | `priority` |
| `Umbral` | `threshold` |
| `Value Ref` | `valueRef` |
| `Compare Type` | `compareType` |

---

## Cómo añadir una alarma MAX o MIN de temperatura (panel_0ry)

### Prerrequisitos

Verifica que los params referenciados existen en la sección `conf` de la definición:

| Param | Rol |
|---|---|
| `param_c_ag1_max_s1` | Grupo de prioridad de la alarma MAX S1 (configurable por el usuario) |
| `param_A1_alarm_prb1_max` | Umbral numérico de la alarma MAX S1 (ºC) |
| `param_c_ag2_min_s1` | Grupo de prioridad de la alarma MIN S1 |
| `param_A2_alarm_prb1_min` | Umbral numérico de la alarma MIN S1 (ºC) |
| `reg_amv_analog_prb1` | Valor en tiempo real de la Sonda S1 (ºC) — en `analogInputs` |

### Paso 1 — Añadir las entradas en `alarms.cloud`

Añade los dos objetos al final del array `alarms.cloud` en la definición del dispositivo:

```json
{
  "ref": "alarm_cloud_error_1",
  "description": "Alarma MAX Sonda S1",
  "priority": "param_c_ag1_max_s1",
  "threshold": "param_A1_alarm_prb1_max",
  "valueRef": "reg_amv_analog_prb1",
  "compareType": "above"
},
{
  "ref": "alarm_cloud_error_2",
  "description": "Alarma MIN Sonda S1",
  "priority": "param_c_ag2_min_s1",
  "threshold": "param_A2_alarm_prb1_min",
  "valueRef": "reg_amv_analog_prb1",
  "compareType": "below"
}
```

Lógica de evaluación (responsabilidad del microservicio):
- MAX: si `reg_amv_analog_prb1 > param_A1_alarm_prb1_max` → alarma activa.
- MIN: si `reg_amv_analog_prb1 < param_A2_alarm_prb1_min` → alarma activa.

### Paso 2 — Validar el JSON

```bash
python3 -m json.tool src/schemas/device-definitions/panel_0ry_7302.json > /dev/null && echo "OK"
```

### Paso 3 — Registro en el microservicio de timeseries

> **Nota:** Esta sección requiere verificación con el equipo de backend del microservicio.
> El microservicio de timeseries necesita conocer el `ref` de las nuevas alarmas para empezar
> a evaluarlas y registrar el historial. El procedimiento exacto depende de cómo el micro
> recibe actualizaciones de la definición (polling, evento AMQP, recarga manual, etc.).

Acciones típicas que hay que confirmar:
- [ ] El micro relee la definición automáticamente al desplegarse, o hay que forzar una recarga.
- [ ] Los `ref` nuevos (`alarm_cloud_error_1`, `alarm_cloud_error_2`) deben estar presentes
  en la lista de alarmas registradas del micro para ese modelo de dispositivo.
- [ ] Si el micro tiene una colección/tabla de alarmas conocidas, añadir las dos entradas con
  sus metadatos (`ref`, `compareType`, umbrales por defecto si aplica).

---

## Estado actual en panel_0ry_7302

Las dos alarmas se añadieron y el JSON está validado (2026-06-15):

```
alarms.cloud[3]  alarm_cloud_error_1  MAX S1  above  param_A1_alarm_prb1_max
alarms.cloud[4]  alarm_cloud_error_2  MIN S1  below  param_A2_alarm_prb1_min
```

---

## Referencia: patrón en otras definiciones

Para confirmar el patrón, el mismo esquema MAX+MIN existe en otras definiciones:

| Definición | Alarm MAX (above) | Alarm MIN (below) |
|---|---|---|
| `panel_1ry_6401` | `alarm_cloud_time_cool` | `alarm_cloud_time_setpoint` |
| `panel_2ry_7102` | `alarm_cloud_time_cool` | `alarm_cloud_time_setpoint` |
| `panel_4ry_6201` | `alarm_cloud_time_cool` | `alarm_cloud_time_setpoint` |

---

## Errores comunes a evitar

| Error | Consecuencia |
|---|---|
| Comentarios inline en el JSON (`// texto` o texto tras un valor) | JSON inválido — la definición no carga |
| Falta de coma entre propiedades | JSON inválido |
| `priority` o `threshold` apuntando a un `ref` inexistente en `conf` | La alarma no resolverá el umbral ni la severidad |
| `valueRef` apuntando a un registro que no existe en `analogInputs`/`digitalInputs` | El microservicio no podrá leer el valor a comparar |
| `compareType` incorrecto (p. ej. `"above"` en una alarma de mínima) | La alarma nunca se activa o se activa al revés |
