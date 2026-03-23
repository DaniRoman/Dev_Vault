Conexión layer 

Administración > FirmwareUpdate > Launch firmware, buscar este endPoint y hacer un fork, si es nuevo va a un topic si es viejo a otro

Esos topics me llevan a la conexión layer para que esta envie un cmd con petición de firmware update

En el Script de pruebas tengo que ver como puedo hacer un envio para solicitar el _/fota_ o el _/PLMN_

Una vez echa esa solicitud en la conexión layer llega esa petición y cuando llega una petición _/fota_ o el _/PLMN_ tiene que devolver el ultimo fichero o firmware disponible para ese numero de serie



`/Fota` y `/PLMN` son **resources/endpoints de retrieve del dispositivo** por CoAP. El dispositivo hace una petición tipo:

- `.../fota`
- `.../plmn`

y el backend le responde con un fichero o contenido concreto.

 `/Fota`, FOTA` = **Firmware Over The Air**
- actualización remota de firmware del dispositivo, el flujo lo trata como acción `fota` y busca el firmware pendiente para ese device.
- el dispositivo pregunta “¿tengo firmware nuevo?”
- cloud responde con el binario/contenido del firmware si existe

## Flujo `Device` envia `Request fota`

>[!tip] Flujo envio `ack` y `/fota`
>El Device una vez envia el mensaje `ack` después no enviara un `audit` (esto lo hace solo para parámetros) en ves de esto enviara otro tipo de mensaje (Buscar mensaje)
 
 Para que el device entre en `/fota`, en `client-perte` necesitas:

El `cmd_ack` solo confirma la orden.

Después, el cliente/device tiene que lanzar explícitamente una **request GET `/${serial}/fota`** para descargar el firmware. Y para que responda con firmware, tienen que cumplirse estas condiciones

Aunque hagas bien el `GET /fota`, si no hay match te devolverá error.
Que el device tenga:

- `deviceDefinition`
- `commercialVersion`
- `firmwareVersion`
- licencia/estado válido
- existencia en `device` y `manufactured`

Y que exista un `firmwareUpdate` compatible.

### Hay dos flujos distintos en Perte:

### 1) Mensajes normales del device
- método `POST`
- llevan `payload`
- ese payload va en **CBOR**
- y se protege con **CRC**
- ejemplos: `sample`, `status`, `event`, `cmd_ack`, `param_ack`

### 2) Retrieve de firmware
- método `GET`
- ruta `/{serial}/fota`
- **no depende de `ty`**
- entra por la URL, no por el body

La intención ya va en la **URL**, no en un payload serializado. El CRC se usa para validar el **payload recibido**.

Si no mandas payload:
- no hay nada que serializar
- no hay nada que firmar con CRC

---

### El device pide el firmware

### Después, el device hace un `GET /{serial}/fota`
 `cl-perte` entra en `processFota()`
El backend:

- identifica `request.method === "GET"`
- ve que la URL acaba en `/fota`
- busca el `firmwareUpdate` compatible para ese device

### 10) Si encuentra match
Hace esto:

- localiza el `file`
- lee el binario
- lo devuelve al device en la respuesta CoAP

### 11) El device recibe el firmware
Y a partir de ahí ya hace su lógica interna de actualización.

