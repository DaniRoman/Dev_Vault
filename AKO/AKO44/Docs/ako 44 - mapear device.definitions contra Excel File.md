## Ubicación y explicación de los parámetros

>Recursos de excel y word para especificaciones [Recurso](https://ako0.sharepoint.com/teams/DesarrolloModificaciondeProducto/NEWDARWIN/Documentos%20compartidos/Forms/AllItems.aspx?id=%2Fteams%2FDesarrolloModificaciondeProducto%2FNEWDARWIN%2FDocumentos%20compartidos%2FEspecificaci%C3%B3n&viewid=415e4dff%2D290a%2D42c1%2Dad11%2D49b751afccbc) 

>[!important] importante
>Hay tres tipos de dispositivos, no comunicados, *(EDGE)*, *(NBIOT)*
> Tipo 3 hace referencia a dispositivos  *(EDGE)* tipo 4 a *(NBIOT)* Último dígito dígito me dice que protocolo utiliza, `2-CoAP` - `1-MQTT
> PV = Program versión, version, firmware del dispositivo.

![[Pasted image 20260210104331.png]]

![[Pasted image 20260210104134.png]]

*carpeta en el excel hace referencia a la pestaña del cloud*

![[Pasted image 20260210104047.png]]

*Y en el json* 

![[Pasted image 20260210112616.png]]

El nivel dos hace referencia al contenido de esa carpeta/pestaña

![[Pasted image 20260210112902.png]]

![[Pasted image 20260210113008.png]]

Y en el `json` lo encontraríamos en `conf`

![[Pasted image 20260210113231.png]]

La clave `code` hace referencia al `grupo/carpeta/ventana` - `nombre parametro`

En la carpeta `nb - config` el `columna parametro tx` parámetro `param_tx_interval`: así es como encontraremos en el código la variable nombrada para ese parámetro

![[Pasted image 20260210114102.png]]

Esta la encontramos en el `json > "conf" > "ref": "param_tx_interval" 

`Factor divisor` hace referencia a `json conf > modbus > convert`

![[Pasted image 20260210114739.png]]

![[Pasted image 20260210114818.png]]

### Traducción del mensaje con el device mediante ModBuss

Por ejemplo para un cambio de parámetros el device nos mandaría un mensaje tal.
Buscaria la referencia al modbus, substituiria el numero por el parámetro y dividiria el número por le factor de conversión

![[Pasted image 20260210115333.png]]
![[Pasted image 20260210115446.png]]

```txt
[520, 5, 521, 30]
[L2=5, TZ=30]
```