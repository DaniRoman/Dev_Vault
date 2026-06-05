>***Que es comm_sync***

---

>***Descripción de tarea***

Añadir mensaje `comm_sync`  a mensajes permitidos por el `cloud/device` en el `translator` 
Ese comando se gestionara en el micro `Comands12830 > comm_sync` mediante su `handler` y `topic` asociado haciendo exactamente lo mismo que ya esta haciendo el micro `Comands12830 sync` en la parte de `hardaware` y `plmn` y  estas quitarlas del handler del `sync`  
En el translator el mensaje del `sync` los campos `hardware` y `plmn` serán opcionales 

---

>***comm_sync workFlow***

[[AKO 44 - comm_syn workflow.canvas]]

---

>****Aceptar nuevo mensaje del `Device` en el translator***

[[AKO 44 - Aceptar nuevo mensaje del device en el translator]]