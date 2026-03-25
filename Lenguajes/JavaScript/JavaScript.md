
### Casos para recibir `null, undefined o valores vacios`
[[JavaScript]]

### Fallback con nullish coalescing

```js
const fileId = fileRef?._id ?? fileRef;
```

`??` devuelve el operando derecho solo cuando el izquierdo es `null` o `undefined`, no cuando es `false`, `0` o `""`.

#### Diferencia entre `||` y `??`
***Operador*** `||`

Devuelve el valor de la derecha cuando el de la izquierda es **falsy**.

Valores falsy

- `false`
- `0`
- `""`
- `null`
- `undefined`
- `NaN`

 Ejemplo

```ts
const result = left || right;
```

> “si el valor izquierdo no vale a nivel booleano, usa el derecho”

Casos

```ts
"" || "default"        // "default"
0 || 10                // 10
false || true          // true
null || "default"      // "default"
undefined || "default" // "default"
```

***Operador `??`***

Devuelve el valor de la derecha solo cuando el de la izquierda es **null o undefined**.
Casos nullish

- `null`
- `undefined`

Ejemplo

```ts
const result = left ?? right;
```

> “si el valor izquierdo no existe, usa el derecho”

Casos

```ts
"" ?? "default"        // ""
0 ?? 10                // 0
false ?? true          // false
null ?? "default"      // "default"
undefined ?? "default" // "default"
```

Regla rápida para diferenciarlos

`||` cuando quieras reemplazar valores “vacíos” o falsy

- trabaja con **truthy/falsy**
- sustituye si el valor izquierdo “no vale” en booleano    
- más agresivo

Ejemplos:

- texto vacío
- cero
- false
- null
- undefined
    
`??` cuando solo quieras reemplazar ausencia real de valor

- trabaja con **null/undefined**
- sustituye solo si el valor izquierdo “no existe”
- más preciso para ids, paths, configs y datos opcionales

Ejemplos:

- `null`
- `undefined`

Frase para memorizar

**`||` pregunta “¿es truthy?”**  
**`??` pregunta “¿existe?”**

Ejemplo comparativo final

```ts
const a = "" || "X";   // "X"
const b = "" ?? "X";   // ""

const c = 0 || 99;     // 99
const d = 0 ?? 99;     // 0

const e = null || "X"; // "X"
const f = null ?? "X"; // "X"
```

---


>[!tip] Doc oficial nullish coalescing
>[M mdn__](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Nullish_coalescing)
### Acceso seguro a propiedades / optional chaining

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