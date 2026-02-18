
[Recurso](https://docs.gitlab.com/user/ssh/)

### Chuleta SSH para GitLab (macOS)

### Ver si existen archivos de claves

`ls ~/.ssh`

- Ejemplo típico:
    

`id_ed25519  id_ed25519.pub`

o know hosts (Es un **archivo que guarda los “hosts confiables”** con los que te has conectado por SSH.)
### Crear una nueva clave SSH

`ssh-keygen -t ed25519 -C "daniromanpt@gmail.com"`

- Ruta por defecto: `~/.ssh/id_ed25519`
    
- `.pub` → clave pública
    
- `.id_ed25519` → clave privada (nunca se comparte)
    

### Añadir la clave al agente SSH

`eval "$(ssh-agent -s)"       # iniciar agente ssh-add ~/.ssh/id_ed25519     # añadir clave`

### Listar claves cargadas en el agente

`ssh-add -l`

- Muestra las claves activas y sus fingerprints
    

### Borrar claves del agente

`ssh-add -D`

- Quita todas las claves cargadas (no borra los archivos físicos)
    

### Ver contenido de las claves

`cat ~/.ssh/id_ed25519.pub    # ver clave pública (para GitLab) cat ~/.ssh/id_ed25519        # ver clave privada (no compartir)`

### Borrar archivos de claves

`rm ~/.ssh/id_ed25519*`

- ⚠️ Borra **todas las claves relacionadas** (solo si quieres limpiar todo)
    

###  Subir clave pública a GitLab

1. GitLab → **Preferences → SSH Keys**
    
2. Pegar el contenido de `id_ed25519.pub`
    
3. Title descriptivo: `GitLab Laptop Daniel`
    
4. Add key ✅
    

### Probar conexión SSH

`ssh -T git@gitlab.ako.com`

- Esperado: `Welcome to GitLab, @daniromanpt!`
    

**Si falla por VPN/UID:**

`ssh -T -l git git@gitlab.com`

- Forza usar el usuario `git` y evita conflictos de UID.
    

---

💡 **Tips finales**

- Mantén la **clave privada segura** (`id_ed25519`)
    
- Nombrar la clave con tu **email y servicio** ayuda a distinguir varias claves
    
- Para no cargar cada vez el agente, agrega al `~/.zshrc`:
    

`eval "$(ssh-agent -s)" ssh-add ~/.ssh/id_ed25519`


---

>[!Error] Error con el VPN conectado 


 # SSH + GitLab + VPN: error UID y solución

### 1️⃣ Problema que apareció

- Error al usar SSH/GitLab con VPN activa:
    

`No user exists for uid 1471442371 Permission denied (publickey)`

- Causas:
    
    - La VPN altera cómo macOS identifica tu **UID/usuario local**.
        
    - SSH intenta usar tu usuario local para autenticarse → falla.
        
    - No es problema de la clave SSH, sino de **conflicto UID/VPN**.
        

---

### 2️⃣ Pasos previos importantes

1. Revisar si hay claves SSH:
    

`ls ~/.ssh`

- Solo aparece `known_hosts` → no hay claves todavía.
    

2. Crear clave SSH fuera de la VPN:
    

`ssh-keygen -t ed25519 -C "daniromanpt@gmail.com" ssh-add ~/.ssh/id_ed25519`

- Se crean `id_ed25519` (privada) y `id_ed25519.pub` (pública).
    

3. Subir clave pública a GitLab.
    

---

### 3️⃣ Solución final

- Crear **archivo de configuración SSH**: `~/.ssh/config`
    

`Host gitlab.ako.com   User git   HostName gitlab.ako.com   IdentityFile ~/.ssh/id_ed25519   IdentitiesOnly yes`

- Esto fuerza:
    
    - Usar el usuario `git` en GitLab.
        
    - Usar la clave correcta.
        
    - Ignorar el UID local que falla con VPN.
        
- Con esto, ahora se puede:
    

`git clone git@gitlab.ako.com:software/akonet.cloud/akocloud-api.git ssh -T git@gitlab.ako.com`

✅ Funciona incluso con VPN activa.

