Necesitamos devolver eventos y alarmas activas, `cuando un select con: eventActive alarmActive` que devuelva la respuesta un array de eventos y alarmas activas, 

aqui es donde se escriben los eventos en las colecciones de perte correspondientes![[Pasted image 20260227133455.png]]
En `micros.status.perte.updates` es mejor no meter lógica de eventos. Mejor hacer dos nuevo controller en  `api.controllers.eventPerte/alarmPerte` basandome en el ``api.controllers.event` para que este devuelva los eventos y alarmas de perte activos para ese dispositivo

mover los modelos `event-perte, alarm-perte` a la api 

IDEM al alarmsActive