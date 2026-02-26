### `status.conf`

Este reacciona a cambios de parámetros (mute, enabled_probes, etc.), y actualiza campos/valores del dispositivo en la base de datos para que el resto del sistema sepa cómo debe comportarse/mostrarse.

Este micro expone varios handlers, uno por modelo:

`handleStatusConfInputAD20`, `AD20V3`, `DIGITAL`, `AD7`, `CAMREGIS`, `DARWIN`, `MULTISONDA`, `AD1`, `AD2`, `AD3`, `CCV2`.

Que hacen estos handlers
- Parsea el json de entrada `JSON.parse(msg)`

```json
"{\"deviceInformation\":{...},\"payload\":{...}}"
```

```json
{
  "deviceInformation": {
    "serialNumber": 974130021,
    "device": {
      "_id": "6989bc8e59e52a4bafd618a9",
      "model": "panel_1ry_6402"
    }
  },
  "payload": {
    "ty": "sample",
    "d": [[1772078034, 83]],
    "d1": [[1772078034, 1, 0]]
  },
  "info": {
    "address": "10.10.2.201"
  }
}
```
- Recorre el `message.conf` (La lista de parámetros de confg recibidos)
- Segun el `conf.ref` funciones internas realizan cambios.

Ejemplo: A este micro le llega algo tipo 

- `context.device._id`
- `conf: [{ ref: "...", value: ... }, ...]`

Para luego aplicar una lógica de negocio como habilitar/deshabilitar sondas, activar mute...

### `status.network`

Gestiona eventos/entradas relacionadas con conectividad (dispositivo NB-IoT) y con SIM (actualización suspension ) y con geolocalización.
Recibe un mensaje:
- Lo parsea
- Valida los cambios mínimos (`context.device conectivity imsi`)
- Actualiza los estados/atributos en BD (online/offline, SIM, cell info)
- Actualización del histórico de geolocalización
### `status.updater`

Recibe mensajes de **status** del dispositivo, actualiza en la BD del cloud el “estado actual” (`lastStatus`, online, timestamps, valores/virtuals), descarta mensajes viejos y publica eventos internos tipo `status.updated.<model>` y notificaciones en tiempo real. También recalcula contadores (alarmas) y algunas reglas extra (p. ej. “healthy_status”).
