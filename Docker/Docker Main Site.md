>[!warning] Cosas que quiero hacer!!
>Crear DockerFile de mi aplicación y meterlo en un servicio con un docker compose!!!

[Recurso](https://www.novasphereailab.com/blog/dockerizing-nodejs-applications-guide-2026)
## Flujo para arrancar un compose con DockerFile

- Primera vez → `docker compose up`
- Cambié Dockerfile o deps → `docker compose up --build`
- Solo cambió `.env` → **NO build**, solo `up`

## Acceder a localhost desde dentro de un contenedor

>[!warning] Recuerden!
>Cuando un contenedor Docker intenta acceder a `localhost`, apunta a sí mismo, no a tu máquina host. Para llamar a un servicio que corre en tu host desde dentro del contenedor, hay que usar `host.docker.internal` (o la IP de tu host en Linux). Esto permite que el contenedor Node haga peticiones reales al servicio que corre en tu máquina.
>[Recurso](https://docs.docker.com/desktop/features/networking/networking-how-tos/#use-cases-and-workarounds)

## Docker volumes 

[Recurso](https://oneuptime.com/blog/post/2026-01-16-docker-bind-mounts-vs-volumes/view)

## Docker Compose Health Checks

^dda853
[Recurso](https://last9.io/blog/docker-compose-health-checks/)