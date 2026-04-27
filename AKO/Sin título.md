api/alar/id/

event

enviar info del evento quien ack y demas info importante esto en el endpoint 

12840.alar.ack


legacy, las alarmas llegan mediante event, is alarm
12830 llegan mediante alarm-12830


el cliente puede hacer un alarma recibida pulsando un boton y enviando un ACK en `/api/alarm/:id/ack`

una vez ahi dependiendo si es un modelo nuevo o un modelo viejo se tiene que gaurdar info de quien a echo el ack y se envia a la cola que gestiona esa alarma como apagarla 


Flujo actual 

```js
//Legacy
copyData.event.ack = {
	ackAt: new Date(),
	ackAtGMT: DateHelper.parseUTCOffset(new Date(), device.getUTCOffset()),
	ackBy: parseExternalUserEmail(this.context.user.username),
	comment: comment
}
//Se envia a
SendAmqp.sendEventRPC("event.ack", copyData, 50000)
```

