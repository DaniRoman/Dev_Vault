## Flujo del proyecto

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

```js
{
  "deviceInformation": { ... },
  "parsedData": {
    "events": [
      {
        "date": "2026-02-17T07:51:58.000Z",
        "active": true,
        "eventRef": "event_log_reset",
        "eventValue": 100,
        "counterId": "1",
        "isAlarm": false
      }
    ]
  },
  "deviceType": "12830"
}
```

### Flujo Conexión Layer
[[AKO 44 - Conexion Layer Workflow.canvas]]

Manual de comunicación entre dispositivo y cloud [protocolo ](https://ako0.sharepoint.com/:w:/t/DesarrolloModificaciondeProducto/DEVICE_COMMUNICATION_PROTOCOL/IQANIw2Uzjg2QpMzVO0zj5Z5AWI-WY01IyDjBINtVQTB-ck?e=bLD5hS&ovuser=5a94156b-5d3f-467b-b767-561717bb62ca%2Cdaniel.roman%40ako.com&clickparams=eyJBcHBOYW1lIjoiVGVhbXMtRGVza3RvcCIsIkFwcFZlcnNpb24iOiI1MC8yNjAxMDQwMDkyNSIsIkhhc0ZlZGVyYXRlZFVzZXIiOmZhbHNlfQ%3D%3D).

- Define **qué tipos de mensajes existen** (status, event/alarm, sample, link, etc.).
- Para cada tipo, define **qué campos lleva** y **cómo se organizan** (muchas veces en arrays para ahorrar bytes)
- el mensaje **no viaja como JSON en texto**, sino como **CBOR** (un binario equivalente a ese “JSON”).
- al final se añade un **CRC32** para comprobar integridad (que el payload no está corrupto y corresponde al dispositivo/uuid).

***Ejemplo de uso***
Lo usaría cuando necesite saber qué bytes deberían significar qué cosa
Para `event`, el payload debe tener `ty: "event"` y una `d` con filas del estilo `[ts, event_id, val, st, cnt_id]` (según la spec).
Compruebo que el mensaje respeta esa forma.
`{ ty:"event", d:[[ts, 19, 100, 1, 1]] }`

---

El micro translator es la capa que convierte los mensajes que llegan desde la capa de conexión en datos “entendibles” para el cloud y los distribuye internamente. Recibe mensajes ya autenticados y los valida para asegurar que tienen una estructura correcta. Interpreta el contenido según el tipo de mensaje y lo transforma desde un formato compacto del protocolo a un formato interno normalizado. Con esa salida, publica el mensaje en el canal interno adecuado para que lo consuma el micro especializado (eventos, muestras, estado, etc.). También gestiona la comunicación relacionada con parámetros: confirma recepciones, guarda pendientes y reintenta envíos cuando toca. Además puede manejar mensajes que van en sentido contrario (cloud → dispositivo), adaptándolos al formato esperado y enviándolos por el canal de salida correspondiente. En resumen: traduce, valida, normaliza y enruta mensajes entre dispositivos y micros de negocio.

El mensaje es enviado por rabbit a un microservicio u otro en función del tipo de mensaje,  cada micro tiene su especialidad `sample, events, status`

### Flujo Translator

[[AKO/AKO44/Docs/AKO 44 - Driver Translator Perte WorkFlow.canvas|AKO 44 - Driver Translator Perte WorkFlow]]
## Conceptos del dispositivo

### Sondas

| Característica      | Sonda Digital                    | Sonda Analógica                                           |
| ------------------- | -------------------------------- | --------------------------------------------------------- |
| Señal entregada     | Binaria (0 o 1)                  | Continua (rango de valores)                               |
| Tipo de información | Sí/No, Encendido/Apagado         | Medición exacta (temperatura, presión, etc.)              |
| Ejemplo 1           | Sensor de puerta abierta/cerrada | Termómetro de resistencia (RTD)                           |
| Ejemplo 2           | Interruptor de límite            | Sensor de presión (0–5 V o 4–20 mA)                       |
| Uso típico          | Detectar presencia, posición     | Medir magnitudes físicas como temperatura, nivel, presión |

>[!important] importante
>`akocloud-api > src > schemas > device-definitions > panel_1ry_6402` el último dígito dígito me dice que protocolo utiliza, `2-CoAP` - `1-MQTT`

## Protocolos de red
[[Ako44 - Protocolos]]

## Device communication protocol

> Aquí puedo mapear los `event_id` de los eventos con sus respectivos códigos en el `json`

[Recurso ](https://ako0.sharepoint.com/:w:/r/teams/DesarrolloModificaciondeProducto/DEVICE_COMMUNICATION_PROTOCOL/_layouts/15/Doc.aspx?sourcedoc=%7B940D230D-38CE-4236-9333-54ED338F9679%7D&file=ALWAYS_ON_DEVICE_EN12830_COMMUNICATION_PROTOCOL_rev2.docx&action=default&mobileredirect=true&wdOrigin=TEAMS-MAGLEV.p2p_ns.rwc&wdExp=TEAMS-TREATMENT&wdhostclicktime=1771252569867&web=1)

### Mook device conexión 
[[Ako44 - Mok device conexión Script.canvas]]

## Manufactured device license code

[[ako 44 - Manufactured device license code ]]

## Como interpretar el `schema > device.definitions` en el archivo `newDarwingPanel` del sharePoint
[[ako 44 - mapear device.definitions contra Excel File ]]