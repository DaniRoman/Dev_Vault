
[Recurso](https://lenguajejs.com/npm/introduccion/instalacion-node-con-nvm/)

**Node Version Manager**  
Permite **instalar, cambiar y gestionar versiones de Node.js** en un mismo sistema.

Sirve para
- Usar **varias versiones de Node**
- Cambiar de versión **sin reinstalar**
- Evitar conflictos entre proyectos
- No usar `sudo` con npm

### Comandos básicos

```sh 
nvm install 20        
# instala Node 20 
nvm install --lts    
# instala versión LTS 
nvm use 20           
# usa Node 20 
nvm use --lts        
# usa LTS nvm ls               
# versiones instaladas 
nvm current          
# versión activa
```

###  Versiones por proyecto

Archivo `.nvmrc`:

`18`

Uso:

`nvm use`

###  Relación con npm

- **nvm** → gestiona versiones de **Node**

- **npm** → gestiona **paquetes**


###  Dónde se instala

`~/.nvm`

Se carga desde:

`~/.zshrc`

### 📌 Verificación

`node -v npm -v which node`

>[!Error] Error
>
>al instalar node 14
># Node 14 + Mac M1/M2 (nvm)

- ❌ Error: `zcfree … C23` → falla compilación Node 14 en ARM64.
    
- ⚠ Causa: Node 14 antiguo + clang moderno → incompatible con ARM64.
    
- ✅ Solución rápida:
    
    - Usar Node moderno: `nvm install 18 && nvm use 18`
        
    - Node 14 legacy (Rosetta):
        
        `arch -x86_64 zsh -c 'export NVM_DIR="$HOME/.nvm"; [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"; nvm install 14; nvm use 14'`
        
- 💡 Nota: npm no falla → problema de compilación Node, no paquetes. 