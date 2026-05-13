Modo Live En detalles del dispositivo, este tiene la opción de mostrar sus datos en vivo `Live` esto manda una petición al `Device` después de que este envié un `ack` nos mandara  los datos.
![[Pasted image 20260513093347.png]]
![[Pasted image 20260513093405.png]]


Como saber a que corresponde `sy` los símbolos que vemos en la pantalla del dispositivo.

![[Pasted image 20260513102153.png]]

![[Pasted image 20260513102129.png]]

Si pasamos el numero 8615 a `Binario` y leemos este al reves (Es el orden como se lee el binario tenemos que la primera posición empezando desde la derecha es un true)
![[Pasted image 20260513102300.png]]

Mirando en la definición tenemos que para esa posición corresponde el `ds_ako` (La `A` de `AKO` en la parte superior izquierda en el display del panel)

![[Pasted image 20260513102610.png]]

Y mirando en el translator el cual ya hizo el parse, vemos que efectivamente esta en `True` 

![[Pasted image 20260513110828.png]]

