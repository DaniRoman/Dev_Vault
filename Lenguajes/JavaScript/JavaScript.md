
### Casos para recibir `null, undefined o valores vacios`
[[JavaScript - Casos Recibir null, undefined o valores vacios]]

### Fallback con nullish coalescing

^d7d631

```js
const fileRef: any = (response as FirmwareUpdate)?.file;
const fileId = fileRef?._id ?? fileRef;
```

`??` devuelve el operando derecho solo cuando el izquierdo es `null` o `undefined`, no cuando es `false`, `0` o `""`. ^320450

>[!tip] Doc oficial nullish coalescing
>[M mdn__](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Nullish_coalescing)

>[!example] Enlaces de interes a este caso
>[[JavaScript - Difereréncia entre or y ??]]
>[[JavaScript - Casos Recibir null, undefined o valores vacios]]

---

### Diferencia entre `||` y `??`
[[JavaScript - Difereréncia entre or y ??]]

---
### Acceso seguro a propiedades / optional chaining

[[JavaScript -  Acceso seguro a Propiedades Optional chaining]]
```js
(response as FirmwareUpdate)?.file
fileRef?._id
```

Si `response` o `fileRef` son `null` o `undefined`, **no lanza error** y devuelve `undefined`.

>[!tip] Doc oficial optional chaining
>[M mdn__](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Optional_chaining)

>[!example] `(response as FirmwareUpdate)`  Aserción de tipos
[[TypeScript - Main Site#^55df50]]

---
### Conceptos

>[!success]

- ***Strict Mode***: [Recurso](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode)
- ***ECMAScript 6***: Versión de JavaScript. [Recurso](https://talent500.com/blog/what-is-es6-javascript-guide/)

## Metodos

`JSON.parse(), JSON.stringify()`: [Recurso a difference between JSON.parse & JSON.stringify](https://www.geeksforgeeks.org/javascript/what-is-difference-between-json-parse-and-json-stringify-methods-in-javascript/)
### Reduce
[Recurso](https://developer.mozilla.org/es/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce)
### Callbacks

>[!success] Concepto

Un **callback** es una función que pasas como argumento a otra función para que esa función la ejecute después, normalmente por cada elemento o cuando ocurre algo. En el ejemplo de `find`, el callback se ejecuta con cada objeto del array y devuelve `true/false` para decidir cuál es el que se busca.

>[!example] Ejemplo

```js
function esHumedad(obj) {
  return obj.code === "humidity";
}

const humidity = populatedDevice.values.find(esHumedad);


const humidity = populatedDevice.values.find(obj => obj.code === "humidity");
```

>[!tip] ¿ Cuando usarla ?

SÍ (típico)

- **Arrays**: cuando recorres/transformas datos sin escribir bucles a mano  
    (`find`, `map`, `filter`, `forEach`, `reduce`).
- **Eventos / listeners**: cuando algo ocurre y quieres reaccionar  
    (Rabbit consumer, `on('message', cb)`, clicks, timers).
- **Funciones genéricas/reutilizables**: cuando una función hace un “proceso” y tú le pasas la acción concreta  
    (ej. `procesarDatos(datos, transformFn)`).
    
Cuándo NO (o mejor evitar)

- Cuando la lógica es **lineal y simple**: un `for` y ya.
- Cuando los callbacks se anidan mucho (**callback hell**) → mejor `async/await` o promesas.
- Cuando necesitas **control claro del flujo** (errores, retornos, secuencia) y el callback lo complica.
- 
### Spread Operator

Los tres puntos `...` son el **operador de propagación (spread)** en JavaScript/TypeScript.  
Copian **todas las propiedades de un objeto** a otro nuevo.  
Luego puedes **sobrescribir o agregar** propiedades específicas.  
En tu ejemplo, copia todo de `result` y convierte `serialNumber` e `imei` a `string`.  
Así creas un objeto igual al original pero **compatible con JSON**.

[Recurso](https://developer.mozilla.org/es/docs/Web/JavaScript/Reference/Operators/Spread_syntax)


### NaN

^eaec43

`NaN` significa **“Not a Number”**: _no es un número válido_.

### `Math.Floor`

^fb6174

- `Math.floor(...)` quita los decimales y deja un **entero**.

### Math.Random

^2b5f58

genera un número “aleatorio” (pseudoaleatorio) **decimal** entre 0 (incluido) y 1
Puede dar valores como `0.12`, `0.734`, `0.9991`…
### Date.now( )

^0655fa
- `Date.now()` devuelve el tiempo actual en **milisegundos** desde el 1/1/1970 (Unix epoch).
- Al dividir entre `1000`, lo conviertes a **segundos**.
 da el **timestamp Unix en segundos** (ej. `1700000000`), útil para guardarlo/enviarlo en formatos compactos.
 ### Unix time (lo que tú estás viendo)

Es una forma estándar de representar una fecha/hora como un número:

- **Cantidad de segundos** (o milisegundos) transcurridos desde el **1 de enero de 1970 00:00:00 UTC**.
- Ejemplo: `1700000000` (segundos) representa una fecha concreta.
- si ves valores tipo `1700000000000` → ms; si ves `1700000000` → segundos.