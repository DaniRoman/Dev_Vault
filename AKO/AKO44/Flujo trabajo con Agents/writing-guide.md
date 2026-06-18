# Task document writing guide

This guide explains how to fill in `task-template.md` so that AI agents and human reviewers can pick up a task cold without asking questions.

## When to create a task document

Create one before touching any code if the task:
- spans more than two files,
- requires understanding an existing flow before changing it, or
- might need a bug investigation phase.

Copy `task-template.md`, rename it `[TICKET-ID]-[slug].md`, and place it under `docs/ia/`.

## Document structure and order (read this first)

The template starts with a non-negotiable **"Reglas de estructura y claridad"** block. Follow that order
strictly — the document must be understandable by **someone who does not know the task**, without reading
the code:

1. Objetivo → 2. Scope → 3. **Glosario** → 4. **Contexto funcional** → 5. **Flujo ANTES** →
   6. **Problema/limitación** → 7. **Implementación paso a paso** → 8. **Flujo DESPUÉS** →
   9. **Resultado y validación** → Apéndices.

Key rules when filling these:

- **Do not open with the final architecture.** Explain how the system worked **before** and what was wrong
  with it first.
- **Glosario:** one row per internal term you use (micro, handler, sample, serie, umbral, job, aggregate,
  activación/desactivación, queue/collection names, identifiers…). If you write "the micro saves the
  series", the reader must be able to look up *which* micro, what the *series* is, and what gets saved.
- **Contexto funcional:** what the affected microservice is for and what it normally does with its input.
- **Flujo ANTES vs DESPUÉS:** one Mermaid diagram each when the behavior changes (see "Code flow diagrams").
- **Implementación paso a paso:** numbered steps (what changed, what was added, what stopped depending on
  other processes, what bug was fixed) — *before* showing the final diagram.
- **Resultado, validación end-to-end y bugs corregidos van al FINAL**, never in the objective.

`KNT-2307-cloud-alarm-temp-maxmin.md` is the reference example of this structure.

## Filling in each section

### Metadata — Status field

Use these values only:

| Value         | Meaning                                      |
|---------------|----------------------------------------------|
| `planned`     | Doc created, no phase started yet            |
| `in-progress` | At least one phase started, task not shipped |
| `done`        | All phases complete, branch ready for PR     |

Always append a short qualifier after `in-progress` when useful:
`in-progress — bug fix`, `in-progress — blocked on external service`.

### Code flow diagrams

Add a Mermaid `flowchart` or `sequenceDiagram` in **"Flujo ANTES"** to trace the path as it worked before
the task, and a second one in **"Flujo DESPUÉS"** to show the final behavior. End with a one-line summary of
the key difference. If a bug is found, annotate the relevant diagram with ❌ markers before writing the fix.

The final diagram must be **pedagogical, not just correct** (see the template's "Flujo final — OBLIGATORIO"):

- Label each node with *what it does* in plain language, not only the class/column name.
- Add a **legend** (what each microservice represents, which emoji/color marks new vs. unchanged paths) and
  group nodes by microservice with `subgraph`.
- Make explicit: what data enters, which decisions are taken (branches + which case takes which branch),
  when an alarm/state activates, when it deactivates, and which processes are no longer needed.

**When to use `flowchart`:**
Use for request → controller → service → DB flows and for showing where a bug sits.

```mermaid
flowchart TD
    A["POST /api/firmware-update/:id/launch"] --> B["FirmwareUpdateController.launch()"]
    B -->|"launcher = context.user._id ✅"| C["FirmwareUpdateService.setLauncher()"]
    C --> D[("FirmwareUpdate doc\nlauncher saved")]
    B --> E["FirmwareUpdateService.assignDevices()"]
```

**When to use `sequenceDiagram`:**
Use when timing or ordering between services matters (e.g. API → RabbitMQ → Translator).

```mermaid
sequenceDiagram
    participant UI
    participant API
    participant AMQP
    participant Translator
    UI->>API: POST /launch
    API->>AMQP: sendCmd12830 {ref: "server_command_fota_update"}
    AMQP->>Translator: message received
    Translator->>Translator: CMDParser.lookup(ref)
```

**Rules:**
- Keep diagrams focused. One diagram per flow, not one diagram for the whole system.
- Label edges with the key variable or condition, not generic "calls".
- Add ❌ or ✅ to highlight the bug or the fix.
- Do not paste full code into diagrams. One node = one responsibility.

### Decisions and risks table

The table in Phase 1 forces you to record the **why**, not just the what.
If you can't fill in the "Reason" column, the decision isn't ready to be made.

### Bug investigation phase

When validation reveals a bug, **do not jump straight to the fix**.
Instead:

1. Copy the bug investigation block from the template comment at the bottom.
2. Add it as a new phase (`Phase N: Bug investigation — [symptom]`).
3. Add the phase to the phases table with status `pending`.
4. Trace the flow, confirm each bug with file + line evidence.
5. List the fix options, pick one, and document why.
6. Only then add a `Phase N+1: Bug fix` phase and start coding.

This discipline prevents fixing the wrong thing. The KNT-2303 task is a reference
example: Phase 4 traced three distinct bugs before Phase 5 touched any code.

### Validation checklist

Check items off as you go, not all at once at the end.
Add task-specific items when the generic ones don't cover the case:

```markdown
- [ ] `initDefinitions.js` re-run after JSON schema changes (see KNT-2303 pattern)
- [ ] Dependent service restarted after MongoDB update
- [ ] Log entry shows correct `launcherSnapshot.username`
```

### Problems found

Use this field in Phase 2 honestly. If you found debug `console.log` left in production
code, a fallback that silently drops data, or a related script that doesn't sync a field —
record it here. Do not silently fix it if it's out of scope; document it as a risk instead.

## Phase lifecycle

```
planned
  └─► (agent reads task) ─► in-progress
        └─► (phase N done, user confirms) ─► phase N+1 starts
              └─► (all phases done) ─► done
```

**Never advance a phase without explicit user confirmation.**
The agent should end every phase response with the phrase:
> "Phase N complete. Ready to start Phase N+1: [name] when you confirm."

## What not to put in a task document

- Full code blocks copied from source files. Reference by `file:line` instead.
- Long stack traces or raw log dumps. Summarize the key line.
- Outdated assumptions left as if they were facts. Strike them or delete them.
- Git history (`who changed what when`). That lives in `git log`.
- PR description prose. The task doc is for the agent; the PR description is for reviewers.
