>[!example] Feature 2253
>[Enlace a tarea](https://ako-team.atlassian.net/browse/KNT-2253)

***Objetivo***
Al Eliminar un `Device` este tiene que setearse en[[`status: pending delete`]] *(Este estado no existe)*. Se le envía un `cmd` para[[ cambiar el `param_lic_sys_state` =  `2`.]] 
Una vez tenemos una confirmación `ACK: ok` Device tiene que mandar un `audit` con param_lic_sys_state =  2 y en ese micro `backlog.perte stateController` comprobar antes de borrar el Device ese audit y que `status: pending delete` 
`ManufacturedDevice` se mostara como desactivado

>[!tip] Ruta para empezar el flujo de borrado
>`https://api.dev.akonet.cloud/api/device/:id/remove`
>>[!tip] Recuperar un endpoint mediante navegador
>Para saber el endponit al que apuntar desde el navegador en `devtools > network` puedo ver las llamadas que se hicieron a la `api`

>[!warning] Flujo de retry x 3
*(cerciorase que el flujo de `retry` para el caso de que el dispositivo no me envio una confirmación se da 3 veces)*

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


***Cambio de parametro ***

Para hacer un cambio de parámetros en el dispositivo lo haremos a través de `<cmd>` la información de como se muestra este en el `payload` y sus diferentes opciones las encontrare en el documento del protocolo de comunicación en la sección de `JSON cmd`  [Documento](https://ako0.sharepoint.com/:w:/t/DesarrolloModificaciondeProducto/DEVICE_COMMUNICATION_PROTOCOL/IQANIw2Uzjg2QpMzVO0zj5Z5AWI-WY01IyDjBINtVQTB-ck?e=bLD5hS&ovuser=5a94156b-5d3f-467b-b767-561717bb62ca%2Cdaniel.roman%40ako.com&clickparams=eyJBcHBOYW1lIjoiVGVhbXMtRGVza3RvcCIsIkFwcFZlcnNpb24iOiI1MC8yNjAxMDQwMDkyNSIsIkhhc0ZlZGVyYXRlZFVzZXIiOmZhbHNlfQ%3D%3D)

Un punto a tener en cuenta es que `cmd` se comporta de dos maneras
1. Cuando es `< 100 ` define una serie de comandos `<cmd>` específicos de cada dispositivo estos los puede ver en `deviceDefinition > commands` También en el excel en la pestaña de `cmd server`
	- Para mandar ese cambio utilizare la función de la api `controllers/api/configuration/send-amqp/sendCmd12830` 

>[!tip] Concepto
>Cuando nos referimos a <> que 100 hacemos referencia al valor que aparece en el payload en la posición que define el fichero de protocolo de comunicación, en este caso 530 - valor para ese conf
>![[Pasted image 20260310121655.png]]
>_figura1 translator layer_

2. `> 100` El dispositivo lo entiende como un cambio de valor en los parámetros disponibles para ese este. 
	- Para mandar ese cambio utilizare la función de la api `controllers/api/configuration/send-amqp/sendConfig12830`



>[!warning]
Una ve se mande la auditoria con el 530 que se autodestrulla



