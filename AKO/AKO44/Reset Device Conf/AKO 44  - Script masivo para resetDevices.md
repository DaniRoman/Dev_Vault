
## Qué hace el script

El script nace de una necesidad concreta: hay dispositivos en el sistema cuya configuración se ha desviado de los valores que define su `DeviceDefinition` — ya sea por cambios manuales, migraciones o actualizaciones de firmware. En lugar de corregirlos uno a uno desde la UI, este script automatiza el reset masivo llamando al endpoint que ya existe en la API.

---

### El flujo en palabras

Antes de tocar nada, el script valida que tengas las tres variables de entorno necesarias: la URL de la API, el usuario y la contraseña. Si falta alguna, para inmediatamente y te dice cuál.

Lo primero que hace con la API es autenticarse. Hace un `POST /api/auth` con tus credenciales y guarda el token que recibe. Ese token lo va a usar en todas las llamadas siguientes como header `x-authtoken`. Si la autenticación falla, el script se detiene — no tiene sentido continuar sin permisos.

Con el token en mano, obtiene la lista completa de devices. Como podrían ser miles, no los pide todos de golpe: va haciendo peticiones en bloques de 500 (`skip` + `limit`) hasta que ha traído el total. De cada device solo pide `_id`, `serialNumber` y `name` — lo mínimo necesario.

Aquí entra el **modo dry-run**, que es el comportamiento por defecto. Si no pasas `--execute`, el script simplemente imprime la lista de devices que se verían afectados y termina sin hacer nada. Esto te permite revisar el alcance antes de comprometerte.

Si pasas `--execute`, el script recorre cada device uno a uno y llama `PUT /api/device/reset/:id`. Lo que ocurre dentro del servidor en ese momento es: busca el `DeviceDefinition` asociado al device, toma los valores por defecto que define, y sobreescribe la configuración actual del device con esos defaults. El script loguea en tiempo real si cada device fue OK o si falló.

Al final imprime un resumen: cuántos salieron bien, cuántos fallaron, y la lista completa de los que fallaron con el error de cada uno. Si hubo algún error, termina con código de salida 1 — útil si algún día se integra en un pipeline.

---

### Por qué está diseñado así

**Sequencial en lugar de paralelo** — podría procesar varios devices a la vez, pero hacerlo en serie es más seguro: no satura la API, los logs son legibles en orden y si algo falla es más fácil identificar dónde.

**Dry-run por defecto** — la operación es irreversible (sobreescribe configuración), así que la primera ejecución siempre es segura. Hay que optar activamente por `--execute`.

**`--limit N`** — pensado para validar en local o staging antes de soltar el script contra todos los devices de producción. Ejecutas con `--limit 10`, co

```mermaid
flowchart TD
    A(["node resetAllDevicesConfiguration.js"]) --> B{"Valida env vars"}

    B -->|faltan vars| Z1(["Exit 1<br/>nota que falta"])
    B -->|OK| C["Muestra modo<br/>DRY RUN o EXECUTE"]

    C --> D["POST /api/auth?isAdminApp=true"]:::api
    NOTE1["Necesita:<br/>API_URL<br/>AUTH_USER<br/>AUTH_PASS"]:::note
    NOTE1 -.-> D

    D -->|sin authToken| Z2(["Exit 1<br/>authentication failed"])
    D -->|authToken OK| E["Pagina GET /api/device<br/>bloques de 500"]:::api

    NOTE2["Necesita:<br/>x-authtoken header<br/>isAdminApp=true<br/>select: _id serialNumber name"]:::note
    NOTE2 -.-> E

    E --> F{"--limit flag?"}
    F -->|si| G["Recorta lista<br/>al N indicado"]
    F -->|no| H["Usa lista completa"]

    G --> I{"Modo?"}
    H --> I

    I -->|DRY RUN| J["Imprime lista<br/>de devices afectados"]
    J --> K(["Exit 0<br/>sin cambios"])

    I -->|EXECUTE| L["Loop secuencial<br/>por cada device"]

    L --> M["PUT /api/device/reset/:id<br/>?isAdminApp=true"]:::api

    NOTE3["Necesita:<br/>x-authtoken header<br/>device._id como path param<br/>HTTP 204 = exito"]:::note
    NOTE3 -.-> M

    M -->|204 OK| N["Log OK + contador"]
    M -->|error HTTP| O["Log ERR + guarda en lista"]

    N --> P{"Quedan devices?"}
    O --> P

    P -->|si| L
    P -->|no| Q["Imprime resumen<br/>OK / Errors / Total"]

    Q --> R{"Hubo errores?"}
    R -->|si| Z3(["Exit 1<br/>lista devices fallidos"])
    R -->|no| S(["Exit 0"])

    classDef api fill:#1e3a5f,stroke:#4a90d9,color:#ffffff
    classDef note fill:#2d2d2d,stroke:#666666,color:#dddddd
```


