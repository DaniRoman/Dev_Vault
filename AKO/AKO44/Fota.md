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


## Fase 2: el cloud prepara la actualización

### 5) Existe un `firmwareUpdate` activo
Tiene que haber uno compatible con el device, por ejemplo con match por:

- `deviceDefinition`
- `commercialVersion`
- `minVersion <= firmwareVersion del device`
- `active: true`
- `file` válido

Si esto no existe, `/fota` no encontrará nada.

---

## Fase 3: el cloud avisa al device

### 6) La API / backend genera el comando
Tu flujo actual es:

- entra algo por API
- eso provoca que se genere un `cmd` hacia el device
- el device recibe la orden de que hay actualización

### 7) El device responde con `cmd_ack`
Ese `ack` solo significa:

- “he recibido la orden”

Pero **todavía no ha descargado el firmware**.

---

## Fase 4: el device pide el firmware

### 8) Después, el device hace un `GET /{serial}/fota`
Aquí está el punto importante:

- no manda `ty: "fota"`
- no manda payload CBOR
- no manda CRC
- simplemente hace un `GET` a la URL `/fota`

### 9) En `cl-perte` entra en `processFota()`
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

---

## Resumen en una sola línea

### Flujo completo
1. `status`  
2. se guarda `firmwareVersion` en el `device`  
3. la API genera `cmd` de actualización  
4. el device responde `cmd_ack`  
5. el device hace `GET /{serial}/fota`  
6. el backend encuentra el `firmwareUpdate` y le devuelve el binario

---

## Qué depende de qué

### El `status` sirve para:
- rellenar `firmwareVersion`

### El `cmd_ack` sirve para:
- confirmar que recibió la orden

### El `GET /fota` sirve para:
- descargar realmente el firmware

---

## Dónde se suele romper

### Caso 1: el device no ha mandado `status`
Entonces:

- `firmwareVersion` queda vacío
- FOTA no encuentra update

### Caso 2: no existe `firmwareUpdate` compatible
Entonces:

- entra en `/fota`
- pero no encuentra nada que servir

### Caso 3: el device nunca hace `GET /fota`
Entonces:

- aunque haya `cmd_ack`
- nunca descarga el firmware

---

## Chuleta rápida para ti

### Para que un device nuevo reciba firmware:
1. crear device
2. mandar `status`
3. comprobar en BD que ya tiene `firmwareVersion`
4. tener un `firmwareUpdate` compatible
5. lanzar el comando de actualización
6. recibir `cmd_ack`
7. hacer `GET /{serial}/fota`
8. recibir el binario

---

## Micros implicados en ese flujo

### Para el `status`
- `cl-perte/perte-coap`
- `driver/translator.perte`
- `status.perte/device`

### Para servir el firmware
- `cl-perte/perte-coap`

### Para generar/enviar el comando
- depende del flujo de API/comandos que uséis, pero suele pasar por el traductor y salida CoAP

---

## Si quieres, te hago el mismo flujo pero en formato “diagrama” o “paso a paso con logs a revisar”.
