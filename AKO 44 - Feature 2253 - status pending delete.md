La arquitectura actual implementa métodos de `update` propios en el controlador abstracto y implementaciónes de este en el controlador de device.
Pero ya que hacemos una comprovacion de permisos previa llamaremos al método `findOneUpdate` del model directamente en controlador de device
Update del campo con mongo [[Mongo - Main Site#^ff361f]]