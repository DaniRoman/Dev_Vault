>Requerimientos 

>[!warning] Pasar a limpio

Ahora tengo todo el panorama completo. Aquí va la guía:

---

# Guía de implementación: nuevo tipo de mensaje en el translator.perte

## Arquitectura de capas (orden de ejecución)

```
Device (CoAP POST)
    ↓
[1] cl-perte/perte-coap          → Recibe el mensaje raw
    ↓ AMQP: driver.input.perte
[2] translator.perte/microservice.ts → handleInput() — orquesta todo
    ↓
[3] InputValidator                → valida firmware + schema
    ↓
[4] InputProcessor                → decide qué parser usar
    ↓
[5] Parser → XxxParser            → transforma el payload a dominio
    ↓
[6] publishParsedData()           → publica en el topic AMQP destino
    ↓
[7] Micro consumidor (events, status, net-sync…)
```

---

## Paso a paso: qué tocar y en qué orden

### Paso 1 — Definir el `MessageType`
**Archivo:** `src/perte-msg-types/globals-message.type.ts`

Añade el nuevo tipo al enum. El valor string es el que viene en el campo `ty` del payload del device:

```ts
export enum MessageType {
  // ...existentes...
  MY_NEW_MSG = "my_new_msg",   // ← añadir aquí
}
```

---

### Paso 2 — Crear el tipo e interfaz del mensaje
**Archivo:** `src/perte-msg-types/my-new-msg-message.type.ts` *(nuevo)*

Sigue el patrón de `sync-message.type.ts` o `net_sync-message.type.ts`. Dos cosas en el mismo fichero:

**A) Interfaz TypeScript** — estructura del payload que envía el device:
```ts
export interface MyNewMsgMessage {
  id: Id;      // array: [serialNumber, progVer, progRev, progSubrev, jsonSpec]
  ty: MessageType;
  ts: CurrentTimestamp;
  bid: BlockIdentification;
  d: Array<...>;   // datos específicos
}
```

**B) Schema JSON** — para validación en runtime:
```ts
export const MyNewMsgMessageSchema = {
  type: "object",
  properties: {
    id: { /* SyncIdSchema como referencia */ },
    ty: TySchema,
    ts: TsSchema,
    bid: NumberSchema,
    d:   { type: "array", ... },
  },
  required: ["id", "ty", "ts", "bid", "d"],
  additionalProperties: false,
};
```

---

### Paso 3 — Re-exportar desde el barrel de tipos
**Archivo:** `src/micros/driver/translator.perte/tl-lib/types/message-types.ts`

```ts
export {
  MyNewMsgMessage,
  MyNewMsgMessageSchema,
} from "../../../../../perte-msg-types/my-new-msg-message.type";
```

---

### Paso 4 — Definir la interfaz de salida (parsed)
**Archivo:** `src/micros/driver/translator.perte/tl-lib/types/tl-types.ts`

Define cómo quedará el mensaje **después** de parsearlo (lo que recibirá el micro consumidor):

```ts
export interface ParsedMyNewMsg {
  myNewMsg: {
    field1: string;
    field2: number;
    // ...
  };
}
```

Y añade el topic destino en el `TopicMap`:

```ts
export const TopicMap: Map<MessageType, string> = new Map([
  // ...existentes...
  [MessageType.MY_NEW_MSG, "perte.input.my_new_msg"],  // ← añadir
]);
```

---

### Paso 5 — Crear el parser
**Archivo:** `src/micros/driver/translator.perte/tl-lib/parsers/implementations/MyNewMsgParser.ts` *(nuevo)*

Implementa `IMessageParser<MyNewMsgMessage>`. Sigue el patrón de `NetSyncParser`:

```ts
import { IMessageParser } from "./IMessageParser";
import { ClOutgoingMessage } from "../../types/cl-types";
import { DefinitionReader } from "../../services/DefinitionReader";
import { MyNewMsgMessage } from "../../types/message-types";
import { ParsedMyNewMsg } from "../../types/tl-types";

export class MyNewMsgParser implements IMessageParser<MyNewMsgMessage> {
  public async parse(
    message: ClOutgoingMessage,
    definitionReader: DefinitionReader
  ): Promise<ParsedMyNewMsg | null> {
    const payload = message.payload as MyNewMsgMessage;
    const { d } = payload;
    // transformar d[] a campos con nombre
    const [field1, field2] = d;
    return { myNewMsg: { field1, field2 } };
  }
}
```

---

### Paso 6 — Registrar el parser en el barrel y en `Parser.ts`
**Archivo:** `src/micros/driver/translator.perte/tl-lib/parsers/implementations/index.ts`

```ts
export { MyNewMsgParser } from "./MyNewMsgParser";
```

**Archivo:** `src/micros/driver/translator.perte/tl-lib/parsers/Parser.ts`

```ts
import { MyNewMsgParser } from "./implementations";
// ...
private registerParsers() {
  this.parsers = new Map([
    // ...existentes...
    [MessageType.MY_NEW_MSG, new MyNewMsgParser()],  // ← añadir
  ]);
}
```

---

### Paso 7 — Registrar el schema en `InputValidator.ts`
**Archivo:** `src/micros/driver/translator.perte/tl-lib/services/InputValidator.ts`

```ts
// 1. Import
import { MyNewMsgMessageSchema } from "../types/message-types";

// 2. En initializeSchemaMap():
map.set(MessageType.MY_NEW_MSG, MyNewMsgMessageSchema);
```

---

### Paso 8 — Lógica especial en `microservice.ts` (solo si necesaria)
**Archivo:** `src/micros/driver/translator.perte/microservice.ts`

La mayoría de mensajes pasan por el flujo genérico (`processMessage` → `publishParsedData`) y **no necesitan tocar** el microservice. Solo añade lógica aquí si el mensaje requiere un tratamiento especial, como `CMD_ACK` (confirma un OutputMessage) o `PARAM_ACK` (confirma un ParamMessage).

Si el mensaje es "normal" (sync, sample, status, net_sync…), el flujo genérico ya lo maneja solo con el topic del `TopicMap`.

---

## Resumen visual de los 8 ficheros

| # | Archivo | Qué defines |
|---|---|---|
| 1 | `globals-message.type.ts` | `MessageType.MY_NEW_MSG = "my_new_msg"` |
| 2 | `my-new-msg-message.type.ts` *(nuevo)* | Interfaz + JSON Schema del payload raw |
| 3 | `tl-lib/types/message-types.ts` | Re-export del tipo y schema |
| 4 | `tl-lib/types/tl-types.ts` | `ParsedMyNewMsg` + entrada en `TopicMap` |
| 5 | `parsers/implementations/MyNewMsgParser.ts` *(nuevo)* | Transformación raw → parsed |
| 6 | `parsers/implementations/index.ts` + `Parser.ts` | Registro del parser |
| 7 | `services/InputValidator.ts` | Registro del schema |
| 8 | `microservice.ts` | Solo si hay lógica especial post-parse |


