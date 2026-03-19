
## Diferencia simple

Piensa así:

- **`PARAM`** = “cómo quiero que quede configurado el equipo”
- **`CMD`** = “qué acción quiero que haga ahora”

La diferencia más importante es:

- **`PARAM`** cambia la **configuración**
- **`CMD`** dispara una **acción puntual**

---

## Ejemplo fácil con un dispositivo Perte

### Caso 1: cambiar el setpoint
Quieres que el equipo Perte trabaje a **4 ºC** en vez de 6 ºC.

Eso es un **`PARAM`**.

¿Por qué?
Porque no le estás diciendo “haz algo ahora”, sino:
- guarda esta configuración
- a partir de ahora trabaja con este valor

En vuestro código esto encaja con `ParamParser`, porque ahí existen cosas como:
- `setPointValue`
- `configurations`
- `defaultParameters`
- `isFirstActivation`

O sea, `PARAM` se usa para mandar configuración del equipo.

---

### Caso 2: ejecutar una acción puntual
Ahora imagina que al Perte le dices algo como:
- lanzar una acción manual
- ejecutar una orden concreta
- disparar una función del dispositivo ahora mismo

Eso es un **`CMD`**.

¿Por qué?
Porque no estás cambiando la configuración base del equipo, sino diciendo:
- “haz esto ahora”

En el código, eso entra como:
- `cmd_ref`
- `cmd_value`

y lo procesa `CMDParser`.

---

## Una forma de recordarlo

### `PARAM`
Es como decirle al Perte:

- “tu setpoint ahora será 4”
- “usa esta plantilla/configuración”
- “trabaja con estos parámetros”

Es una **forma de operar**.

### `CMD`
Es como decirle al Perte:

- “ejecuta esta orden”
- “haz esta maniobra”
- “lanza esta acción ahora”

Es una **orden puntual**.

---

## Ejemplo comparando ambos

### `PARAM`
“Cambia el setpoint del Perte a 4 ºC”

Resultado esperado:
- el valor configurado del equipo cambia
- ese valor queda como nueva referencia de funcionamiento

### `CMD`
“Ejecuta ahora una acción X del Perte”

Resultado esperado:
- el equipo hace una acción concreta
- no necesariamente cambia su configuración permanente

---

## En vuestro código, qué diferencia práctica tienen

### `PARAM`
Pasa por:
- `handleOutputParam(...)`
- `processMessage(... MessageType.PARAM ...)`
- `ParamParser`
- `ParamMsgService`

Y después:
- si ya había un `PENDING`, compara el mensaje completo
- si es igual, no lo vuelve a guardar
- si es distinto, lo reemplaza

### `CMD`
Pasa por:
- `handleOutputCommand(...)`
- `processMessage(... MessageType.CMD ...)`
- `CMDParser`
- `OutputMsgService`

Y después:
- si ya había un `PENDING`, fusiona `payload.d`
- por eso puede acumular varias órdenes

---

## La diferencia “de negocio”

La forma más útil de verlo es esta:

### Un `PARAM`
dice:
> “quiero que el dispositivo quede así configurado”

### Un `CMD`
dice:
> “quiero que el dispositivo haga esto”

---

## Regla mental muy buena

Hazte esta pregunta:

### ¿Estoy cambiando una configuración estable?
- Sí → **`PARAM`**

### ¿Estoy pidiendo una acción puntual?
- Sí → **`CMD`**

---

## Ejemplo todavía más intuitivo

### Termostato de casa

- **`PARAM`**: cambiar la temperatura objetivo de 21 a 19
- **`CMD`**: pulsar “forzar encendido ahora”

El primero cambia la configuración.  
El segundo ejecuta una acción.

Con Perte pasa lo mismo.

---

## En vuestro translator

Ahora mismo la separación está bastante clara:

- `PARAM` usa campos de configuración como `setPointValue` y `configurations`
- `CMD` usa campos de orden como `cmd_ref` y `cmd_value`

Así que, resumiendo:

- **`PARAM` en Perte** = cambio de parámetros/configuración
- **`CMD` en Perte** = orden puntual al equipo

## Si quieres

Puedo darte ahora **un ejemplo realista de JSON de entrada** de:
- un `PARAM` para cambiar setpoint
- y un `CMD` para lanzar una orden

y te digo exactamente por qué uno cae en `handleOutputParam` y el otro en `handleOutputCommand`.
