
- Necesitamos devolver eventos y alarmas activas, `cuando un select con: eventActive alarmActive` que devuelva la respuesta un array de eventos y alarmas activas, 

- En `micros.event.updater.perte.microservice`es donde se escriben los eventos en las colecciones de perte correspondientes

- En `micros.status.perte.updates` es mejor no meter lógica de eventos. Mejor hacer dos nuevo controller en  `api.controllers.eventPerte/alarmPerte` basandome en el ``api.controllers.event` para que este devuelva los eventos y alarmas de perte activos para ese dispositivo

- Que desde el controlador de device añades un nuevo select , para que si te lleva eventsActive, que llame al controllador de EventPerte y devolver los eventos solo activos.

- Mover los modelos `event-perte, alarm-perte` a la api 

- Ir al micro de `events updater.perte` y actualizar el campo `alarmCount` en `Device` cuando se activa o desactiva una alarma, el número de `alarmCount` ha de coincidir con lo que devuelve `alarmsActive`

[[AKO - feature/KNT-2243 - errores]]

