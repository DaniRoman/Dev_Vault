

![[Pasted image 20260623112604.png]]

El esquema de arriba resume las dos cosas. Y para arrancar, estos son los mensajes exactos que le mandas a cada IA (uno por proyecto, por turnos):

**Cómo coordinas una feature nueva (la receta, por turnos):**

Paso 0 — tú creas la carpeta, copias las 4 plantillas, y rellenas la visión funcional del `overview.md` (qué es la feature, para qué sirve), aunque sea en dos frases. Eso le da el norte a las tres IAs.

Paso 1 — abres la **IA de api** primero (ve los dos lados) y le dices:

```
Feature nueva: <nombre>. Lee docs/features/<nombre>/overview.md (visión funcional)
y api.md. Documenta tu capa en api.md desde tu código: endpoints, qué recibes,
qué emites. En el overview, añade a la tabla de seams las filas de tus fronteras
(qué recibes de cliente, qué mandas a micro) como ⏳ por confirmar.
Donde el flujo entre en cliente o micro, deja ⏳ PENDIENTE. No inventes su scope.
```

Paso 2 — abres la **IA de micro**, mismo encargo para `micro.md`, y además: _"confirma o corrige las filas de seam que dejó api que te afecten"_.

Paso 3 — abres la **IA de cliente**, igual para `client.md`, y confirma su seam con api.

Paso 4 — vuelves a una IA (la de api suele ir bien) y le dices: _"con las tres capas ya rellenas, completa el flujo end-to-end (sección 2) y el diagrama del overview"_. Ese paso es el ensamblaje final.

**Para documentar una feature nueva** (el caso de ahora), a cada IA por separado:

```
Lee docs/features/<feature>/overview.md y <tu-capa>.md.
Documenta SOLO tu capa en su archivo, desde lo que ves en tu código.
Donde el flujo entre en otra capa, deja un bloque ⏳ PENDIENTE con el
contrato que ves desde la tuya. No inventes el scope de las otras.
Confirma o corrige las filas de la tabla de seams del overview que te toquen.
```

Cambias `<tu-capa>` por `client`, `api` o `micro` según a qué IA se lo digas. El orden no importa mucho, pero suele ir bien empezar por `api` (es la que ve los dos lados: qué recibe del cliente y qué manda al micro).

**Para un trabajo concreto** (un bug o un cambio), a la IA del proyecto que toque:

```
Lee docs/features/<feature>/overview.md.
Copia docs/templates/work-template.md a docs/work/incidents/ (o tasks/),
fija Tipo, y trabaja solo la fase que te diga. Enlaza la feature, no la
reescribas. Al cerrar, reconcilia el doc de feature.
```

La idea clave que ves en el dibujo: las IAs **no se hablan entre ellas**, se pasan el testigo a través de los archivos. Una escribe su parte y deja constancia en la tabla de seams; la siguiente lee eso y confirma su lado. Tú eres quien va abriendo cada IA por turnos.

Para tu piloto, lo único que falta es abrir la IA de `cliente` con el primer mensaje (su `client.md` ya tiene el hueco preparado).