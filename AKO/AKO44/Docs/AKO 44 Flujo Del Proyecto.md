
Los dispositivos IoT se conectan al Cloud mediante la capa de conexión que es la puerta de entrada y salida entre los dispositivos y el cloud. Mantiene el servidor de comunicación y recibe las peticiones del dispositivo con el mensaje en bruto (bytes). Extrae la identidad del dispositivo y el tipo de operación a partir de la petición (por ejemplo si el dispositivo está enviando datos o solicitando información). A partir de la ruta/endpoint detecta acciones especiales como vinculación o actualizaciones. Delega la validación del dispositivo y la integridad del mensaje (comprobación de CRC y decodificación del contenido) para asegurarse de que no está corrupto y pertenece a un dispositivo permitido. Si todo es correcto, entrega el mensaje al bus interno para que siga el flujo hacia la traducción y los micros de negocio. Si el mensaje no es válido, responde al dispositivo con un código de error. En el sentido inverso (cloud → dispositivo), prepara el mensaje de salida, lo empaqueta con su integridad y lo envía al dispositivo por el mismo canal de comunicación.

Lo que llega al micro CoAP (capa de conexión)

```txt
request.payload = [ CBOR_BYTES ... ][ CRC32_4_BYTES ]
                  ^^^^^^^^^^^^^^^^  ^^^^^^^^^^^^^^^
                  payload real      firma integridad
```

- `CBOR_BYTES`: longitud variable (casi todo el mensaje)
- `CRC32_4_BYTES`: siempre los **últimos 4 bytes** (32 bits)

Cómo lo deja la capa de conexión por dentro (tras validar)

```js
// esto es el "payload" decodificado (ya NO son bytes)
{
  id: [974130021, 6000, 1, 26, 0],
  ty: "event",
  d: [
    [1771245275, 19, 100, 1, 1],
    [1771245285, 25, 50, 1, 2],
    [1771245295, 30, 0, 0, 3]
  ]
}
```

Qué manda la capa de conexión al translator (por Rabbit)

```json
{
  "deviceInformation": {
    "serialNumber": 974130021,
    "uuid": "2E19CFDA300B3F...",
    "license": true,
    "device": { "_id": "6989bc8e...", "model": "panel_1ry_6402", ... }
  },
  "payload": {
    "id": [974130021,6000,1,26,0],
    "ty": "event",
    "d": [[1771245275,19,100,1,1], ...]
  }
}
```

Qué hace el translator con eso Ya le llega JSON.
- valida estructura
- interpreta `ty:"event"`
- traduce el contenido compacto (`d: [...]`) a formato cloud normalizado
- publica a un topic interno según el tipo (`perte.input.event`, etc.)
- **Clave:** aquí ya no hay `ty/id/d`.  
Ahora hay `parsedData.samples[]` con `refs` legibles.

```js
//Forma 4
{
  "deviceInformation": { ... },
  "parsedData": {
    "samples": [
      {
        "ts": 1771411753,
        "analogInputs": [
          { "ref": "reg_amv_analog_prb1", "value": 225 },
          { "ref": "reg_amv_analog_prb2", "value": 650 },
          { "ref": "reg_amv_analog_actual_sp", "value": -50 }
        ],
        "digitalInputs": [
          { "ref": "reg_amv_digital_var_device_on", "value": 1, "counter": 150 },
          { "ref": "reg_amv_digital_var_door_open", "value": 0, "counter": 75 }
        ]
      },
    ]
  },
  "deviceType": "12830"
}

```

Después Entra al micro Sample (Injector) El injector recibe exactamente la **Forma 4**.
Pero internamente lo procesa “por muestra”:
- coge `parsedData.samples[]`
- para cada item crea un “mensaje de una sola muestra” (interno)

```js
{
  "deviceInformation": { ... },
  "parsedData": {
    "ts": 1771411773,
    "analogInputs": [...],
    "digitalInputs": [...]
  },
  "deviceType": "12830"
}
```

> Lo que publica el Sample injector (timestream-like)

```json
{
  "timestamp": "Wed Jan 21 1970 13:03:31 GMT+0100 ...",
  "context": { "hidden": "hidden" },
  "sample": [
    { "timestamp": "...", "value": 228, "type": "values", "code": "reg_amv_analog_prb1" },
    { "timestamp": "...", "value": 652, "type": "values", "code": "reg_amv_analog_prb2" },
    { "timestamp": "...", "value": -49, "type": "values", "code": "reg_amv_analog_actual_sp" }
  ]
}
```
---
