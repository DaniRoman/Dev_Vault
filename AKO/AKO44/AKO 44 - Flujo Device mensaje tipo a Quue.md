Sí. Piensa el flujo con **dos decisiones distintas**:

1. **Qué handler entra**: depende del topic/queue que invoca al micro.
2. **Qué parser/topic final se usa**: depende de `payload.ty`.

**Flujo de entrada normal desde driver**
Entra en [microservice.ts](</Users/daniel.roman/Library/CloudStorage/OneDrive-AKOElectromecanicaSA/Dev/ako44/akocloud-micros/src/micros/driver/translator.perte/microservice.ts:49>) por:

```ts
handleInput(msg: string)
```

Ahí pasa esto:

```txt
Rabbit/topic input
  -> TranslatorPerte.handleInput(msg)
  -> JSON.parse(msg)
  -> isValidClOutgoingMessage(parsedMsg)
  -> initializeInputValidator()
  -> inputValidator.validateMessage(parsedMsg)
  -> processMessage(parsedMsg, validationResult, ProcessMethod.INPUT)
  -> según resultado, publishParsedData(...)
```

Dentro de `validateMessage`, se decide el tipo por el campo:

```ts
payload.ty
```

Eso ocurre en [InputValidator.ts](</Users/daniel.roman/Library/CloudStorage/OneDrive-AKOElectromecanicaSA/Dev/ako44/akocloud-micros/src/micros/driver/translator.perte/tl-lib/services/InputValidator.ts:177>):

```ts
const { ty } = payload as PayloadWithType;
const schema = this.getSchemaForMessageType(ty);
```

Si el schema pasa, devuelve:

```ts
{ isValid: true, messageType: ty }
```

Luego vuelve al micro y llama a [processMessage](</Users/daniel.roman/Library/CloudStorage/OneDrive-AKOElectromecanicaSA/Dev/ako44/akocloud-micros/src/micros/driver/translator.perte/microservice.ts:200>):

```ts
this.processMessage(parsedMsg, validationResult, ProcessMethod.INPUT)
```

**Cómo acaba en InputProcessor**
En `microservice.processMessage`, antes de parsear, llama a:

```ts
this.inputProcessor.shouldProcessMessage(...)
```

Eso está en [InputProcessor.ts](</Users/daniel.roman/Library/CloudStorage/OneDrive-AKOElectromecanicaSA/Dev/ako44/akocloud-micros/src/micros/driver/translator.perte/tl-lib/services/InputProcessor.ts:144>).

Importante: solo hace lógica especial para `SYNC`:

```ts
if (messageType !== MessageType.SYNC) {
  return true;
}
```

Si es `SYNC`, mira timestamp para evitar procesar sync antiguos.

Después, como venía con `ProcessMethod.INPUT`, entra en este case:

```ts
case ProcessMethod.INPUT:
  return await this.inputProcessor.processMessage(...)
```

Y ahí ya entra en [InputProcessor.processMessage](</Users/daniel.roman/Library/CloudStorage/OneDrive-AKOElectromecanicaSA/Dev/ako44/akocloud-micros/src/micros/driver/translator.perte/tl-lib/services/InputProcessor.ts:19>).

**Selección del parser**
Dentro de `InputProcessor.processMessage`:

```txt
InputProcessor.processMessage
  -> initParser()
  -> parser.parseMessage(message, messageType)
```

Luego [Parser.ts](</Users/daniel.roman/Library/CloudStorage/OneDrive-AKOElectromecanicaSA/Dev/ako44/akocloud-micros/src/micros/driver/translator.perte/tl-lib/parsers/Parser.ts:49>) busca el parser por `messageType`:

```ts
const parser = this.parsers.get(messageType);
```

El mapa está en [Parser.ts](</Users/daniel.roman/Library/CloudStorage/OneDrive-AKOElectromecanicaSA/Dev/ako44/akocloud-micros/src/micros/driver/translator.perte/tl-lib/parsers/Parser.ts:173>):

```ts
[MessageType.SYNC, new SyncParser()],
[MessageType.COMM_SYNC, new CommSyncParser()],
[MessageType.STATUS, new StatusParser()],
[MessageType.SAMPLE, new SampleParser()],
...
```

Así que si llega:

```json
{ "ty": "sync" }
```

termina en `SyncParser`.

Si llega:

```json
{ "ty": "comm_sync" }
```

termina en `CommSyncParser`.

**Después de parsear**
Vuelve al `handleInput`.

Ahí hay 3 caminos principales:

```txt
PARAM_ACK
  -> confirmParam()
  -> updateLastMessageTimestamp()
  -> renewAndHandleRecovery()
  -> no publishParsedData

CMD_ACK
  -> confirmOutput()
  -> updateLastMessageTimestamp()
  -> renewAndHandleRecovery()
  -> publishParsedData()

Cualquier otro mensaje: SYNC, STATUS, SAMPLE, ALARM, COMM_SYNC, etc.
  -> reenvía param pendiente si existe
  -> reenvía cmd pendiente si existe
  -> updateLastMessageTimestamp()
  -> renewAndHandleRecovery()
  -> publishParsedData()
```

El topic final sale de `TopicMap`, por ejemplo:

```ts
[MessageType.SYNC, "12830.input.sync"]
[MessageType.COMM_SYNC, "12830.input.comm_sync"]
```

Resumen corto:

```txt
handleInput
  -> InputValidator decide messageType desde payload.ty
  -> microservice.processMessage(..., ProcessMethod.INPUT)
  -> InputProcessor.shouldProcessMessage
  -> InputProcessor.processMessage
  -> Parser.parseMessage
  -> parser específico según messageType
  -> vuelve al micro
  -> publishParsedData al topic correspondiente
```