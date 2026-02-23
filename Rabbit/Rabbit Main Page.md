
>[!warning] Conceptos Básicos Rabbit  MQ
>[Recurso en video](https://www.youtube.com/watch?v=JsIIuvK5BG0)

**Productor (publisher) vs consumidor (consumer)**
- El productor “publica” mensajes.
- El consumidor “lee/consume” mensajes.
- No se hablan directo: siempre pasan por Rabbit.

**Exchange**
- Es como un “router” dentro de Rabbit.
- Los productores publican **a un exchange**, no a una cola.
- El exchange decide a qué colas mandar el mensaje según la _routing key_ .
- Puedes usar **exchanges distintos** según **qué quieres publicar** y **cómo quieres que se reparta** el mensaje.
	- - te permiten separar:
	- **dominios** (driver, cloud, eventos, etc.)
	- **tipos de tráfico** (comandos vs eventos vs reintentos)
	- **reglas de routing** (quién recibe qué)
	Ejemplo típico:
	- `akocloud` = “bus central”
	- `ako.services.driver` = mensajes del dominio driver
	- `akocloud-retry` / `akocloud.dead` = reintentos / mensajes muertos

**Queue**
- Es el “buzón” donde realmente se almacenan mensajes.
- Los consumidores consumen **de una queue** (no del exchange).
- Si no hay consumidor, el mensaje se queda en la queue (si está encolado ahí).

**Routing key** / **Topic key**
- Es una “etiqueta” que viaja con el mensaje (ej: `backlog.input.panel_1ry_6402`).
- El exchange la usa para decidir a qué colas enrutar.

**Binding**
- Es la “regla” que conecta un **exchange → cola** y dice:  
“esta cola quiere recibir mensajes cuya routing key cumpla X”.
- Ejemplo mental: “Todo lo que llegue a `akocloud` con `backlog.input.panel_1ry_6402` mándalo a la cola `ako.services.backlog.perte`”.
- Sin binding, aunque publiques, **no llega a ninguna cola**.

>[!Info] Caso practico con AKO 44
## Como leer GUI de Rabbit MQ

![[Pasted image 20260220104153.png]]

Mirando el exchange `ako.services.backlog`, > FROM ***akocloud*** 

**From:** `akocloud`  
**Routing key:** `backlog.12830.input` (y otras)

Eso significa que los mensajes entran a este exchange desde `akocloud` cuando llevan esas “etiquetas” (routing keys). Alguien publica en `akocloud` → y Rabbit los trae aquí si la key coincide.

Parte de abajo (TO queue)

**To:** `ako.services.backlog.mediator.perte`  
**Routing key:** `backlog.12830.input` / `backlog.12830.cancelation`

Este exchange mete esos mensajes en esa cola**.

## 3) ¿Dónde está tu micro entonces?

Tu micro **NO está en el exchange**.  
Tu micro está **leyendo la cola**:

✅ `ako.services.backlog.mediator.perte`

Así que el camino es:

**akocloud** → (entra aquí por la key) → **ako.services.backlog** → (sale) → **cola `...mediator.perte`** → **micro consume**