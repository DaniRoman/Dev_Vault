# Cuándo puedes recibir `null` o `undefined`

Esto suele significar **ausencia real de valor**.

## `undefined`

Suele aparecer cuando:

- una propiedad **no existe**
- una variable está declarada pero **sin asignar**
- una función **no devuelve nada**
- accedes a algo opcional que no viene informado

### Ejemplos

const obj = {};  
obj.id; // undefined

let x;  
x; // undefined

function getValue() {}  
getValue(); // undefined

## `null`

Suele aparecer cuando:

- el sistema **quiere representar explícitamente que no hay valor**
- la BD guarda un campo como `null`
- una API devuelve “sin dato” de forma intencional

### Ejemplo

const user = {  
  avatar: null  
};

## Apunte corto

- `undefined` = no está definido / no vino
- `null` = sí vino, pero vacío de forma explícita

---

# 2. Cuándo puedes recibir valores “vacíos” o falsy

Aquí el valor **sí existe**, pero en contexto booleano se comporta como falso.

## Falsy más comunes

- `false`
- `0`
- `""`
- `null`
- `undefined`
- `NaN`

## Casuísticas típicas

### `""` string vacío

Cuando:

- un input de texto viene vacío
- una API devuelve una cadena vacía
- un campo existe pero sin contenido

const name = "";

### `0`

Cuando:

- el valor numérico correcto puede ser cero
- contadores, índices, cantidades, flags numéricos

const retries = 0;

### `false`

Cuando:

- es un booleano válido y negativo
- flags tipo activo/inactivo, habilitado/deshabilitado

const enabled = false;

### `NaN`

Cuando:

- una operación numérica falla
- conversión inválida a número

Number("abc"); // NaN