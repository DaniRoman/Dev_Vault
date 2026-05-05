## Flujo de los tests

>[!warning] Pasar a limpio

Sí, ahora **para el flujo que quieres entender tengo todo lo necesario**.

Lo único que faltaría, solo si quieres entrar al detalle visual exacto del HTML, sería el fichero:

```ts
../../lib/report-template
```

porque `CsvToHtmlConverter.ts` no pinta el HTML directamente: llama a `buildHtmlReport(...)`. Pero para entender el flujo **resultados → lista → CSV → HTML**, con lo que has pasado es suficiente.

## Flujo completo

El flujo empieza en `test-error-comm-real.ts`. Ese runner selecciona el modo `local` o `production`, carga `.env` o `.env.prod`, selecciona `timingMode`, crea una instancia de `TestCaseErrorComm` y ejecuta `executeTestSteps()`.

La instancia de `TestCaseErrorComm` hereda de `BaseComTestCase`. En el constructor de la base se crea la configuración desde `config` o desde variables de entorno, y se inicializan dos listas importantes:

```ts
this.testResults = [];
this.stateTransitions = [];
```

La primera guarda los resultados de cada paso. La segunda guarda transiciones de estado del device.

La “lista de resultados” se construye cada vez que el test llama a:

```ts
this.addResult(...)
```

Ese método mete un objeto dentro de `this.testResults` con `step`, `description`, `passed`, `status` y `details`. Esa es la lista que después se convierte en `steps.csv`.

En `TestCaseErrorComm`, los pasos van llamando a `addResult`. Por ejemplo, el paso 1 valida Redis, el paso 2 valida API, y el paso 3 tiene ya la bifurcación entre `local` y `production`: en local dispara CoAP automáticamente; en production espera confirmación manual del usuario.

Al final del flujo, después de los pasos 16 y 17, se desconecta Redis, se imprime el resumen y se llama a:

```ts
this.exportReport(redisSnapshots);
```

Ese es el punto donde empieza la generación de CSV y HTML.

## Cómo se crean los CSV

`exportReport(...)` está en `BaseComTestCase`. Primero calcula datos generales: nombre del test, total de pasos, cuántos han pasado, cuántos han fallado y crea una carpeta nueva dentro de `reports`, con un timestamp en el nombre.

Luego define una función `row(...)` para convertir arrays de valores en líneas CSV, escapando comas, comillas y saltos de línea. Esto es importante porque evita que un `details` con coma rompa el CSV.

Después crea cuatro bloques de texto CSV:

```ts
summaryLines
stepsLines
redisLines
transitionLines
```

`summaryLines` contiene metadatos del test: nombre, título, fecha, total, passed, failed, modo, device, serial, redis, ventanas L2/TXE y filas extra. `stepsLines` sale directamente de `this.testResults`. `redisLines` sale de los snapshots de Redis. `transitionLines` sale de `this.stateTransitions`.

Finalmente escribe estos ficheros:

```txt
summary.csv
steps.csv
redis.csv
state-transitions.csv
```

y justo después llama a:

```ts
generateReportFromCsv(subDir)
```

para generar el HTML.

## Cómo se pasa de CSV a HTML

`CsvToHtmlConverter.ts` es el conversor. Su comentario inicial ya resume la idea: lee `summary.csv`, descubre automáticamente el resto de CSV del directorio y genera un `index.html` standalone.

Primero usa `parseCsv(...)`, que convierte el texto CSV en objetos JavaScript. La primera línea del CSV se toma como cabecera y cada línea posterior se convierte en un objeto con esas columnas. Por ejemplo, `steps.csv` pasa de texto CSV a algo parecido a:

```ts
[
  {
    Step: "1",
    Status: "PASS",
    Description: "Check Redis connection",
    Details: "PONG"
  }
]
```

Luego `generateReportFromCsv(directoryPath)` exige que exista `summary.csv`. Si no existe, lanza error. Después lee `summary.csv`, separa las claves reservadas (`Name`, `Title`, `RunAt`, `Total`, `Passed`, `Failed`) del resto de propiedades. Las reservadas se usan para cabecera/estadísticas del reporte; las otras van como propiedades del test.

Después auto-descubre todos los demás `.csv` del directorio excepto `summary.csv`, los ordena, los parsea y los convierte en tablas. El título de cada tabla sale del nombre del archivo: por ejemplo `state-transitions.csv` se transforma en `State Transitions`.

Por último llama a `buildHtmlReport(...)`, le pasa la cabecera, estadísticas, propiedades y tablas, y escribe el resultado en:

```txt
index.html
```

## En una frase

El flujo es:

```txt
executeTestSteps()
  → addResult() va llenando testResults[]
  → redisSnapshots[] y stateTransitions[] guardan datos extra
  → exportReport() transforma esas listas en CSV
  → generateReportFromCsv() lee los CSV
  → buildHtmlReport() genera el HTML
  → se guarda index.html
```

Una cosa importante: **el HTML no se genera directamente desde `testResults`**, sino desde los CSV. Primero se materializa todo en archivos `.csv`; después el conversor lee esos archivos y construye la página.
### ErroComm 

