

> Es un microservicio que actúa como intermediario entre dispositivos IoT y la base de datos. Recibe mensajes por COAP, valida el dispositivo, procesa la información y responde. Además, puede enviar comandos o actualizaciones a los dispositivos.

startServer() → Encender el servidor

Este método:

1. Inicializa la autenticación (conexión DB + lógica).
2. Comprueba si el servidor está habilitado.
3. Crea el servidor COAP.
4. Escucha mensajes.
5. Se queda esperando peticiones.

La parte más importante:

```ts
this._coapServer.on("request", (request, response) =>
  this._onMessage(request, response),
);
```

TODO mensaje que envía un dispositivo entra por aquí.

> Este método enciende el servidor y lo pone a escuchar mensajes de dispositivos en un puerto.

---

### InitializeAuthentication() → Conexión con base de datos

Aquí se construye toda la parte de datos:

1. Obtiene modelos de Mongo.
2. Crea un adapter (capa intermedia).
3. Conecta a la base de datos.
4. Crea repositorios.
5. Crea el servicio `Authentication`.

La clase `Authentication` es CLAVE porque:

- Valida dispositivos.
- Procesa mensajes.
- Genera respuestas.

> Aquí se prepara todo lo necesario para saber si un dispositivo es válido y qué hacer con sus mensajes.

---

## onMessage() → El corazón del microservicio

Este es el CORE.

Cada vez que un dispositivo envía algo:

- Comprueba que RabbitMQ esté activo.
	Si no → 503 (error).

- Extrae datos importantes:

```ts
serialNumber
requestAction (GET/POST)
ip
payload
```

- Construye un objeto interno estándar:

```ts
messageParseCl
```

Esto normaliza el mensaje para que el sistema lo entienda.

- Valida el mensaje:

```ts
_checkDeviceAndMessage(...)
```

Esto usa la clase `Authentication`.

- Responde:

	- ✅ Si todo bien → 200
    
	- ❌ Si algo falla → 405
    

> Cuando llega un mensaje, el sistema identifica el dispositivo, valida que exista y que el mensaje sea correcto, y luego responde OK o error.

---

## launchCoapRequestToDevice() → Enviar mensajes a dispositivos

Aquí el microservicio actúa como cliente COAP.

- Enviar comandos
- Enviar actualizaciones
- Enviar firmware
- Enviar respuestas activas

1. Crea una request COAP.
2. La envía al dispositivo.
3. Si todo va bien → log de éxito.
4. Si falla → log de error.

> No solo escucha dispositivos, también puede hablarles.

---

##  RESUMEN GLOBAL 

Este microservicio:

1. Se conecta a base de datos.
2. Escucha dispositivos por COAP.
3. Cuando un dispositivo envía algo:
    - Extrae su serial
    - Lo valida
    - Procesa el mensaje
    - Responde OK o error
4. También puede enviar mensajes de vuelta.
    





