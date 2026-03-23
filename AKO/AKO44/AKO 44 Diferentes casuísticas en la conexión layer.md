La **connection layer distingue las casuísticas por dos cosas**:

- **el método HTTP/CoAP**: `GET` o `POST`
- **la ruta (`url`)**: si acaba en `/fota`, `/dfota`, `/plmn`, `/link` o en nada especial
    

Para algunas rutas retrieve la intención va en la URL, no en el body**. Cuando entra una petición del device, pasa por `_onMessage()`.

Ahí hace esto:

1. saca el **serialNumber** desde la URL
2. decide si la acción es:
    
    - `"retrieve"` si es `GET`
    - `"create"` si es `POST`
        
3. mira si la URL termina en algo especial:
    
    - `/fota`
    - `/dfota`
    - `/plmn`
    - `/link`
        
4. monta `messageParseCl`


- `POST` → `requestAction = create`, la intención suele venir en el payload.
- `GET` → `requestAction = retrieve`, la intención puede venir en la ruta.
- Si la URL acaba en `/fota`, `/dfota`, `/plmn` o `/link`, se rellena `retiveAction`.
- `/fota` no necesita body tipo traductor; la CL lo identifica por la URL.