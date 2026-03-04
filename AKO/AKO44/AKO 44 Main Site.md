## Flujo del proyecto general
[[AKO 44 Flujo Del Proyecto]]
## Conceptos

En un equipo IoT (un controlador, registrador, etc.) normalmente tienes **entradas** (sensores) y **salidas** (actuadores).

- **Entradas** → sondas/sensores/interruptores (lo que el equipo lee)
- **Salidas** → relés/actuadores (lo que el equipo enciende/apaga)

### Relé

Un relé es una **salida** que el equipo puede **activar/desactivar** para controlar algo:

- encender compresor, ventilador, resistencia de desescarche, luz…
- En datos, un relé suele ser **digital**: 0/1 (OFF/ON)

> “Un aparato tiene más relés” = puede controlar más cosas (más salidas).

### Sondas

es un sensor**, y normalmente entra al equipo como una **entrada**.

- **Sonda = sensor** (mide temperatura/humedad/…)    
- En electrónica/IoT se considera una **entrada** porque el equipo “recibe” ese valor.
- Suele ser **entrada analógica** (un número), aunque hay sondas/interruptores que son digitales (ej. “puerta abierta/cerrada”).

| Característica      | Sonda Digital                    | Sonda Analógica                                           |
| ------------------- | -------------------------------- | --------------------------------------------------------- |
| Señal entregada     | Binaria (0 o 1)                  | Continua (rango de valores)                               |
| Tipo de información | Sí/No, Encendido/Apagado         | Medición exacta (temperatura, presión, etc.)              |
| Ejemplo 1           | Sensor de puerta abierta/cerrada | Termómetro de resistencia (RTD)                           |
| Ejemplo 2           | Interruptor de límite            | Sensor de presión (0–5 V o 4–20 mA)                       |
| Uso típico          | Detectar presencia, posición     | Medir magnitudes físicas como temperatura, nivel, presión |

>[!important] importante
>`akocloud-api > src > schemas > device-definitions > panel_1ry_6402` el último dígito dígito me dice que protocolo utiliza, `2-CoAP` - `1-MQTT`

### Samples & Eventos

***Sample (muestra / telemetría)*** ¿cuánto vale X en el tiempo? ⇒ **sample**

- **Qué es:** una **medición** (data point) de una o varias variables.
- **Frecuencia:** suele ser **periódico** (cada X segundos/minutos) o “cuando hay lectura”.
- **Forma:** normalmente valores numéricos/continuos con timestamp.
- **Uso típico:** series temporales, gráficas, promedios, tendencias, umbrales.

	- `temperatura=4.2°C`
	- `humedad=55%`
	- `corriente=1.8A`
	- `setpoint_actual=4`

***Evento (event)*** ¿qué pasó / cambió? ⇒ evento

- **Qué es:** una **ocurrencia discreta**: “ha pasado algo”.
- **Frecuencia:** **no periódica**; ocurre cuando cambia algo o se dispara una condición.
- **Forma:** suele incluir un **código/tipo** (y quizá severidad, payload corto).
- **Uso típico:** auditoría, alarmas, notificaciones, workflows, incidencias

- Alarma de alta temperatura ACTIVADA
- Puerta abierta
- Defrost iniciado / finalizado
- Error E12
- Configuración aplicada



## Protocolos de red
[[Ako44 - Protocolos]]
## Flujos de trabajo
[[AKO 44 Flujos de trabajo]]
## Status Legacy
[[AKO 44 - Status Legacy]]
## Manual de comunicación entre dispositivo y cloud

^88cdea

[protocolo ](https://ako0.sharepoint.com/:w:/t/DesarrolloModificaciondeProducto/DEVICE_COMMUNICATION_PROTOCOL/IQANIw2Uzjg2QpMzVO0zj5Z5AWI-WY01IyDjBINtVQTB-ck?e=bLD5hS&ovuser=5a94156b-5d3f-467b-b767-561717bb62ca%2Cdaniel.roman%40ako.com&clickparams=eyJBcHBOYW1lIjoiVGVhbXMtRGVza3RvcCIsIkFwcFZlcnNpb24iOiI1MC8yNjAxMDQwMDkyNSIsIkhhc0ZlZGVyYXRlZFVzZXIiOmZhbHNlfQ%3D%3D).

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

### Documento para métricas

[Recurso](https://ako0-my.sharepoint.com/:x:/r/personal/fmaataoui_ako_com/_layouts/15/Doc.aspx?sourcedoc=%7B14551CBA-3E92-4704-99CC-ACD1E74D41EF%7D&file=Chart%20new%20DARWIN%20Panel.xlsx&wdLOR=cAEAF4F08-87EF-CF44-857B-21C042E18A1A&fromShare=true&action=default&mobileredirect=true)


## Manufactured device license code
[[ako 44 - Manufactured device license code ]]
## Como interpretar  parámetros de la definición del dispositivo 
[[ako 44 - mapear device.definitions contra Excel File ]]
## Feature

[[AKO44 - feature-KNT-2243]]
## Scripts 

>[!success] Script para simular datos de un dispositivo

[[Ako44 - Mok device conexión Script.canvas]]

>[!warning] Crear un payload a mano 

Para probar los eventos de la activación de la alarma y la desactivación de esta

```sh
# Alarma 1 → alarmCount = 1
node bin/examples/client-perte.js 974130021 0x2E19CFDA300B3F385A343536754B33324B572E7E alarm -v -pl "{\"id\":[974130021,6000,1,26,0],\"ty\":\"alarm\",\"d\":[[$(date +%s),1,85,1,1]]}"


# Alarma 2 (distinto alarm_id y cnt_id) → alarmCount = 2
node bin/examples/client-perte.js 974130021 0x2E19CFDA300B3F385A343536754B33324B572E7E alarm -v -pl "{\"id\":[974130021,6000,1,26,0],\"ty\":\"alarm\",\"d\":[[$(date +%s),2,90,1,2]]}"

  

# Alarma 3 → alarmCount = 3
node bin/examples/client-perte.js 974130021 0x2E19CFDA300B3F385A343536754B33324B572E7E alarm -v -pl "{\"id\":[974130021,6000,1,26,0],\"ty\":\"alarm\",\"d\":[[$(date +%s),3,70,1,3]]}"

# Desactivar alarma 2 (st=0) penultimo parametro → alarmCount = 2
node bin/examples/client-perte.js 974130021 0x2E19CFDA300B3F385A343536754B33324B572E7E alarm -v -pl "{\"id\":[974130021,6000,1,26,0],\"ty\":\"alarm\",\"d\":[[$(date +%s),2,0,0,2]]}"
```