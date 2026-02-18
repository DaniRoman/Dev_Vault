
>[!example] Recurso de Skillshare
>[Fuente de Recurso]()

Errores inesperados como una conexión fallida a una base de datos como practica recomendada hay que contar con estas situaciones inesperadas y manejarlas correctamente enviando mensajes de error adecuados al cliente y registrar la excepción en el servidor para mas adelante mirar cuales son los errores que están sucediendo con frecuencia y como manejarlos. 

![[Pasted image 20260206094216.png]]

En un contexto donde hago una llamada a mongo desde mi api con postman y mongo puede devolverme un error.

Este seria el error real visto desde la terminal de mi api, por defecto si la conexión con mongo no puede establecerse, el driver de mongo intentara reconectarse tres veces mas 

![[Pasted image 20260206094556.png]]

Pero nuestra app ya peto y en un caso real donde el cliente de mongo volviese a estar activo en 1 min con nuestra implementación actual, node no volvería a estar activo y no podría atender a ningún otro cliente de mongo

![[Pasted image 20260206095059.png]]

Entonces en el handler que se ocupara de solicitar esa llamada con una promesa o un `await` necesitamos envolverlo todo en un bloque `try` para manejar las promesas rechazadas  y agregar el bloque  `catch`  donde obtendré el error y le enviare una respuesta adecuada al cliente .

![[Pasted image 20260206100202.png]]
Con esta implementación el cliente ya estaría recibiendo el mensaje y el error que implementamos

![[Pasted image 20260206100554.png]]

y en el servidor ya no veo ese error que mongo me mandaba que resulto en la terminación del proceso en mi api apagandola
![[Pasted image 20260206100511.png]]

## Siguiente escenario 
 
 En un escenario donde hubiésemos de cambiar el mensaje que se le muestra al cliente deberia que ir por cada manejador de rutas donde tengo ese mensaje en el try catch y cambiar el mensaje de error a mano. 
 Entonces vamos a mover esta lógica para manejar los errores en algún lugar de una manera centralizada para si en el futuro quiero cambiar algo poder hacerlo solo en un lugar.

Aquí es donde se registran las funciónes de midleware en el index.ts,  y podemos definir una función especial `error middleware` al final de todos los `middlewares` , con cuatro parámetros, los cuales el primero sera el error que capturamos en algún lugar del código, y en esta función aplicamos toda la lógica para manejar los errores



![[Pasted image 20260206115553.png]]

Ahora volviendo al controlador anterior quiero pasar el control que antes hacia en el catch a mi `error middleware`  agregando un nuevo parámetro `next` a mi función para pasar el control al siguiente `middleware` en el procesamiento de requests así que paso ese parámetro `next` en el bloque de `catch` con el error como argumento.

![[Pasted image 20260206120150.png]]

ahora al registrar este tipo de funcion en index despues de todas las funciones middleware existentes, cuando llamemos next acabaremos en nuestro error middleware. y el error pasado sera el primer error de la funci´ñon ahora todos los errores estaran centralizados aqui y cualwuier cambio lo haremos aqui 
 

![[Pasted image 20260206115553.png]]

En un una implementación del codigo de errores real, este tendra muchas lineas de codigo con lo que no es lo mas limpio ponerlas todas en el index.ts, emntonces en index orquestraremos y haremos un arreglo de alto nivel y los detalles encapsulados en otro modulo `archivo.js` dentro de `middleware > error.middleware`

![[Pasted image 20260206121559.png]]

![[Pasted image 20260206121757.png]]

y finalemnte hago referencia a la funcion en el app. use,  no llamo a esta función solo hago una referencia a ella 

## Siguiente punto 

![[Pasted image 20260206100202.png]]
El problema de esta implementación es que este bloque `try catch` deberíamos implementarlo en cada `route handler` (controlador de las rutas) agregando muchísimo ruido al código dificultando la lógica real de cada ruta con lo que moveremos esta `hight handle error logic` a otro lugar para que con una sola función se pueda reutilizar en todas las rutas.

primero de todo crearemos una función reutilizable `asyncMiddleware` la cual servirá como plantilla en la cual implementaremos el  bloque `try catch` esta sera flexible y se adaptara a las diferentes rutas 

la funcion tomara una `route handler` como argumento dentro del `try` llamaremos ese `handler` y si algo sale mal, el bloque `catch` cojera ese error y lo pasara a la función `next`

lo unico que cambaira sera la lógica de cada `route handler`

![[Pasted image 20260206123354.png]]

para usar este `middleware` en la ruta la paso como argumento y quito el parámetro next también y me cargo el bloque try catch
![[Pasted image 20260206130820.png]]
importante como los `route handlers` son asíncronos tengo que usar `await` con el `handler` dentro de mi función `asyncMiddleware` 

![[Pasted image 20260206130741.png]]

al  ´handler()´ de dentro de la función también tendré que pasarle los argumentos `req, res`

>[!important] Importante
> Llamamos a `handler()` dentro la la `asyncMiddleware` y necesitamos pasarle  los parametros `req, res, next`, pero estos objetos, no están definidas en ningún punto de mi función.
> Cuando definimos una ruta `route.get("/", (req, res, next) => {})`,  
> nosotros no llamamos esta función por nosotros mismos, pasamos una referencia a esta y express se encarga de pasar los parámetros necesarios para `req, res, next` en el tiempo de ejecución 
> En nuestro código envés de llamar directamente haremos que devuelva una función que actuara como `route handler` que express puede llamar, esta función tendrá los parámetros `req, res, next`.
> Cuando express llame esta `return function` proporcionara los objetos necesarios que son `req, res, next`
> 
> ![[Pasted image 20260206131852.png]]

moveremos este también a `middleware > async.ts`
 Este wraper lo pasaremos delante de cada route.get pero de esta manera volvemos a implementar ruido todo y que se solventaron problemas y quedo mas limpio que anteriormente
  
![[Pasted image 20260208095643.png]]
Este enfoque es llamado Middleware global de errores + wrapper async propio

## Loggin Errors
En cada aplicación empresarial necesitamos registrar todas las excepciónes que se lanzan en la aplicación para poder ir al registro y poder ver las areas de nuestra app que seria conveniente mejorar, para esto utilizaremos winston, este es el código en [npm winston]() el cual copiaremos en nuestro modulo `error.middleware`

`level`: 
  El nivel de registro que tipo de log se va a registrar, este filtra los niveles por gravedad la severidad asciende por los niveles de lo mas a lo menos importante 
  error: 0 --> higest priority, warn: 1, info: 2, http: 3, verbose: 4, debug: 5, silly: 6 ---> Least priority, Establecer el nivel de esto en info, logeara de info hacia mas prioritario. y lo superior quedara bloqueado

`transport:` sistema de almacenamiento para nuestros logs por lo que define como se envían los registros
	`console:` Para logear mensajes en la consola 
	`File`, 
	`http` 
y dentro de estos puedo definir de que manera y que nivel quiero que muestren 
 
 ```js
 const winston = require('winston');

//Definición del objeto createLogger
const logger = winston.createLogger({
  level: 'info',
  
  // en formato en como aparecen los registros, texto plano, objeto...
  transports: [
    new winston.transports.File({ level: 'error', filename: 'error.log',  }),
    new winston.transports.Console({ level: 'error',format: winston.format.json(),}),
  ],
});
 ```

Existen plugins y modulos de npm para poder registrar los errores en mongo y demas sitios.
Tambien es posible estableces un formato personalizado para cada `nivel` por separado

Vamos a crear un nivel personalizado para usar colores para mapear los tipos de errores en formato simple

![[Pasted image 20260209085334.png]]

Con este logger costumizádo ya esta listo para importarse en el `error.middleware` declaro el objeto logger y a este le indico el nivel de prioridad (Este se comportara de la manera que le indique en los transportes)`logger.info() - logger.error()`

```js 
// O directamente pasarle el error del middleware
funcion (err, req, res){
logger.error({message: err.message})
}

```


>[!Warning] Reglas de oro 
>
>
>
>
>


## 2️⃣ Regla de oro por capas

### Repository (SQL)

- ❌ NO `try/catch`
    
- ❌ NO logs
    
- ✅ deja que el error suba
    

### Service

- ✅ decide _qué significa_ el fallo
    
- ❌ NO logs técnicos
    
- ✅ lanza `AppError` de negocio
    
- ✅ deja subir errores técnicos
    

### API client (axios)

- ❌ NO `return AppError`
    
- ❌ NO logs
    
- ✅ **throw**
    

### Error middleware

- ✅ **ÚNICO sitio de `logger.error`**
    
- ✅ aquí diferencias SQL vs API