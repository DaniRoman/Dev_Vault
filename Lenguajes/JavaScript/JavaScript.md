## Reduce

>[!important] Pasar a limpio
[Recurso](https://developer.mozilla.org/es/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce)

## Strict Mode

>[!important] Pasar a limpio
[Recurso](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode)

## ECMAScript 6

>[!note] Nota
Versión de JavaScript. Concretamente, es la sexta versión del estándar ECMAScript que define cómo funciona JavaScript. 
Aunque a menudo se utilizan indistintamente, estas son las diferencias técnicas entre ambos:
ECMAScript (ES): el estándar oficial y el documento de referencia (especificaciones) del lenguaje.
JavaScript (JS): el lenguaje de programación real que implementa esos estándares.
ES6 (ECMAScript 2015): una importante actualización lanzada en 2015 que introdujo características «modernas» como las funciones de flecha, let/const y clases. 
En 2026, las características de ES6 se consideran el estándar del sector y a menudo se denominan «JavaScript moderno»

[Recurso](https://talent500.com/blog/what-is-es6-javascript-guide/)

## Spread Operator

Los tres puntos `...` son el **operador de propagación (spread)** en JavaScript/TypeScript.  
Copian **todas las propiedades de un objeto** a otro nuevo.  
Luego puedes **sobrescribir o agregar** propiedades específicas.  
En tu ejemplo, copia todo de `result` y convierte `serialNumber` e `imei` a `string`.  
Así creas un objeto igual al original pero **compatible con JSON**.

[Recurso](https://developer.mozilla.org/es/docs/Web/JavaScript/Reference/Operators/Spread_syntax)


## NaN

^eaec43

`NaN` significa **“Not a Number”**: _no es un número válido_.

## `Math.Floor`

^fb6174

- `Math.floor(...)` quita los decimales y deja un **entero**.

## Math.Random

^2b5f58

genera un número “aleatorio” (pseudoaleatorio) **decimal** entre 0 (incluido) y 1
Puede dar valores como `0.12`, `0.734`, `0.9991`…
## Date.now()

^0655fa
- `Date.now()` devuelve el tiempo actual en **milisegundos** desde el 1/1/1970 (Unix epoch).
- Al dividir entre `1000`, lo conviertes a **segundos**.
 da el **timestamp Unix en segundos** (ej. `1700000000`), útil para guardarlo/enviarlo en formatos compactos.
 ### Unix time (lo que tú estás viendo)

Es una forma estándar de representar una fecha/hora como un número:

- **Cantidad de segundos** (o milisegundos) transcurridos desde el **1 de enero de 1970 00:00:00 UTC**.
- Ejemplo: `1700000000` (segundos) representa una fecha concreta.
- si ves valores tipo `1700000000000` → ms; si ves `1700000000` → segundos.