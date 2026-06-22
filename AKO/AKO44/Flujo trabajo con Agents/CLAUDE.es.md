# CLAUDE.md

@AGENTS.md

## Notas de Claude Code

Claude Code carga este archivo como memoria del proyecto. La línea `@AGENTS.md` importa las instrucciones compartidas para agentes desde la raíz del repositorio para que Claude y Codex puedan seguir las mismas reglas.

Cuando trabajes en una tarea documentada:

1. `AGENTS.md` se carga automáticamente mediante el import `@AGENTS.md` anterior; no hace falta volver a leerlo manualmente.
2. Lee el documento de la tarea dentro de `docs/ia/`.
3. Trabaja solo en la fase solicitada.
4. Actualiza el documento de la tarea después de la fase.
5. No pases a la siguiente fase sin confirmación explícita.

Si parece que Claude no sigue estas instrucciones, ejecuta `/memory` y verifica que este archivo `CLAUDE.md` esté cargado.
