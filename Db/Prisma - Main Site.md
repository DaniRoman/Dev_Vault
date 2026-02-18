## Flujo si cambias el modelo
Aplicar migraciones y regenerar cliente
```sh
#!/bin/bash
# Prisma – Resumen de comandos para desarrollo y producción

# ------------------------------
# 1️⃣ Aplicar migraciones pendientes (producción/contenedor)
# Aplica todas las migraciones que existen en prisma/migrations
npx prisma migrate deploy

# ------------------------------
# 2️⃣ Crear nuevas migraciones (desarrollo)
# Genera migraciones según cambios en schema.prisma y las aplica
npx prisma migrate dev --name el-cambio-que-hice

# ------------------------------
# 3️⃣ Generar Prisma Client
# Genera el cliente que usa Node.js (PrismaClient)
npx prisma generate

# ------------------------------
# 4️⃣ Sincronizar schema con DB existente
# Trae la estructura actual de la base de datos y actualiza schema.prisma
npx prisma db pull


```

## Error de instalación de Cliente

>[!error] Error TS2305 – `PrismaClient` no exportado en Prisma 7 

En Prisma **v7**, si en `schema.prisma` usas `provider = "prisma-client"` con un `output` personalizado, el cliente **NO se genera en `@prisma/client`**.  
Por eso TypeScript muestra el error:  
`Module '"@prisma/client"' has no exported member 'PrismaClient'`.

```prisma
generator client {
  provider = "prisma-client"
  output   = "../generated/prisma"
}
```

```ts
//En este caso, la importación correcta es:
import { PrismaClient } from '../generated/prisma'
```

Si quieres importar desde `@prisma/client`, debes usar el provider clásico:

```prisma
generator client {
  provider = "prisma-client-js"
}
```

Y luego

```ts
import { PrismaClient } from '@prisma/client'
```

[Enlace a Resolución ](https://stackoverflow.com/questions/67746885/prisma-client-did-not-initialize-yet-please-run-prisma-generate-and-try-to-i)


## Errores
>[!Error] Error de timeout al conectar Node/Prisma con MySQL en Docker
El error de timeout ocurre cuando Prisma intenta abrir una conexión y MySQL aún no está listo para aceptar conexiones reales.
Aunque el contenedor esté “running”, eso no significa que la base de datos esté inicializada completamente.`depends_on` solo garantiza orden de arranque, no que el servicio esté listo.
Prisma entonces intenta conectarse varias veces y termina en **P1001** o pool timeout.
El problema es de sincronización entre contenedores, no de credenciales.
Se soluciona con un **healthcheck** en MySQL y `condition: service_healthy`.
También puede mitigarse con un script `entrypoint` que espere correctamente.
En producción, la solución correcta es usar healthcheck + retry controlado, no solo sleep manual. [[Docker Main Site#^dda853]]


## Indices En Prisma

[Recurso](https://blog.logrocket.com/how-configure-indexes-prisma/)

