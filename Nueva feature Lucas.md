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

Mirando especificaciones del dispositivo en el [excel](https://ako0.sharepoint.com/:x:/r/teams/DesarrolloModificaciondeProducto/NEWDARWIN/_layouts/15/Doc.aspx?sourcedoc=%7B514EB616-8124-49F9-A674-3C8150BBA1E0%7D&file=2024_NEW_DARWIN_PANEL_2RY_REV5.xlsx&action=default&mobileredirect=true) registro `conf.LIC` podemos ver los posibles valores.

EL procedimiento para un cambio de parámetro es mediante el flujo de 
[[AKO 44 - Flujo de cambio de parámetros Cloud - Device.canvas]]

La explicación de que es un `cmd` la encuentro en las notas de la deficniones 

cmd en Tendremos que enviar un `cmd` con ese cambio en configuración 
>[!tip] Explicar tema CMD y que funcion usa cada uno cmd o param [protocolo de comunicación](https://ako0.sharepoint.com/:w:/t/DesarrolloModificaciondeProducto/DEVICE_COMMUNICATION_PROTOCOL/IQANIw2Uzjg2QpMzVO0zj5Z5AWI-WY01IyDjBINtVQTB-ck?e=bLD5hS&ovuser=5a94156b-5d3f-467b-b767-561717bb62ca%2Cdaniel.roman%40ako.com&clickparams=eyJBcHBOYW1lIjoiVGVhbXMtRGVza3RvcCIsIkFwcFZlcnNpb24iOiI1MC8yNjAxMDQwMDkyNSIsIkhhc0ZlZGVyYXRlZFVzZXIiOmZhbHNlfQ%3D%3D)

tiene que cambiar a 2, pero este cambio solo sera para los dispositivos `new` con lo que me puedo ayudar del código de `api > lib > helpers > device-classifier.ts`.
Con todo esto es tipo de mensaje con su cuerpo válido para enviar lo encontrare en `src/controller/send-amqp/sendConfig12830` 

KNT-2253


param para aplicación plantillas y dar de alta
los comands, tanto comands como cambio parametro c

dos tipos de cmd depende el tipo aue venga eso lo veo en el texto always on device comminication protocol ,  esta como notas. 

cmd se comporta de dos maneras

cuando es menor que 100 cada dispositiovo tienen sus cmds cmd server en el exvel lo puedo ver ,  en device defitions el campo comands  cmd va al conexion layer , y luego translator.

si es mayo sera un cambio de parametros. sendConfig12830

>[!warning]
Una ve se mande la auditoria con el 530 que se autodestrulla

>[!error] Error de `findOne`
>`[error]`: statecontroller.12830 - Failed to process backlog message: Cannot read properties of undefined (reading 'findOne')

Reiniciar los micros si se da este caso que los modelos no cargan por la base de datos.


