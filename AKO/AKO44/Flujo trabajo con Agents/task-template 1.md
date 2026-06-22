# Task: [TICKET-ID] — [Short title]

## ⚠️ Reglas de estructura y claridad del documento (NO negociable)

> El documento debe poder leerlo **alguien que no conoce la tarea** y entenderla — y poder explicarla —
> **sin revisar el código**. Para lograrlo, respeta estas reglas:
>
> 1. **Orden obligatorio del relato** (no saltar pasos ni adelantar la solución):
>    1. Objetivo → 2. Scope → 3. Glosario → 4. Contexto funcional → 5. Flujo **ANTES** →
>    6. Problema/limitación del flujo anterior → 7. Implementación **paso a paso** →
>    8. Flujo **DESPUÉS** → 9. Resultado y validación → Apéndices.
> 2. **No empezar por la arquitectura final.** Antes hay que explicar cómo funcionaba el sistema **antes**
>    de la tarea y qué problema tenía.
> 3. **Incluir un Glosario** y/o aclaraciones breves: no dar por entendido ningún término interno
>    (micro, handler, sample, serie, umbral, job, aggregate, activación, nombres de colas/colecciones…).
>    Si se dice "el micro guarda la serie", debe quedar claro **qué micro**, **qué es la serie** y **qué se guarda**.
> 4. **Incluir una sección de Contexto funcional** que explique para qué sirve el microservicio afectado.
> 5. **Diferenciar siempre flujo ANTES y flujo DESPUÉS**, con un diagrama por cada uno cuando aplique.
> 6. **Explicar la implementación por pasos** (qué se modificó, qué se añadió, qué dejó de depender de
>    otros procesos, qué bug se corrigió) **antes** de mostrar el flujo final.
> 7. **El Resultado, la validación end-to-end y los bugs corregidos van al FINAL**, nunca en el objetivo
>    ni al principio.
> 8. **Diagrama Mermaid intuitivo y autoexplicativo** siempre que la tarea afecte a un flujo de datos,
>    microservicios, handlers, jobs o procesos de negocio (ver sección "Flujo final del microservicio").

## Metadata

| Field  | Value                              |
|--------|------------------------------------|
| Ticket | [TICKET-ID]                        |
| Branch | `feature/[TICKET-ID]`              |
| Author | [name]                             |
| Created | [YYYY-MM-DD]                      |
| Status | `planned` / `in-progress` / `done` |

## Context and motivation

Why does this task exist? What problem does it solve?

## Scope

**In scope:**
- …

**Out of scope:**
- …

## ⚠️ Integridad de datos — fuente de verdad (NO negociable)

> Esta regla aplica a TODAS las tareas. Léela antes de tocar datos.

- **Nunca añadir, inventar ni parchear datos a mano** (Mongo, SQL, ficheros de config) para
  "hacer que funcione" una prueba. Un parche manual tapa el problema real y produce validaciones falsas.
- **Fuente de verdad de la configuración de un dispositivo:**
  1. La `devicedefinition` (colección `devicedefinitions`) — define `alarms.cloud`, `conf`, `analogInputs`, etc.
  2. El **esquema/documento real del device** en Mongo (colección `devices`) y cómo se puebla desde la definición.
  3. El código que propaga definición → device (akocloud-api / micros), no una inserción manual.
- Si en una prueba **falta un dato** (p.ej. `device.alarms.cloud` ausente), **NO se inyecta a mano**:
  primero hay que entender **qué mecanismo real lo debería poblar** y por qué no lo hizo
  (¿propagación de la definición? ¿endpoint de akocloud-api? ¿paso de provisión?). Ese hueco ES un hallazgo
  de la investigación, no un obstáculo a saltar.
- Si para una prueba local hace falta sembrar datos, debe hacerse **reproduciendo el mecanismo real**
  (mismo origen, mismo shape, idealmente el mismo código/endpoint), documentado y claramente marcado como
  "siembra de prueba", nunca como dato canónico.
- Antes de comparar nombres/valores, **verificar contra la definición y el esquema reales**, no contra
  suposiciones ni contra un documento de prueba editado a mano.

## Glosario (leer antes que nada)

> Lista de términos internos que aparecen en este documento, con una mini-descripción. No des por sabido
> nada: micro, handler, sample, serie temporal, umbral, job, aggregate, activación/desactivación, nombres
> de colas y colecciones, identificadores… Rellena solo los que uses.

| Término | Qué significa **en esta tarea** |
|---------|----------------------------------|
|         |                                  |

## Contexto funcional del microservicio

> ¿Qué microservicio se ve afectado y para qué sirve? ¿Qué hace normalmente con la entrada que recibe?
> Suficiente para que alguien externo entienda el resto del documento.

## Flujo ANTES de la tarea

> Cómo se comportaba el sistema **antes** de tus cambios. Incluye un diagrama Mermaid del flujo previo.

```mermaid
flowchart TD
    A["Entrada"] --> B["Microservicio"] --> C[("Almacenamiento / cola")]
```

## Problema / limitación del flujo anterior

> Qué fallaba o no cubría el flujo de arriba. Este es el motivo de la tarea.

## Phases

| # | Phase name     | Status    |
|---|----------------|-----------|
| 1 | Investigation  | `pending` |
| 2 | Implementation | `pending` |
| 3 | Validation     | `pending` |

> Add or remove phases as needed. Never advance without explicit confirmation.

---

## Phase 1: Investigation

**Status:** `pending`

### Code flow

> Trace the affected path. Annotate with ❌ where bugs sit.

```mermaid
flowchart TD
    A["POST /api/endpoint"] --> B["Controller.method()"]
    B --> C["Service.method()"]
    C --> D[("MongoDB")]
```

### Findings

What did you find? Be specific: file, line, what it does vs what it should do.

### Decisions and risks

| Decision | Reason | Risk |
|----------|--------|------|
|          |        |      |

---

## Phase 2: Implementation

**Status:** `pending`

### Files changed

| File | Change summary |
|------|---------------|
|      |               |

### Decisions made

### Problems found

<!--
> If the flow changed significantly, update the diagram here.
> Otherwise a note in "Decisions made" is enough.
-->

---

## Phase 3: Validation

**Status:** `pending`

### Steps

| # | Action | Result | Note |
|---|--------|--------|------|
| 1 | `npx tsc --noEmit` | ✅ / ❌ | |
| 2 | … | | |

### Remaining risks

---

## Flujo final del microservicio (Mermaid) — OBLIGATORIO

> **Toda tarea debe terminar con un diagrama Mermaid claro, actualizado e intuitivo** que explique
> el comportamiento del microservicio. Requisitos:
>
> - Debe poder entenderse **sin leer el código**: alguien externo debe poder explicárselo a otra persona.
> - **Pedagógico, no solo correcto.** Etiqueta cada nodo con *qué hace* en lenguaje claro (no solo el
>   nombre de la clase/columna). Incluye una **leyenda** (qué representa cada micro, qué color/emoji marca
>   lo nuevo vs. lo de siempre) y agrupa por microservicio con `subgraph`.
> - Deja explícito en el diagrama: **qué datos entran**, **qué decisiones se toman** (bifurcaciones y qué
>   rama toma cada caso), **cuándo se activa** y **cuándo se desactiva**, y **qué procesos dejan de ser
>   necesarios** (si aplica).
> - Cuando la tarea **cambia el comportamiento**, incluir **dos** diagramas: **ANTES** y **DESPUÉS**,
>   marcando qué parte existía antes y qué parte cambia, más una frase que resuma la diferencia clave.
> - Trazar el camino completo (mensaje/endpoint → controller → servicio/use case → DB/colas/Mongo),
>   no solo el trozo modificado.
> - Mantenerlo sincronizado con el código real al cerrar cada fase (igual que el resto del doc).
> - Para bugs, anotar con ❌ dónde falla antes de escribir el fix.

```mermaid
flowchart TD
    A["Entrada (mensaje / endpoint)"] --> B["Controller"]
    B --> C{"¿bifurcación?"}
    C -->|"caso 1"| D["Use case / servicio"]
    C -->|"caso 2"| E["Ruta alternativa"]
    D --> F[("DB / cola / Mongo")]
```

---

<!--
## Phase N: Bug investigation — [short symptom]

Use this phase when a bug is found during validation or in production.
Copy, rename, and fill it in before writing any fix.

**Status:** `pending`

### Symptom

What the user or log observed. Be concrete.

### Flow traced

> Annotate the existing diagram with ❌ where the bug occurs.

```mermaid
flowchart TD
    A["POST /api/endpoint"] --> B["Controller"]
    B -->|"launcher = undefined ❌"| C["Service"]
    C -->|"if(launcher) → false ❌"| D["early return — launcher never saved"]
```

### Bugs confirmed

**Bug #N — [short label]**

File: `src/…:line`

Expected: …
Actual: …

### Files implicated in the fix

| File | Change needed |
|------|--------------|
|      |              |

### Design decision for next phase

Options considered and which one was chosen, with rationale.
-->
