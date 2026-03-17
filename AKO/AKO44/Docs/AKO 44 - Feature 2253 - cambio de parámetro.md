
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