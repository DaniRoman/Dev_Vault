>[!important] Importante ***`microservice.json`***

## Flujo de trabajo con IA .md
[[AKO 44  - Flujo de trabajo con IA]]

## Devices creados

```json
{
	newDevices:
	[
		{
		"name": "panel_0ry_7302",
		"deviceDefinition": "{{69fdc84da35d2373f93df2fa}}",
		"commercialVersion": "AKO-D14012N",
		"commercialName": "AKO-D14012N",
		"connectivity": "nbiot",
		"licenseCode": "5cd520177b4d0d002f472c16",
		"serialNumber": "{{974130023}}",
		"imei": "{{9741300023}}",
		"validationCode": "{{9741300023}}",
		"imsi": "{{9741300023}}",
		"uuid": "2E19CFDA300B3F385A343536754B33324B572E7E"
		},
		
		{
		"name": "panel_4ry_6202",
		"deviceDefinition": "6981e314cc2593f5137d03e8",
		"commercialVersion": "AKO-D14423N",
		"commercialName": "AKO-D14423N",
		"connectivity": "nbiot",
		"licenseCode": "5cd520177b4d0d002f472c16",
		"serialNumber": "974130024",
		"imei": "9741300024",
		"imsi": "9741300024",
		"uuid": "2E19CFDA300B3F385A343536754B33324B572E7E",
		"_id": "6a02fa149669739498a9d79b",
		},
		{
		"name": "panel_4ry_6202",
		"deviceDefinition": "6981e314cc2593f5137d03e8",
		"commercialVersion": "AKO-D14423N",
		"commercialName": "AKO-D14423N",
		"connectivity": "nbiot",
		"licenseCode": "5cd520177b4d0d002f472c16",
		"serialNumber": "974130024",
		"imei": "9741300024",
		"imsi": "9741300024",
		"uuid": "2E19CFDA300B3F385A343536754B33324B572E7E",
		"_id": "6a02fa149669739498a9d79b",
		}
	],
	oldDevices:
	[
		{
		
		},
		
	]
}
	
```


## Unificar Endpoints
[[AKO44 - Unificar Endpoints]]
## Logs
[[AKO 44 - Guia_Logs_Microservicios]]

## ResetDevices
[[AKO 44 - Reset Conf for Devices]]
## Activación de  _`Panel0Ry`_
[[AKO 44 - Activación de Panel0Ry]]

## `Live`
[[AKO 44 - Live Messages]]

## ***`alarm-ACK`***
[[alarm-ACK workflow.canvas]]

## ***`Fota`***
[[Fota]]

## ***`PLMN`***
[[AKO 44 - PLMN]]

## ***`Micro ErrorComm`***
[[AKO 44 - ErrorComm]]

## ***`Net-Sync`***
[[AKO 44 - Net-Sync]]

## ***`comm_sync`***
[[AKO 44 - comm_syn]]

## `Tests`
[[AKO 44 TestCases]]

## Translator
[[AKO 44 - Translator]]

## Protocolo de comunicación dispositivo y cloud

[Documento](https://ako0.sharepoint.com/:w:/t/DesarrolloModificaciondeProducto/DEVICE_COMMUNICATION_PROTOCOL/IQANIw2Uzjg2QpMzVO0zj5Z5AWI-WY01IyDjBINtVQTB-ck?e=bLD5hS&ovuser=5a94156b-5d3f-467b-b767-561717bb62ca%2Cdaniel.roman%40ako.com&clickparams=eyJBcHBOYW1lIjoiVGVhbXMtRGVza3RvcCIsIkFwcFZlcnNpb24iOiI1MC8yNjAxMDQwMDkyNSIsIkhhc0ZlZGVyYXRlZFVzZXIiOmZhbHNlfQ%3D%3D)
- Documento de especificación del protocolo entre dispositivo y cloud. ^91b034
- Define transporte y formato: NB-IoT + UDP + CoAP + JSON/CBOR.
- Describe cada tipo de mensaje (`status`, `audit`, `cmd`, `sync`, etc.) y la estructura de su payload.
- Sirve para interpretar y construir correctamente los mensajes entre firmware y backend.

## Diferencia entre CMD y Parámetro
[[AKO 44 Diferencia entre CMD y Param]]
## Configurar Darwin 2Ry

[Excel de Conf de parámetros en dispositivo](https://ako0-my.sharepoint.com/:x:/r/personal/fmaataoui_ako_com/_layouts/15/Doc.aspx?sourcedoc=%7B14551CBA-3E92-4704-99CC-ACD1E74D41EF%7D&file=Chart%20new%20DARWIN%20Panel.xlsx&wdLOR=c99E32DE4-E50B-3441-A947-77B3FBF12E90&fromShare=true&action=default&mobileredirect=true&wdOrigin=TEAMS-MAGLEV.p2p_ns.rwc&wdExp=TEAMS-TREATMENT&wdhostclicktime=1773646586414&web=1)
[Excel de  Chart new Darwin](https://ako0.sharepoint.com/:x:/r/teams/DesarrolloModificaciondeProducto/NEWDARWIN/_layouts/15/Doc.aspx?sourcedoc=%7BEEE089AD-7016-4C7B-9872-92AE4155BE82%7D&file=2024_NEW_DARWIN_PANEL_2RY_REV4.xlsx&action=default&mobileredirect=true)

>[!warning] Como mapear Excel de configuración junto con los parámetros del otro excel
>

![[Pasted image 20260316084231.png]]

Mirando el nombre del parámetro `Ini1` y el valor un `9` en el excel de configuración de parametros lo busco en el excel de `chart new Darwing` en la sección de `device` encuentro el nombre del parámetro, 

![[Pasted image 20260316084645.png]]

y me quedo con la dirección de modbus la cual utilizare para modificar ese parametro en el dispositivo a traves del programa [[AKO 44 - Configurar Alarmas desde Cloud o en Crudo]]
## Configurar alarmas desde cloud, o en crudo desde el device directamente
[[AKO 44 - Configurar Alarmas desde Cloud o en Crudo]]
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


## Errores

[[AKO 44 - Errores y sus soluciónes]]
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
## Features

[[AKO44 - Features Main Site]]
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

## Indicadores

[Recurso de Excel](https://ako0-my.sharepoint.com/:x:/g/personal/fmaataoui_ako_com/IQC6HFUUkj4ER5nMrNHnTUHvAVrogbp4fs-CyKxK4Mn6QoY?wdOrigin=TEAMS-MAGLEV.p2p_ns.rwc&wdExp=TEAMS-TREATMENT&wdhostclicktime=1772711678220&web=1)


>[!warning] credenciales chat gpt
[sw.support@ako.com](mailto:sw.support@ako.com "mailto:sw.support@ako.com")
pass: vbWj8Ur4WWxX
