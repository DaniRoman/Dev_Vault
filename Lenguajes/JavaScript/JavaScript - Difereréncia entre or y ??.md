
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
