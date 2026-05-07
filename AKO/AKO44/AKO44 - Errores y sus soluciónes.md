>[!error] `Device` comunica pero no cambia su estado a `online`
>

![[imagen (2).png]]
![[imagen (1).png|697]]

En los logs puede verse que el `Device` comunica pero algo hace que el mensaje no llegue a ser tratado por su colección

>[!tip] Solución

Mirar que formato de mensaje esta enviando el device por ejemplo el de `status`

![[imagen (3).png]]

Lo checkeo con el documento  [`always on device`](https://ako0.sharepoint.com/:w:/t/DesarrolloModificaciondeProducto/DEVICE_COMMUNICATION_PROTOCOL/IQANIw2Uzjg2QpMzVO0zj5Z5AWI-WY01IyDjBINtVQTB-ck?e=bLD5hS&ovuser=5a94156b-5d3f-467b-b767-561717bb62ca%2Cdaniel.roman%40ako.com&clickparams=eyJBcHBOYW1lIjoiVGVhbXMtRGVza3RvcCIsIkFwcFZlcnNpb24iOiI1MC8yNjAxMDQwMDkyNSIsIkhhc0ZlZGVyYXRlZFVzZXIiOmZhbHNlfQ%3D%3D) 

![[Pasted image 20260507085026.png]]

En la foto anterior vemos que en `d1 y d2`recibimos `null` pero en el `documento` no el siguiente paso seria mirar el `schema` del `translator` para ver que tipo de mensajes y tipos soporta 

---
