Implementado nuevos valores para status en el modelo.

![[Pasted image 20260311084042.png]]

La arquitectura actual implementa métodos de `update` propios en el controlador abstracto y implementaciones de este en el controlador de device.
Pero ya que hacemos una comprobación de permisos previa llamaremos al método `findOneUpdate` del model directamente en controlador de device
Update del campo con mongo [[Mongo - Main Site#^ff361f]]. 

Después de chequear si el dispositivo es nuevo seguiremos con el flujo nuevo, sino continuara  de manera normal 

![[Pasted image 20260311084158.png]]