## Type & Interface

`interface` y `type` son dos formas de definir tipos en TypeScript.

- **`interface`** para describir la forma de objetos o contratos de clases.
- **`type`** cuando necesites uniones, tuplas, alias de tipos primitivos o combinaciones más avanzadas.

```ts
interface User {
  name: string;
  id: number;
}
```

```ts
//Composicion de tipos
type Status = "open" | "closed";
```

### Partialas Type

`Partial<T>` es un tipo utilitario de TypeScript.  
Convierte todas las propiedades de un tipo `T` en opcionales.  
Sirve sobre todo para `update`, porque permite enviar solo los campos que cambian.  
Ejemplo: `Partial<Device>` permite pasar `{ active: false }` sin tener que mandar todo el objeto.  
No añade propiedades nuevas: solo admite campos que ya existen en el tipo original.  
Por eso operadores como `$set` dan error si el parámetro espera `Partial<Modelo>`.

```ts
type Device = {
  name: string;
  active: boolean;
  serialNumber: string;
};
Partial<Device>

type PartialDevice = {
  name?: string;
  active?: boolean;
  serialNumber?: string;
};
```
## Type assertions

^e3193a

[TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#type-assertions) 

- **`as` es una aserción de tipo**: indica a TypeScript cómo tratar una variable. (TypeScript Handbook) ^55df50
- **No valida en runtime**: no transforma ni comprueba el dato real. (TypeScript Handbook)
- **Uso en el ejemplo**: `(response as FirmwareUpdate)?.file` permite acceder a `file` como si `response` fuera `FirmwareUpdate`.
- **Riesgo**: si `response` no cumple esa estructura, la aserción no te protege; solo calla al compilador. (TypeScript Handbook)

## Extracción segura de id

## CheatSheet

[Recurso](https://devhints.io/typescript)

## Curso de TY

>[Fuente del recurso](https://fullstackopen.com/es/part9/primeros_pasos_con_type_script#la-sintaxis-alternativa-para-arrays)

## Instalar entorno en Express

> [Fuente del recurso](https://blog.logrocket.com/express-typescript-node/)

## Interfaces & Abstract classes

>[!example] Recurso
>[Fuente del recurso](https://www.typescripttutorial.net/typescript-tutorial/interfaces-vs-abstract-classes/)


## Optional Property Class

Las propiedades opcionales también se pueden crear declarándolas con un signo de interrogación justo después del nombre. Este signo indica al compilador de TypeScript que la propiedad declarada es opcional.
[Recurso](https://www.geeksforgeeks.org/typescript/optional-property-class-in-typescript/)

## Record Type

^b87f0d

Diccionario** (un objeto) para guardar pares **clave → valor**.

const strategies: Partial<Record<string, number>> = {};

- `Record<string, number>` significa: “un objeto donde **las claves son texto** (`string`) y **los valores son números** (`number`)”.
    
    - Ejemplo: `{ "a": 1, "b": 2 }`
        
- `Partial<...>` significa: “ese objeto puede estar **incompleto** (puede no tener algunas claves)”.
    
    - O sea, si preguntas por una clave que no existe, te dará `undefined`.
        
- `= {}` significa: “lo creo **vacío** al principio”.
[Recurso](https://mrando-via.medium.com/el-tipo-record-en-typescript-c40a4c3bbcdc)


## Convención de nombres 

^36fb64

Se usa el prefijo `_` como **señal visual** de que el método o variable es **interna** a la clase (helper), no un “handler” o parte de la interfaz pública del micro. Ayuda a distinguir rápido entre **puntos de entrada** (`handle...`) y **lógica reutilizable** (`_...`). No cambia el comportamiento por sí mismo: es solo una **convención de estilo**, la privacidad real la dan `private/protected/public`.

## Opciones del compilador

>[!error] Error
>Utilizar [esta guia](https://www.typescriptlang.org/docs/handbook/modules/theory.html) para solventar errores de import export path desconocidos (Desde Theory hasta compiler options) (en `packaje.json` quitar el type: y en en ficher `tsconfig type: commonJs`)

## Instalar versiones

```sh
npm install -g typescript@4.7.3
# -g → instalación global, disponible en toda la máquina.@4.7.3 → versión específica que quieres.

Comprobar versión instalada

tsc -v

npm install typescript@4.7.3 --save-dev

```


