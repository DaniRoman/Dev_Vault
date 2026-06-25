
Un **sistema de documentación compartido** para que tres IAs (una por repo: `cliente`, `api`, `micro`) documenten y trabajen sobre un producto que cruza varios proyectos, sin que ninguna invente la parte de otra.

La idea central es una separación en **dos capas**:

- **`features/`** — conocimiento duradero: _cómo funciona el producto_. Una carpeta por feature, con un `overview.md` (visión + flujo + tabla de seams) y un fichero por capa (`client.md`, `api.md`, `micro.md`).
- **`work/`** — lo temporal: _lo que investigas o cambias ahora_ (incidencias y tareas). **Consume** las features, no las reescribe.

Y el mecanismo que hace que las IAs colaboren sin pisarse: cada IA es **dueña de su capa**, documenta solo hasta su frontera, deja **⏳ PENDIENTE** lo que es de otra, y los puntos de contacto se cierran en una **tabla de seams** que confirman los dos lados. Las IAs no se hablan: se comunican escribiendo en el mismo archivo, por turnos.

Lo que existe hoy: las plantillas, las guías con los prompts, las reglas en `AGENTS.md`/`CLAUDE.md`, y una feature real ya sembrada (`notification-rules`) como ejemplo.

![[Pasted image 20260623112604.png]]

### Cómo será el flujo

Hay dos flujos según lo que necesites. En ambos tú coordinas, abriendo cada IA por turnos.

**Flujo A — documentar una feature** (cuando quieres dejar registrado cómo funciona algo):

1. Tú creas la carpeta de la feature, copias las plantillas y escribes la visión funcional en dos frases.
2. Abres la IA de `api` → documenta `api.md` y declara sus fronteras en la tabla de seams.
3. Abres la IA de `micro` → documenta `micro.md` y confirma/corrige los seams que le tocan.
4. Abres la IA de `cliente` → documenta `client.md` y confirma su seam con api.
5. Una IA ensambla el flujo end-to-end del overview con las tres capas ya escritas.

Resultado: una feature documentada que servirá de contexto para todo lo que venga después. (Prompts en `guides/document-a-feature.md`.)

**Flujo B — trabajar un problema o un cambio** (el día a día):

1. Tú le das a la **primera IA** (normalmente `api`) la **descripción del problema**. Ella lee la feature como contexto, **crea** el doc de trabajo en `docs/work/`, rellena su capa y deja ⏳ PENDIENTE lo de las otras.
2. Abres las **siguientes IAs** → cada una rellena su hueco mirando el contexto de la feature y confirma sus seams.
3. Al cerrar, si se aprendió algo que cambia cómo funciona la feature, se **reconcilia** el doc de feature (la capa 1 queda al día).

Resultado: el problema resuelto y documentado, y la base de conocimiento más completa que antes. (Prompts en `guides/work-on-an-issue.md`.)

**La regla que sostiene todo el sistema**, en una frase: cada IA escribe solo lo suyo, deja marcado lo ajeno, y el archivo compartido en `docs/` es el punto de encuentro donde se cierran las fronteras. Eso es lo que evita que una IA invente el scope de otra, que era tu problema de partida.

vamos a empezar a implemematr loop ingeniering para que solo se encargue una para usar tareas y no tener que habalr una por una