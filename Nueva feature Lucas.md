>[!example] Feature 2253
>[Enlace a tarea](https://ako-team.atlassian.net/browse/KNT-2253)

Cuando Elimino un Manufactured Device el parámetro `param_lic_sys_state` definido en el `panel_2ry_7102.json` 
![[Pasted image 20260309094804.png|242]]
que si miramos en el `spech` del [excel](https://ako0.sharepoint.com/:x:/r/teams/DesarrolloModificaciondeProducto/NEWDARWIN/_layouts/15/Doc.aspx?sourcedoc=%7B514EB616-8124-49F9-A674-3C8150BBA1E0%7D&file=2024_NEW_DARWIN_PANEL_2RY_REV5.xlsx&action=default&mobileredirect=true) en `conf.LIC` podemos ver sus valores posibles y el numero `2` sera el que nos interese cambiar en el momento que borremos un `manufactured` a través del siguiente endpoint `https://api.dev.akonet.cloud/api/device/:id/remove` l tendremos que enviar un `cmd` con ese cambio en configuración tiene que cambiar a 2, pero este cambio solo sera para los dispositivos `new` con lo que me puedo ayudar del código de `api > lib > helpers > device-classifier.ts`.
Con todo esto es tipo de mensaje con su cuerpo válido para enviar lo encontrare en `src/controller/send-amqp/sendCmd12830` 

KNT-2253


param para aplicación plantillas y dar de alta
los comands, tanto comands como cambio parametro c

dos tipos de cmd depende el tipo aue venga eso lo veo en el texto always on device comminication protocol ,  esta como notas. 

cmd se comporta de dos maneras

cuando es menor que 100 cada dispositiovo tienen sus cmds cmd server en el exvel lo puedo ver ,  en device defitions el campo comands  cmd va al conexion layer , y luego translator.

si es mayo sera un cambio de parametros. sendConfig12830

>[!warning]
Una ve se mande la auditoria con el 530 que se autodestrulla
