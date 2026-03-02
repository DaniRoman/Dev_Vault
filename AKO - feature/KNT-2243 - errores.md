>[!error] No existe un export default para algún modulo

```js
import mongoose from "mongoose";
```

Sin esa opción, TypeScript es estricto y dice:

> “mongoose no tiene default export”

porque **mongoose normalmente exporta con CommonJS**, algo como:

```js
module.exports = mongoose;
```

y eso **no es** un `export default` de ESModules, y debería ser exportado de esta manera

```js
import * as mongoose from "mongoose";
```

`"esModuleInterop": true` es una opción de TypeScript que **facilita importar módulos CommonJS** (los que hacen `module.exports = ...`) usando sintaxis ESModule (`import ... from ...`) pudiendo utilzar 

```js
import mongoose from "mongoose";
```