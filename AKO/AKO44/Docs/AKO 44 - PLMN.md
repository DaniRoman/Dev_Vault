## ¿Qué es PLMN?

**PLMN** = **Public Land Mobile Network** (Red Pública de Telefonía Móvil). Es una tabla que contiene información de operadores móviles NB-IoT/LTE-M que identifica a los operadores de red por dos números:

- **Operator**: Código identificador del operador (0-65535)
- **Release**: Versión/revisión de la tabla (0-65535)


>[!Example] Flujo `PLMN` [[AKO 44 - Flujo PLMN.canvas]] 

## Estructura del Sistema

El sistema permite gestionar tablas PLMN que se distribuyen a dispositivos específicos. Está compuesto por:

```ts
PlmnList
├── operator (número)
├── release (número)
├── plmnTable (string con números separados por saltos de línea)
├── commercialVersion (array)
├── deviceDefinition (array de dispositivos compatibles)
├── availableDevices (dispositivos que ya tienen esta actualización)
├── file (archivo binario generado)
└── name (nombre descriptivo)
```

### 2. **Endpoints API** del Controlador

|Endpoint|Método|Función|
|---|---|---|
|`/api/plmn-list`|GET|Listar todas las tablas PLMN|
|[/api/plmn-list/{id}](vscode-file://vscode-app/private/var/folders/ty/t6vhvm793c39txh5gvhywj0jbv8vf3/T/AppTranslocation/B11F9001-6F64-4CF4-A333-F6DB525B68CC/d/Visual%20Studio%20Code.app/Contents/Resources/app/out/vs/code/electron-browser/workbench/workbench.html)|GET|Obtener una tabla específica|
|`/api/plmn-list`|POST|Crear nueva tabla PLMN|
|`/api/plmn-list/{id}/launch`|POST|Lanzar actualización a dispositivos|
|`/api/plmn-list/{id}/binary`|GET|Descargar archivo binario|
|`/api/plmn-list/test`|POST|Validar tabla PLMN|

```
┌─────────────────────────────────────────────────────┐
│  1. CREAR TABLA PLMN (POST /api/plmn-list)         │
├─────────────────────────────────────────────────────┤
│ • Se recibe plmnTable (números separados por \n)   │
│ • Se valida con testTable()                        │
│ • Se verifica que no exista duplicada               │
│ • Se guarda en BD con references a dispositivos     │
│ • Se envía mensaje AMQP para generar archivo       │
└─────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────┐
│  2. VALIDAR TABLA (POST /api/plmn-list/test)       │
├─────────────────────────────────────────────────────┤
│ • Cada línea se valida como número                  │
│ • Se detectan: duplicados, bigInt > FFFFFFFF       │
│ • Responde con array de objetos validados          │
└─────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────┐
│  3. LANZAR ACTUALIZACIÓN (POST /api/plmn-list/{id} │
│     /launch)                                        │
├─────────────────────────────────────────────────────┤
│ • Se obtiene la tabla PLMN con definiciones        │
│ • Se seleccionan dispositivos objetivo             │
│ • Se compara operador/release con valores actuales │
│ • Si hay diferencias, se envía comando AMQP al     │
│   dispositivo con plmn_update (0, 1 ó 2)          │
│                                                    │
│ Acciones PLMN:                                     │
│  0 = Cancelar petición anterior                    │
│  1 = Actualizar (acción normal)                    │
│  2 = Actualizar desde firmware del dispositivo     │
└─────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────┐
│  4. DISPOSITIVO RECIBE ACTUALIZACIÓN               │
├─────────────────────────────────────────────────────┤
│ • El dispositivo recibe comando AMQP               │
│ • Compara su plmn.operator/release actual          │
│ • Si son diferentes, procesa la actualización      │
│ • El dispositivo reporta cambio (audit log)        │
└─────────────────────────────────────────────────────┘
```

>[!warning] porque no aparece panel_2ry en deviceDefinitionsList? admin plm ?
>Los modelos que acepta tienen que ir en `admin` `PlmnListEditComponent`

>[!tip] PLMN table
>Como se crea una tabla, que valores tiene que tener y que son estos.

>[!warning] Generar el path 
>Una vez generado en `admin` la nueva lista `plmn` hacemos un post a la `api`, y esta publica el mensaje al micro `commands.plm` este genera el path y guarda el fichero ***¿Guarda el fichero en este micro?***


