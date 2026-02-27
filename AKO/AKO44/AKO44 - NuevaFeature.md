que se devuelva los eventos y alarmas activas
osea que cuando se haga un select con: eventActive alarmActive
que te devuelva en la repsuesta un array de eventos y alarmas activas
creo que lo mejor es crear un controller para las alarmas y eventos perte, y que haya un endpoint de eventsActive y alarmsActive y cuando lo pasas en el select de /device llame a la funcion de eventsActive ene l controller de Events
y alarmsActive en el controller de Alarms


porque los eventos, `updater.perte`el event.updates ya los escribe en la coleccion de eventos y alarmas