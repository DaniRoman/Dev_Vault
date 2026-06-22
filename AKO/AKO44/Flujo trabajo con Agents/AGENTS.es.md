# AGENTS.md

## Propósito

Este archivo define acuerdos de trabajo duraderos a nivel de repositorio para agentes de programación con IA como Codex, Claude Code o cualquier agente compatible que lea instrucciones del proyecto.

Usa este archivo para reglas estables que deban aplicarse en la mayoría de las tareas de este repositorio. No lo uses como registro de tarea, historial de chat ni bloc de notas.

## Orden de prioridad de las fuentes de verdad

Cuando trabajes en este repositorio, sigue este orden de prioridad:

1. El código real, tests, esquemas, migraciones y configuración de ejecución.
2. El documento actual de la tarea dentro de `docs/ia/`.
3. Este archivo `AGENTS.md`.
4. Convenciones existentes del proyecto detectadas en archivos cercanos.
5. La última instrucción explícita del usuario.

Si hay un conflicto, detente y explica el conflicto antes de hacer cambios destructivos.

## Principios operativos principales

- Prefiere cambios pequeños y revisables antes que grandes reescrituras.
- Conserva los contratos públicos existentes salvo que la tarea requiera explícitamente cambiarlos.
- No elimines comportamiento legacy sin documentar el reemplazo y la ruta de migración.
- No introduzcas nuevas dependencias de producción sin una razón clara.
- No hagas cambios amplios solo de formato en archivos no relacionados con la tarea.
- Evita cambios especulativos. Si falta evidencia, inspecciona el código antes de decidir.
- Trata los tests, las comprobaciones de tipos y la salida de build como evidencia de validación.
- Mantén visibles la seguridad, la compatibilidad hacia atrás y el impacto operativo.

## Higiene de contexto

El agente debe mantener el contexto ligero y útil.

Haz:
- Lee el documento de la tarea antes de empezar.
- Lee el conjunto más pequeño posible de archivos fuente relevantes para la fase.
- Resume los hallazgos relevantes antes de editar.
- Actualiza el documento de la tarea después de cada cambio de código en el mismo turno, sin pedir permiso.
- Al actualizar, vuelve a leer las secciones escritas previamente, incluidas fases anteriores, y reescribe cualquier cosa que ya no coincida con el código en disco.
- Registra decisiones, riesgos y resultados de validación.

No hagas:
- Pegar grandes bloques de código en documentos de tarea salvo que sea necesario.
- Guardar logs largos, stack traces o salidas completas de comandos en contexto persistente.
- Duplicar código fuente en Markdown.
- Mantener suposiciones obsoletas, diseños previstos o helpers propuestos en el documento cuando la implementación real ya haya divergido.
- Permitir que el documento de la tarea se desvíe del estado actual del código, ni siquiera dentro de la misma fase.
- Preguntar al usuario si debe actualizar el documento: actualízalo directamente.
- Avanzar a otra fase sin confirmación explícita del usuario.

## Flujo de trabajo para tareas por fases

Cuando una tarea esté documentada en `docs/ia/*.md`, sigue este flujo de trabajo.

### Antes de tocar código

1. Lee este archivo.
2. Lee el documento de tarea relevante dentro de `docs/ia/`.
3. Identifica la fase actual.
4. Trabaja solo en la fase solicitada explícitamente por el usuario.
5. Resume:
   - qué has entendido,
   - archivos que planeas inspeccionar o modificar,
   - riesgos conocidos,
   - suposiciones,
   - validación esperada.

Si falta el documento de tarea, créalo a partir de `docs/ia/task-template.md` antes de implementar.
Consulta `docs/ia/writing-guide.md` para saber cómo rellenar correctamente cada sección.

### Durante la investigación

- Traza el flujo de código afectado, por ejemplo controller → service → DB, o API → AMQP → translator.
- Dibuja un `flowchart` o `sequenceDiagram` de Mermaid en el documento de la tarea para hacer visible el flujo.
- Anota el diagrama con marcadores ❌ donde estén los bugs antes de escribir cualquier fix.
- Si se encuentra un bug durante la validación, abre una fase dedicada de investigación del bug antes de programar la corrección.

### Durante la implementación

- Haz el cambio seguro más pequeño que satisfaga el objetivo de la fase.
- Mantén el comportamiento compatible salvo que la fase diga explícitamente lo contrario.
- Reutiliza helpers, controllers, services, validators y patrones existentes.
- Prefiere extraer lógica compartida antes que duplicar con copy/paste.
- Mantén el manejo de errores coherente con el código alrededor.
- Mantén los logs útiles pero sin ruido.
- Si la fase revela un problema de diseño mayor, documéntalo y detente antes de ampliar el scope.
- Actualiza el diagrama de flujo en el documento de la tarea para reflejar el nuevo comportamiento.

### Después de la implementación

Actualiza el documento de la tarea con:

- estado de la fase,
- archivos modificados,
- resumen de la implementación,
- decisiones tomadas,
- problemas encontrados,
- comandos de validación ejecutados,
- resultados de tests,
- notas de verificación manual,
- riesgos restantes,
- siguiente fase recomendada.

No avances a la siguiente fase sin confirmación explícita del usuario.

## Definición de hecho

Una fase está hecha solo cuando:

- El comportamiento solicitado está implementado o el bloqueo está documentado.
- El código compila o se explica el fallo de build.
- Se han ejecutado los tests relevantes o se documenta por qué no se pudieron ejecutar.
- El documento de la tarea está actualizado.
- El siguiente paso está claro.

## Estándares de programación

### General

- Sigue el estilo existente del proyecto.
- Prefiere nombres explícitos en vez de abreviaturas.
- Prefiere funciones helper puras para lógica reutilizable de ramificación o normalización.
- Mantén las funciones cohesionadas y razonablemente pequeñas.
- Evita efectos secundarios ocultos.
- Evita cambiar comportamiento no relacionado.

### TypeScript / Node.js

- Conserva el target de TypeScript y las convenciones de módulos existentes.
- Prefiere interfaces tipadas para nuevos contratos entre módulos.
- Evita `any` salvo que encaje con código existente o con datos sin tipar.
- Valida la entrada externa en los límites controller/service.
- Mantén los flujos async legibles. Prefiere `async/await` para código nuevo salvo que el archivo alrededor use consistentemente cadenas de promesas.
- No ocultes errores silenciosamente. Devuelve errores compatibles y registra contexto accionable.
- Conserva las convenciones existentes de códigos de estado HTTP.

### Compatibilidad de API

Antes de cambiar un endpoint o la forma de una respuesta, comprueba:

- ruta,
- método HTTP,
- checks de auth/privilege,
- parámetros de query,
- body de la request,
- forma de la response,
- códigos de estado de error,
- compatibilidad con frontend,
- comportamiento de API pública frente a interna.

Si las implementaciones antigua y nueva coexisten, documenta claramente la regla de routing o branching.

### Datos y persistencia

- No cambies esquemas de base de datos, migraciones, índices ni semántica de queries sin scope explícito.
- Valida semántica de fechas, zonas horarias y rangos cuando trabajes con datos time-series.
- Evita queries sin límites.
- Documenta cualquier suposición sobre frescura de datos, retención o servicios externos.

### Observabilidad

Para cambios no triviales, conserva o añade suficiente observabilidad para depurar:

- identificadores como device ID, user ID, company ID cuando corresponda,
- rama tomada, por ejemplo dispositivo antiguo frente a dispositivo nuevo,
- fallos de servicios externos,
- comportamiento de fallback.

Evita registrar secretos, tokens, credenciales o payloads sensibles.

## Estrategia de validación

Prefiere la validación más fuerte disponible en este orden:

1. Tests automatizados existentes.
2. Build de TypeScript.
3. Tests unitarios o de integración dirigidos.
4. Comprobaciones manuales de API con entradas representativas.
5. Inspección estática si no es posible validar en runtime.

Cada fase debe documentar la validación realizada.

## Higiene de Git y revisión

- Mantén los cambios acotados a la fase activa.
- Evita mezclar refactors con cambios de comportamiento salvo que la fase requiera ambas cosas.
- Haz commits o grupos de cambios fáciles de revisar.
- Documenta por qué se hizo un cambio, no solo qué cambió.
- Deja el repositorio en un estado donde otro desarrollador pueda continuar desde el documento de la tarea.

## Prompt recomendado para trabajo por fases

Usa este patrón de prompt cuando pidas a un agente que continúe:

```text
Lee AGENTS.md y docs/ia/<task-file>.md.
Trabaja solo en la Fase <N>: <nombre de la fase>.
Antes de editar, resume qué has entendido, archivos a inspeccionar y riesgos.
Después de implementar, actualiza docs/ia/<task-file>.md con estado de la fase, decisiones, validación y siguiente paso.
No avances a la siguiente fase sin confirmación.
```
