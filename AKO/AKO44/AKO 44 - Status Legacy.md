`status.conf`

Este reacciona a mensaje de configuración actualizada, y aplica efectos en el estado/valores del dispositivo.

Este micro expone varios handlers, uno por modelo:

`handleStatusConfInputAD20`, `AD20V3`, `DIGITAL`, `AD7`, `CAMREGIS`, `DARWIN`, `MULTISONDA`, `AD1`, `AD2`, `AD3`, `CCV2`.

Que hacen estos handlers
- Parsea el json de entrada `JSON.parse(msg)`

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