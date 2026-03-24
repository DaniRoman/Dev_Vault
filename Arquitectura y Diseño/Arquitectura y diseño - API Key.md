Antes de ver soluciones, entiende **qué problema resuelven**:

> "¿Cómo sabe la API que la petición viene realmente de CL-perte y no de un atacante?"

Eso se llama **autenticación entre servicios**. Todas las soluciones (API Key, JWT, mTLS) resuelven eso de formas distintas.

Entiende las 3 soluciones de menos a más
#### Nivel 1 — API Key (la más simple)
Es como una **contraseña** compartida.

```bash
CL envía: GET /file/123  +  header: "X-API-Key: abc123"
API recibe: ¿el header tiene "abc123"? → sí → confiado
```

- Ambos conocen el secreto
- Si alguien lo roba → acceso permanente
- Para cambiarlo → tienes que cambiarlo en las dos máquinas

**Concepto detrás**: ninguno. Es simplemente un secreto compartido.
#### Nivel 2 — JWT con clave simétrica (HS256)

Es como una API Key pero que **caduca**.

```txt
CL: genera token con secreto "xyz" → token válido 5 min
API: verifica el token con el mismo secreto "xyz"
```

- Ambos conocen el secreto
- Mejor que API Key porque el token expira
- Pero si hackean la API → pueden crear tokens porque tienen el secreto