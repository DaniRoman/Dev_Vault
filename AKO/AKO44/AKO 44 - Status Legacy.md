`status.conf`

Este reacciona a mensaje de configuración actualizada, y aplica efectos en el estado/valores del dispositivo.

Este micro expone varios handlers, uno por modelo:

`handleStatusConfInputAD20`, `AD20V3`, `DIGITAL`, `AD7`, `CAMREGIS`, `DARWIN`, `MULTISONDA`, `AD1`, `AD2`, `AD3`, `CCV2`.

Que hacen estos handlers
- Parsea el json de entrada `JSON.parse(msg)`
- Recorre el `message.conf` (La lista de parámetros de confg recibidos)
- Segun el `conf.ref` funciones internas realizan cambios.

Ejemplo: A este micro le llega algo tipo 

- `context.device._id`
- `conf: [{ ref: "...", value: ... }, ...]`

Para luego aplicar una lógica de negocio como habilitar/deshabilitar sondas, acti