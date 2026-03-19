
>[!warning] Diferencia Simple 
>`cmd` es cuando cambio de parámetro `params` solo se hace desde un `endpoint`



- **`PARAM`** = “cómo quiero que quede configurado el equipo” no le estás diciendo “haz algo ahora”, sino:
	- guarda esta configuración
	- a partir de ahora trabaja con este valor
	- cambia la **configuración**
	- Cambiar el SetPoint que el equipo Perte trabaje a **4 ºC** en vez de 6 ºC.
	- - “tu setpoint ahora será 4”
	- “usa esta plantilla/configuración”
	- “trabaja con estos parámetros”
- **`CMD`** = “qué acción quiero que haga ahora” no estás cambiando la configuración base del equipo, sino diciendo:
	- “haz esto ahora, forzar encendido ahora”
	- lanzar una acción manual
	- ejecutar una orden concreta
	- disparar una función del dispositivo ahora mismo
	- dispara una **acción puntual
	- - “ejecuta esta orden”
	- “haz esta maniobra”
	- “lanza esta acción ahora”


Con ese documento [[AKO 44 Main Site#^91b034]] , para saber qué estoy mandando miro **dos cosas**:

1. **el campo `ty`**
2. **el contenido de `d`**

---

 Si `ty = "param"`

- formato específico para parámetros
- lleva `attr`
- lleva `d` como lista plana `registro, valor, registro, valor...`

Ejemplo conceptual:
- “quiero cambiar varios parámetros del equipo”
- “quiero mandar una parametrización”

---

### Si `ty = "cmd"`
Estás mandando un **mensaje CMD**.

Pero aquí viene la trampa importante:

Un `cmd` puede ser:
- un **comando real**
- **o** un **cambio de parámetro** codificado como `cmd`

Eso lo dice tu propio protocolo:

> si `<cmd> > 100`, entonces no es un comando “normal”, sino la **dirección MODBUS del parámetro a cambiar**

---

## Regla 2: si `ty = "cmd"`, mira el valor de `<cmd>`

### Caso A: `<cmd> < 100`
Entonces es un **comando real**

Ejemplo:
- arrancar una acción
- ejecutar una orden puntual
- lanzar una maniobra

O sea:
- acción inmediata
- no necesariamente cambio de configuración persistente

---

### Caso B: `<cmd> > 100`
Entonces realmente estás mandando un **cambio de parámetro**, pero **empaquetado como `cmd`**

O sea:
- en el JSON pone `ty: "cmd"`
- pero funcionalmente estás cambiando un parámetro MODBUS

Esto es justo lo que hace especial vuestro protocolo.

---

## Entonces, ¿cuándo estoy mandando `param` y cuándo `cmd`?

### Estás mandando `param` cuando:
- el JSON sale con `ty: "param"`
- lleva `attr`
- el campo `d` es plano: `[reg1, val1, reg2, val2, ...]`
- normalmente lo usas para **grupo de parámetros**

### Estás mandando `cmd` cuando:
- el JSON sale con `ty: "cmd"`
- `d` contiene comandos/órdenes
- y dentro de `cmd`, si el código es mayor que 100, realmente representa **una escritura de parámetro**

---

## Ejemplos claros con Perte

### 1) `CMD` como comando real
Imagina que en la especificación del Perte el comando `12` significa una acción puntual.

Entonces algo así sería **CMD real**:

````json mode=EXCERPT
{
  "id": [123456],
  "ty": "cmd",
  "bid": 0,
  "d": [[12, null]]
}
````

Cómo leerlo:
- `ty = cmd`
- `12 < 100`
- por tanto: **comando**, no parámetro

---

### 2) `CMD` usado para cambiar un parámetro
Ahora imagina que el registro MODBUS `215` es el setpoint y quieres poner valor `4`.

Entonces esto sigue siendo **JSON cmd**, pero funcionalmente es **un cambio de parámetro**:

````json mode=EXCERPT
{
  "id": [123456],
  "ty": "cmd",
  "bid": 0,
  "d": [[215, 4]]
}
````

Cómo leerlo:
- `ty = cmd`
- `215 > 100`
- por tanto: **no es un comando puro**
- es **cambio de parámetro vía cmd**

---

### 3) `PARAM` como grupo de parámetros
Si quieres mandar varios registros/configuración en bloque:

````json mode=EXCERPT
{
  "id": [123456],
  "ty": "param",
  "attr": [0, 1],
  "d": [215, 4, 216, 1, 220, 30]
}
````

Cómo leerlo:
- `ty = param`
- lleva `attr`
- `d` es plano
- esto es **parametrización agrupada**

---

## Traducción práctica a lenguaje de negocio

### `cmd`
“haz esto”

Pero en vuestro protocolo también puede significar:
“escribe este registro MODBUS”

### `param`
“aplica este bloque de parametrización”

---

## En vuestro translator actual

En el código de `translator.perte`:

### Si entra por `handleOutputParam(...)`
sale como:
- `ty: "param"`

### Si entra por `handleOutputCommand(...)`
sale como:
- `ty: "cmd"`

### Pero además
en el parser de `CMD`, si el valor mapeado tiene dirección MODBUS `> 100`, el sistema lo trata como:
- **parameter change dentro de cmd**

O sea, en vuestro proyecto actual puede pasar esto:

- **semánticamente**: estás cambiando un parámetro
- **en el wire/protocolo**: lo estás mandando como `cmd`

---

## La mejor regla mental

Hazte esta pregunta:

### ¿El JSON final lleva `ty: "param"`?
- Sí → estás mandando **PARAM**

### ¿El JSON final lleva `ty: "cmd"`?
- Sí → estás mandando **CMD**
- y luego miras:
  - `<cmd> < 100` → comando real
  - `<cmd> > 100` → cambio de parámetro codificado como cmd

---

## Lo más importante de todo

En Perte:

- **`param`** = mensaje de parametrización explícito
- **`cmd`** = mensaje genérico de orden
- pero **`cmd` también puede transportar cambios de parámetros** si el código es una dirección MODBUS mayor que 100

Por eso a veces parece que `cmd` y `param` se pisan:  
**no se pisan, pero el protocolo permite que un parámetro viaje por los dos caminos**.

## Si quieres, en el siguiente mensaje te hago una tabla muy concreta:

- **“si veo este JSON, qué está mandando”**
- con 4 o 5 ejemplos reales de Perte, uno por uno.



---


## Tabla rápida: cómo saber qué estás mandando en Perte

### Caso 1
````json mode=EXCERPT
{
  "id": [123456],
  "ty": "param",
  "attr": [0, 1],
  "d": [215, 4, 216, 1]
}
````

- `ty = "param"`
- estás mandando **PARAM**
- es un **bloque de parametrización**
- cambia registros MODBUS en grupo

---

### Caso 2
````json mode=EXCERPT
{
  "id": [123456],
  "ty": "cmd",
  "bid": 0,
  "d": [[12, null]]
}
````

- `ty = "cmd"`
- `12 < 100`
- estás mandando **CMD real**
- es una **acción puntual** del equipo

---

### Caso 3
````json mode=EXCERPT
{
  "id": [123456],
  "ty": "cmd",
  "bid": 0,
  "d": [[215, 4]]
}
````

- `ty = "cmd"`
- `215 > 100`
- formalmente es **CMD**
- pero funcionalmente es un **cambio de parámetro vía cmd**

---

### Caso 4
````json mode=EXCERPT
{
  "id": [123456],
  "ty": "param",
  "attr": [1, 1],
  "d": [300, 10, 301, 20]
}
````

- sigue siendo **PARAM**
- `attr[0] = 1` indica `default parameters = yes`
- se usa para una parametrización más “global” / inicial

---

## Regla definitiva

### Mira primero `ty`

- `ty = "param"` → **PARAM**
- `ty = "cmd"` → **CMD**

### Si es `cmd`, mira luego el primer valor de `d`

- `<cmd> < 100` → comando real
- `<cmd> > 100` → cambio de parámetro enviado como cmd

---

## Aplicado a vuestro código

En `src/micros/driver/translator.perte/microservice.ts`:

- `handleOutputParam(...)` → genera `ty: "param"`
- `handleOutputCommand(...)` → genera `ty: "cmd"`

Entonces:

- si el flujo entra por `handleOutputParam` → estás mandando `PARAM`
- si entra por `handleOutputCommand` → estás mandando `CMD`

## Truco práctico

Si quieres saberlo rápido en logs, busca el `toCLMessage.payload.ty`:

- `"param"` → param
- `"cmd"` → cmd

Si quieres, te puedo enseñar **en ese `microservice.ts` exactamente en qué líneas sale cada uno**.
