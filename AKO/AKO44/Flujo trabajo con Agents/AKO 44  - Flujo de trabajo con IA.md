

![[Pasted image 20260623112604.png]]

El esquema de arriba resume las dos cosas. Y para arrancar, estos son los mensajes exactos que le mandas a cada IA (uno por proyecto, por turnos):

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