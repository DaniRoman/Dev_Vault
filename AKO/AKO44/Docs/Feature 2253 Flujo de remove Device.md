>[!example] Feature 2253
>[Enlace a tarea](https://ako-team.atlassian.net/browse/KNT-2253)

***Objetivo***
Al Eliminar un `Device` este tiene que setearse en`status: pending delete` [[AKO 44 - Feature 2253 - status pending delete]] . Se le envía un `cmd` para  cambiar el `param_lic_sys_state` =  `2`.
[[AKO 44 - Feature 2253 - cambio de parámetro]]
Una vez tenemos una confirmación `ACK: ok` [[AKO 44 - Feature 2253 - Confirmación ACK ok ]]Device tiene que mandar un `audit` con param_lic_sys_state =  2 y en ese micro `backlog.perte stateController` comprobar antes de borrar el Device ese audit y que `status: pending delete` 
`ManufacturedDevice` se mostara como desactivado

>[!tip] Ruta para empezar el flujo de borrado
>`https://api.dev.akonet.cloud/api/device/:id/remove`
>>[!tip] Recuperar un endpoint mediante navegador
>Para saber el endponit al que apuntar desde el navegador en `devtools > network` puedo ver las llamadas que se hicieron a la `api`

>[!success] Flujo para recibir el ACK y el audit del Device 
> [[AKO 44 Diagrama de flujo ack y pending output en cambio de parámetro Cloud - Device - Cloud.canvas]]
> >[!Success] Flujo de micros para un cambio de parámetros
EL procedimiento para un cambio de parámetro es mediante el flujo de 
[[AKO 44 - Flujo de cambio de parámetros Cloud - Device.canvas]]

 >[!tip] A tener en cuenta
>Cuando un device es eliminado este todavía existe en `manufacturedDevice`como algo que salió de fabrica.

---

***Búsqueda del  parametro***

![[Pasted image 20260309094804.png|242]]

*Figura1 `api/schemas/definition/panel_2ry_7102.json`*

Mirando las especificaciones del dispositivo en el [excel](https://ako0.sharepoint.com/:x:/r/teams/DesarrolloModificaciondeProducto/NEWDARWIN/_layouts/15/Doc.aspx?sourcedoc=%7B514EB616-8124-49F9-A674-3C8150BBA1E0%7D&file=2024_NEW_DARWIN_PANEL_2RY_REV5.xlsx&action=default&mobileredirect=true)  en el grupo/pestaña `conf.LIC` podemos ver los posibles valores.


## AKO 44 - Admin
[[AKO  44 - Admin Main Page]]