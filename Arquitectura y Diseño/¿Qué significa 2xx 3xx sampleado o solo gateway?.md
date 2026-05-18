

Esto aplica principalmente a tráfico HTTP.

## 2xx / 3xx

Son respuestas exitosas o redirecciones:

```
200 OK201 Created204 No Content301 Redirect304 Not Modified
```

En sistemas con muchos microservicios, si cada petición exitosa se loggea en todos los servicios, el volumen explota.

Ejemplo:

```
API Gateway recibe request↓Service A↓Service B↓Service C↓Service D
```

Una sola operación puede generar 5 logs de éxito.

## “Sampleado”

Significa guardar solo una parte de esos logs.

Por ejemplo:

```
Guardar solo el 5% de requests exitosos.Guardar siempre errores.Guardar siempre requests lentos.Guardar siempre eventos de auditoría.
```

Ejemplo:

```
100.000 requests 200 OKsampling 5%→ solo se guardan 5.000 logs
```

Pero los errores no se samplean:

```
500 erroressampling ERROR 100%→ se guardan los 500
```

## “Solo gateway”

Significa que para requests exitosos normales, solo loggea la capa de entrada principal.

Ejemplo:

```
API Gateway loggea:POST /devices/{id}/commands 200 83msServicios internos no loggean cada 200 OK,salvo que haya error, latencia alta o evento de negocio.
```

Para vuestro caso:

```
API HTTP → puede aplicar gateway/sampling.Rabbit → no loggear cada mensaje exitoso si son miles; usar sampling o agregación.Device directo → no loggear cada paquete normal si hay alto volumen.
```