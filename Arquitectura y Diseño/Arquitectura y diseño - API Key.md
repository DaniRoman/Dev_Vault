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

## Implementación de código


## Flujo completo: JWT RS256 para comunicación CL-perte → API

---

### Paso 1: Generar par de claves RSA

Creamos una carpeta `keys/` en la raíz del proyecto y generamos el par:

```bash
mkdir -p keys
cd keys
openssl genrsa -out private.pem 2048
openssl rsa -in private.pem -pubout -out public.pem
```

Resultado:
- `keys/private.pem` → clave **privada** (solo en la máquina de CL-perte, **nunca se comparte**)
- `keys/public.pem` → clave **pública** (se copia a la máquina de la API)

---

### Paso 2: Proteger las claves del repositorio

Añadimos al `.gitignore`:

```
keys/
*.pem
```

Para que las claves **nunca suban a Git**.

---

### Paso 3: Instalar dependencia `jsonwebtoken`

```bash
npm install jsonwebtoken
npm install --save-dev @types/jsonwebtoken
```

---

### Paso 4: Modificar `messageProcessor.ts` (CL-perte)

#### 4.1 — Añadir imports

```ts
import * as path from "path";
import jwt from "jsonwebtoken";
import rp from "request-promise-native";
```

#### 4.2 — Reescribir `getFile`

**Antes** (leía del disco local):
```ts
private async getFile(id: string): Promise<Buffer> {
    const file = await this.filesRepo.getByParameter("_id", id, null);
    if (!file) throw new FileNotFoundError();
    const content: Buffer = fs.readFileSync(file.path);
    if (!content) throw new FileEmptyError();
    return content;
}
```

**Después** (pide a la API con JWT firmado):
```ts
private async getFile(id: string): Promise<Buffer> {
    // 1. Leer clave privada
    const privateKeyPath = process.env.SERVICE_PRIVATE_KEY_PATH 
        || path.join(__dirname, "../../../../keys/private.pem");
    const privateKey = fs.readFileSync(privateKeyPath);

    // 2. Firmar JWT (expira en 5 min)
    const token = jwt.sign(
        { service: "cl-perte" }, 
        privateKey, 
        { algorithm: "RS256", expiresIn: "5m" }
    );

    // 3. Pedir el binario a la API
    const apiUrl = process.env.API_URL || "https://tu-api.com";
    const content: Buffer = await rp({
        method: "GET",
        uri: `${apiUrl}/api/file/${id}/binary`,
        headers: {
            "Authorization": `Bearer ${token}`
        },
        encoding: null  // IMPORTANTE: recibir Buffer, no string
    });

    if (!content || content.length === 0) {
        throw new FileEmptyError();
    }
    return content;
}
```

---

### Paso 5: Crear middleware en la API (otro repo)

#### 5.1 — Crear `middlewares/serviceAuth.ts`

```ts
import jwt from "jsonwebtoken";
import fs from "fs";

const publicKey = fs.readFileSync(process.env.CL_PUBLIC_KEY_PATH);

export function serviceAuth(req, res, next) {
    const authHeader = req.headers.authorization;
    
    if (!authHeader || !authHeader.startsWith("Bearer ")) {
        return next(); // no es JWT, sigue con auth normal
    }
    
    try {
        const token = authHeader.split(" ")[1];
        const decoded = jwt.verify(token, publicKey, { 
            algorithms: ["RS256"] 
        });
        req.serviceAuth = true;
        req.serviceName = decoded.service;
        next();
    } catch (err) {
        return res.status(401).json({ error: "Invalid service token" });
    }
}
```

#### 5.2 — Registrar middleware en la ruta

```ts
import { serviceAuth } from "./middlewares/serviceAuth";

app.express.get("/api/file/:id/binary*", serviceAuth, (req, res, next) => {
    this.handleResponse((new this(req.context)).getBinary(req.params.id, req, res), req, res, next);
});
```

#### 5.3 — Modificar `getBinary` para aceptar `req`

```ts
// Antes:
getBinary(id: string, res: any): Promise<any> {
    return this._checkPrivilege()

// Después:
getBinary(id: string, req: any, res: any): Promise<any> {
    const privilegeCheck = req.serviceAuth 
        ? Promise.resolve() 
        : this._checkPrivilege();
    
    return privilegeCheck
      .then(() => {
          return this.Model.getById(id);
      })
      // ... el resto igual
```

---

### Paso 6: Variables de entorno

**En CL-perte:**
```
API_URL=https://tu-api.com
SERVICE_PRIVATE_KEY_PATH=/opt/certs/private.pem
```

**En la API:**
```
CL_PUBLIC_KEY_PATH=/opt/certs/cl-perte-public.pem
```

---

### Resumen del flujo en ejecución

```
1. CL-perte recibe petición FOTA de un dispositivo
2. Busca la última actualización compatible en BD
3. Obtiene el fileId del firmware
4. Lee private.pem del disco
5. Firma un JWT: { service: "cl-perte", exp: 5min }
6. Llama: GET https://api/api/file/{fileId}/binary + Authorization: Bearer <jwt>
7. API → middleware serviceAuth → verifica JWT con public.pem
8. API → getBinary → salta _checkPrivilege → busca file → devuelve binario
9. CL-perte recibe Buffer → lo envía al dispositivo por CoAP
```

---

### Conceptos clave

| Concepto | Qué es |
|---|---|
| **RS256** | Algoritmo de firma asimétrica (RSA + SHA-256) |
| **Clave privada** | Solo la tiene CL-perte. Firma los tokens. |
| **Clave pública** | La tiene la API. Solo puede verificar, no crear tokens. |
| **`encoding: null`** | Obliga a `request-promise-native` a devolver Buffer (no string) |
| **`expiresIn: "5m"`** | El JWT caduca en 5 minutos. Si lo interceptan, dura poco. |
| **`serviceAuth` middleware** | Verifica el JWT y marca `req.serviceAuth = true` |

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