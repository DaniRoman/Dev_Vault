
> ****Regla de notificación****: Entidad que define cómo y cuando se notifica una alarma.

- **Calendario**: día/hora
- **Severidad** (priority: low/medium/high)
- **Destinatarios** (usuarios, devices, grupos)
- **Notificaciones**: un array de acciones con `type` (email, sms, push, **call**, relay), destinatarios, y si requiere acuse (`ack`)
	- Modo Secuencial: ando es `true`, las notificaciones del array se ejecutan **una a una**: si la primera es confirmada (ACK), se detiene la cadena; si no, pasa a la siguiente. Cuando es `false`, todas se disparan en paralelo.
- **Retardos** y **repeticiones**

## Protocolos de red
[[AKO 44 - Protocolos]]
## Diferencia entre CMD y Parámetro
[[AKO 44 Diferencia entre CMD y Param]]