### Mirar logs

Carpeta contenedora para poder mirar los logs

 /Users/daniel.roman/.npm/_logs/2026-01-13T16_23_09_207Z-debug.log

```sh
npm install 

npm start

npm run... 

#Mirar instrucción del script
```

### Añadir variables de entorno al comando

- Al ejecutar `BC=cl MICRO=coap npm start`, las partes `BC=cl` y `MICRO=coap` son **variables de entorno**.
- Estas variables existen **solo para ese comando** y permiten pasar información al proceso Node sin modificar el código.
- `npm start` ejecuta el script `"start"` definido en `package.json`.
- Dentro del código Node, se pueden leer con `process.env.BC` y `process.env.MICRO`.
- Según los valores de estas variables, la aplicación puede arrancar **microservicios distintos** usando el mismo proyecto.
