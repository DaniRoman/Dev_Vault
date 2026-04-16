
## Manejo HTTP de la ruta/listado con cache condicional

En el navegador antes de devolverte el contenido otra vez tras una llamada a alguna api, el servidor y el navegador negocian si **la respuesta ha cambiado o no**.

El flujo típico es:
1. Haces un `GET /api/firmware-update-log`
2. El servidor responde una vez con los datos y un identificador de versión, normalmente un **ETag**
3. En la siguiente petición, el navegador manda:
	1. If-None-Match: W/"..."
		1. Oye servidor, yo ya tengo una copia de este recurso. Si sigue igual, no me lo mandes otra vez.”
		
Entonces el servidor compara ese valor y, si no ha cambiado, responde: ***`304 Not Modified`***, Eso significa:

> “Sigue igual. Usa la copia que ya tienes en caché.”

## Event-Driven Architecture

>[!Example] Recurso 
>[Event-Driven Architecture of NodeJs](https://www.geeksforgeeks.org/node-js/explain-the-event-driven-architecture-of-node-js/)
## Patron de comportamiento Strategy

^81cb34

[Recurso](https://refactoring.guru/es/design-patterns/strategy)
## DTO 
[Recurso](https://www.youtube.com/watch?v=g8uRUTV1hZk)

## API Key

[[Arquitectura y diseño - API Key]]
