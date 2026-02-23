## Flujo del proyecto general
[[AKO 44 Flujo Del Proyecto]]

## Flujo Set Context
[[AKO 44 - Flujo Registrar el Context.canvas]]

## Flujo de cambio de parámetros Cloud - Device
[[AKO 44 - Flujo de cambio de parámetros Cloud - Device.canvas]]

- End point para devolver métricas que timescale esta guardando mas simular mas mensajes para poder recuperar y luego acabar mandando muchos mensajes 

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
### Flujo Translator
[[AKO/AKO44/Docs/AKO 44 - Driver Translator Perte WorkFlow.canvas|AKO 44 - Driver Translator Perte WorkFlow]]

El micro translator es la capa que convierte los mensajes que llegan desde la capa de conexión en datos “entendibles” para el cloud y los distribuye internamente. Recibe mensajes ya autenticados y los valida para asegurar que tienen una estructura correcta. Interpreta el contenido según el tipo de mensaje y lo transforma desde un formato compacto del protocolo a un formato interno normalizado. Con esa salida, publica el mensaje en el canal interno adecuado para que lo consuma el micro especializado (eventos, muestras, estado, etc.). También gestiona la comunicación relacionada con parámetros: confirma recepciones, guarda pendientes y reintenta envíos cuando toca. Además puede manejar mensajes que van en sentido contrario (cloud → dispositivo), adaptándolos al formato esperado y enviándolos por el canal de salida correspondiente. En resumen: traduce, valida, normaliza y enruta mensajes entre dispositivos y micros de negocio.

El mensaje es enviado por rabbit a un microservicio u otro en función del tipo de mensaje,  cada micro tiene su especialidad `sample, events, status`

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