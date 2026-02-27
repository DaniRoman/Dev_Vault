
- Necesitamos devolver eventos y alarmas activas, `cuando un select con: eventActive alarmActive` que devuelva la respuesta un array de eventos y alarmas activas, 

- En `micros.event.updater.perte.microservice`es donde se escriben los eventos en las colecciones de perte correspondientes

- En `micros.status.perte.updates` es mejor no meter lógica de eventos. Mejor hacer dos nuevo controller en  `api.controllers.eventPerte/alarmPerte` basandome en el ``api.controllers.event` para que este devuelva los eventos y alarmas de perte activos para ese dispositivo

- Luego filtrar por eventos, alarmas activas y que desde el controlador de device añades un nuevo select , para que si te lleva eventsActive, que llame al controllador de EventPerte y devolver los eventos solo activos.

- Mover los modelos `event-perte, alarm-perte` a la api 

