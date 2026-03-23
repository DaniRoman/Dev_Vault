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

código
- `src/micros/cl-perte/perte-coap/microservice.ts` → detecta `request.url.endsWith("/fota")`
- `src/micros/cl-perte/cl-lib/process-message/messageProcessor.ts` → `processFota(...)`

---

`PLMN` = **Public Land Mobile Network**

En telecom significa:
- información/listado de red móvil
- operador móvil / tabla PLMN usada por el módem
- parece servir para que el dispositivo descargue un **fichero de lista/configuración PLMN**.
- el dispositivo pide la lista PLMN
- cloud responde con el archivo PLMN correspondiente

 `código`

- `src/micros/cl-perte/perte-coap/microservice.ts` → detecta `request.url.endsWith("/plmn")`
- `src/micros/cl-perte/cl-lib/process-message/messageProcessor.ts` → `processPlmn(...)`
- además en `sync.perte` se guarda info como:
  - `plmn_op`
  - `plmn_rev`

Eso indica que el device también informa de:
- operador PLMN actual
- revisión de tabla PLMN

---

### firmwareUpdate 

Es como una oferta de actualización disponible, un Modelo/documento de base de datos que **apunta a un archivo** de firmware, define para qué devices aplica, y qué **versión** representa.

- `version` → versión del firmware a instalar
- `deviceDefinition` → a qué familia de dispositivos aplica
- `commercialVersion[]` → a qué variantes comerciales aplica
- `minVersion` → desde qué versión mínima se permite actualizar
- `file` / `filePath` → el binario real del firmware
- `active` → si está publicado o no

---

***Como añadir un `firmwareupdate` para mi `device`***
- Crear un `firmwareUpdate` nuevo con una `deviceDefinition` igual al de mi `device` 
- En el `deviceDefinition` esta definido la `commercialVersion` de los dispositivos para esa definición 

>[!warning]
>Asegurarme que mi `device` coincida con esa `commercialVersion` y `comercialName` y también el `firmwareUpdate` que cree

- Fijarme en la `firmwareVersion` de mi `device` que no sea menor que `minVersion` que marca el la que marca el `firmwareUpdate` de ser asi no se mostrara en la pantalla administrador.
- En el administrador Selecciono Commercial Version y subo un archivo Random.
- En `LaunchFirmware` selecciono la compañía y introduzco dígito a dígito el serial number de mi Device 


---

Segun el documento always on Device este es el protocolo para actualizar un firmware

```js
{
  "id": 974130021,
  "ty": "cmd",
  "bid": 1,
  "d": [
    [5, 1]
  ]
}
```

>[!warning] Firmaware Version
>`firmware` esta es la versión inmutable del dispositivo, `firmware revision` es la actualización de la versión base `firmaware subrevision` esta es la actualización actual de `firmware revision` 

>[!tip] Flujo envio `ack` y `/fota`
>El Device una vez envia el mensaje `ack` después no enviara un `audit` (esto lo hace solo para parámetros) en ves de esto enviara otro tipo de mensaje (Buscar mensaje)

### Para que el device entre en `/fota`, en `client-perte` necesitas:

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


## Porque `/fota` no va por el carril de “mensaje del device”

Hay dos flujos distintos en Perte:

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

---

## No hace falta CBOR

Porque **no estás enviando un mensaje** con campos como `ty`, `id`, `d`, etc.

En `fota` lo único que le dice al backend lo que quieres es:

- el método: `GET`
- la ruta: `/123456/fota`

O sea, la intención ya va en la **URL**, no en un payload serializado. El CRC se usa para validar el **payload recibido**.

Si no mandas payload:
- no hay nada que serializar
- no hay nada que firmar con CRC

Y además el propio `CheckIntegrity` está preparado para eso:
- si el payload no existe o mide menos de 4 bytes
- devuelve `success` con payload vacío

Es decir:
- para `GET /fota`
- el backend **tolera request vacía**

---

## La idea clave

### En `POST`
el backend entiende **qué quieres** por el **payload**

### En `GET /fota`
el backend entiende **qué quieres** por la **ruta**

Por eso:

- `POST` → CBOR + CRC
- `GET /fota` → sin body, sin CBOR, sin CRC


Si mandaras CBOR/CRC en `fota`, no estarías siguiendo el patrón que usa vuestro micro para retrieve.

El micro decide `fota` por algo como:

- `request.method === "GET"`
- `request.url.endsWith("/fota")`

No por el contenido del body.

---

## Resumen corto

**Sin CBOR ni CRC porque `/fota` no es un mensaje del traductor, sino una petición retrieve por `GET`, y en vuestro backend esa petición se identifica por la URL y acepta payload vacío.**

Si quieres, te explico ahora **por qué en cambio el `cmd_ack` sí necesita CBOR + CRC**.
