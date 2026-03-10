>[!example] Feature 2253
>[Enlace a tarea](https://ako-team.atlassian.net/browse/KNT-2253)

Cuando Elimino un Device desde el endpoint.`https://api.dev.akonet.cloud/api/device/:id/remove`
cambiare `param_lic_sys_state`  a `2`

![[Pasted image 20260309094804.png|242]]

*Figura1 `api/schemas/definition/panel_2ry_7102.json`*

>[!tip] Recuperar un endpoint mediante navegador
>Para saber el endponit al que apuntar desde el navegador en `devtools > network` puedo ver las llamadas que se hicieron a la `api`


>[!tip] A tener en cuenta
>Cuando un device es eliminado este todavía existe en `manufacturedDevice`como algo que salió de fabrica.

Mirando las especificaciones del dispositivo en el [excel](https://ako0.sharepoint.com/:x:/r/teams/DesarrolloModificaciondeProducto/NEWDARWIN/_layouts/15/Doc.aspx?sourcedoc=%7B514EB616-8124-49F9-A674-3C8150BBA1E0%7D&file=2024_NEW_DARWIN_PANEL_2RY_REV5.xlsx&action=default&mobileredirect=true)  en el grupo/pestaña `conf.LIC` podemos ver los posibles valores.

>[!Success] Flujo de micros para un cambio de parámetros
EL procedimiento para un cambio de parámetro es mediante el flujo de 
[[AKO 44 - Flujo de cambio de parámetros Cloud - Device.canvas]]

Para hacer un cambio de parámetros en el dispositivo lo haremos a traves de `<cmd>` la información de como se muestra este en el `payload` y sus diferentes opciones las encontrare en el documento del protocolo de comunicación en la sección de `JSON cmd`  [Documento](https://ako0.sharepoint.com/:w:/t/DesarrolloModificaciondeProducto/DEVICE_COMMUNICATION_PROTOCOL/IQANIw2Uzjg2QpMzVO0zj5Z5AWI-WY01IyDjBINtVQTB-ck?e=bLD5hS&ovuser=5a94156b-5d3f-467b-b767-561717bb62ca%2Cdaniel.roman%40ako.com&clickparams=eyJBcHBOYW1lIjoiVGVhbXMtRGVza3RvcCIsIkFwcFZlcnNpb24iOiI1MC8yNjAxMDQwMDkyNSIsIkhhc0ZlZGVyYXRlZFVzZXIiOmZhbHNlfQ%3D%3D)

Un punto a tener en cuenta es que existen dos tipos de `comandos`,  `cmd` 
- Entiende xxx, para ver los disponibles mirare en el Excel de las especificaciones técnicas en la pestaña de `CMD Server` 
	- Para mandar ese cambio utilizare la función de la api `controllers/api/configuration/send-amqp/sendCmd12830` 
- El dispositivo lo entiende como un cambio de valor en los parámetros disponibles para ese este. 
	- Para mandar ese cambio utilizare la función de la api `controllers/api/configuration/send-amqp/sendConfig12830`

cmd en Tendremos que enviar un `cmd` con ese cambio en configuración 
>[!tip] Explicar tema CMD y que funcion usa cada uno cmd o param 


---

>[!warning]
Una ve se mande la auditoria con el 530 que se autodestrulla



