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

**Concepto detrás**: criptografía **simétrica** (1 clave, compartida)

#### Nivel 3 — JWT con claves asimétricas (RS256)

```md
CL: firma con clave PRIVADA → solo él puede firmar
API: verifica con clave PÚBLICA → solo puede verificar, no crear
```

- Si hackean la API → no pueden crear tokens
- Si interceptan un token → caduca en minutos
- Cada servicio puede tener su propio par de claves

**Concepto detrás**: criptografía **asimétrica** (2 claves, pública + privada)

## Implementar 
# Recurso para ampliar concepto

#### 3.1 — Primero ve este vídeo (concepto general)

Busca en YouTube:

> **"Public Key Cryptography - Computerphile"**

Son ~10 minutos. Explica con dibujos por qué funcionan 2 claves. Sin código, solo el concepto.

#### 3.2 — Después lee esto (JWT específico)

[https://jwt.io/introduction](https://jwt.io/introduction)

Lee solo estas secciones:

1. What is JSON Web Token?
2. How do JSON Web Tokens work?
3. Juega con el debugger → [https://jwt.io](https://jwt.io/) (pega un token y ve las partes)

#### 3.3 — Después lee la diferencia de algoritmos

[https://auth0.com/blog/rs256-vs-hs256-whats-the-difference/](https://auth0.com/blog/rs256-vs-hs256-whats-the-difference/)

Este artículo te explica **exactamente** la diferencia entre:

- HS256 (simétrico, 1 clave)
- RS256 (asimétrico, 2 claves)

#### 3.4 — Por último, la implementación en Node.js

[https://github.com/auth0/node-jsonwebtoken](https://github.com/auth0/node-jsonwebtoken)

Lee el README, específicamente:

- `jwt.sign()` → cómo firmar
- `jwt.verify()` → cómo verificar
- Los ejemplos con RSA