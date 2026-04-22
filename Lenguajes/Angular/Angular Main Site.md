
## Como identificar los modulos

## Chuleta corta para este proyecto

### 1) Quiero saber qué hace una página
Mira primero:

- `src/app/main/<modulo>/pages/manager/manager.component.ts`

Ahí suele estar casi todo:
- campos
- columnas
- acciones
- qué detalle usa
- qué edit usa

---

### 2) Quiero saber qué API llama
Mira:

- `src/app/main/<modulo>/module.service.ts`

Ahí verás algo como:
- `protected _apiUrl = "/api/plmn-list"`

---

### 3) Estoy en el popup de New/Edit
En `manager.component.ts` busca:

- `bEditComponent`

Si existe, mira:
- `pages/manager/edit.component.ts`
- `pages/manager/edit.html`

Si no existe, el formulario es genérico.

---

### 4) Estoy en el panel de detalle
En `manager.component.ts` busca:

- `bDetailComponent`

Si existe, mira:
- `pages/manager/detail.component.ts`
- `pages/manager/detail.html`

Si no existe, el detalle es genérico.

---

### 5) Quiero saber por qué un botón sale o no sale
Mira en:

- `manager.component.ts`

Busca:
- `actions`
- `hide: (model) => ...`

---

### 6) Quiero saber por qué un campo sale o no sale
Mira en:

- `manager.component.ts`

Busca:
- `schema`

---

## En resumen, orden bueno

Para cualquier pantalla:

1. `manager.component.ts`
2. `module.service.ts`
3. `edit.component.ts/html` o `detail.component.ts/html`

---

## En PLMN concretamente

- comportamiento general:
  - `src/app/main/plmn-list/pages/manager/manager.component.ts`
- API:
  - `src/app/main/plmn-list/module.service.ts`
- popup New/Edit:
  - `src/app/main/plmn-list/pages/manager/edit.component.ts`
- detalle:
  - `src/app/main/plmn-list/pages/manager/detail.component.ts`


### Chuleta 2

Voy a leer todos los ficheros relevantes en paralelo para darte un mapa completo y preciso.

Ahora tengo el mapa completo. Te lo explico ordenado por orden de ejecución real.

---

## Flujo completo: de click en menú a modal renderizado

---

### PASO 1 — Click en el menú lateral: `module.menu.ts`

```
module.menu.ts
```

El ítem del menú registra una ruta, un título y un icono. No hace nada más. Solo es configuración:

```ts
{ id: "deviceAdvancedManagement", route: "device-advanced/management", title: "Device advanced" }
```

Cuando el usuario hace click, Angular Router navega a `device-advanced/management`.

---

### PASO 2 — El router carga el módulo: `module.routing.ts` → `pages.routing.ts`

```
module.routing.ts  →  pages/pages.routing.ts
```

Son dos niveles de routing anidado:

1. `module.routing.ts` detecta la ruta `/device-advanced` → carga **lazy** `DeviceAdvancedPageModule`
2. `pages.routing.ts` detecta el subnivel `/management` → monta `DeviceAdvancedManagerComponent` dentro de `PageComponent`

`PageComponent` es el layout general (menú lateral + contenido). Dentro de él se renderiza el manager.

---

### PASO 3 — Angular instancia `DeviceAdvancedManagerComponent`

```
pages/manager/manager.component.ts
```

Este componente **no tiene template propio**: usa `../../../../shared/manager/manager.html`, que es una sola línea:

```html
<bManager [entityJson]="entityJson" [service]="service" ...></bManager>
```

En el constructor se construye el objeto `entityJson: IBManagerSchema`, que es **la configuración de todo**: columnas, grupos, opciones y acciones. Es la única responsabilidad de este componente. No renderiza nada directamente.

---

### PASO 4 — `BManagerComponent` monta la vista dividida

```
b-manager/b-manager.component.ts  +  b-manager.html
```

`bManager` es el componente genérico que recibe `entityJson` y lo interpreta. Renderiza una **split view**:

```html
<split direction="horizontal">
  <split-area [size]="70">
    <bList ...></bList>       <!-- Tabla/grid (izquierda) -->
  </split-area>
  <split-area [size]="30">
    <bDetail ...></bDetail>   <!-- Panel detalle (derecha) -->
  </split-area>
</split>
```

En `ngOnInit` también procesa el array `entityJson.actions` y **asigna automáticamente `fn`** a los tipos conocidos:

| `type` en actions | Lo que hace `BManagerComponent` automáticamente |
|---|---|
| `"detail"` | Asigna `fn = (row) => this._openDetail(row._id)` |
| `"edit"` | Asigna `fn = (row) => router.navigate([...], { queryParams: { edit: row._id } })` |
| `"delete"` | Asigna `fn = (row) => this._delete(row)` |
| cualquier otro | La `fn` ya la definiste tú en `manager.component.ts` (ej. `"edit-network"`, `"cmd"`) |

---

### PASO 5 — `bList` renderiza la tabla y el menú de tres puntos

```
b-manager/b-list/b-list.html
```

`bList` usa `bGrid` (ag-Grid) para mostrar las filas. Cada fila tiene un menú contextual de tres puntos que itera sobre `entityJson.actions` y llama a `action.fn(model)` al hacer click.

Cuando el usuario hace **doble click** en una fila, `bGrid` emite `rowDoubleClick` → `bList` llama a `openDetail()` → equivale a pulsar "Detail".

---

### PASO 6 — El usuario abre el menú y pulsa "Edit"

```
b-manager.component.ts  →  _openModal()
```

La acción `type: "edit"` que asignó `BManagerComponent` navega a:

```
/device-advanced/management?edit=<id>
```

El router emite el cambio de queryParams. `BManagerComponent` está suscrito a `activatedRoute.queryParams` y detecta el param `edit` → llama a `_openModal(id)`:

```ts
_openModal(id: string): void {
  return this.modal.open(BEditComponent, { width: "1000px" }, {
    entityJson: this.entityJson,
    service: this.service,
    modelId: id,
    component: this.entityJson.options.bEditComponent  // ← AdvancedDeviceEditComponent
  });
}
```

`BModalService.open` llama internamente a `MdDialog.open(BEditComponent)` y pasa los inputs.

---

### PASO 7 — `BEditComponent` se renderiza dentro del modal

```
b-manager/b-edit/b-edit.component.ts
```

Este es el **contenedor genérico del modal de edición**. Él mismo inyecta `MdDialogRef` (por eso puede cerrarse). Hace dos cosas en `ngOnInit`:

1. Llama a `service.getById(modelId, { select: ... })` → obtiene el modelo de la API
2. Si hay `component` (= `bEditComponent`), crea dinámicamente ese componente con `ComponentFactoryResolver`:

```ts
const factory = this.componentFactoryResolver.resolveComponentFactory(this.component);
this.cmpRef = this.target.createComponent(factory);
this.cmpRef.instance.entityJson = this.entityJson;
this.cmpRef.instance.form = this.form;
this.cmpRef.instance.model = this.model;
```

El componente `AdvancedDeviceEditComponent` se renderiza **dentro** del `BEditComponent`, no en un segundo modal. `BEditComponent` pone la cabecera, los botones Save/Close y el contenido personalizado en el `<ng-template #target>`.

---

### PASO 8 — Para "Edit network": la `fn` personalizada abre directamente otro modal

```
manager.component.ts  →  fn: (model) => this.modal.open(AdvancedDeviceEditNetworkComponent)
```

Las acciones que no son `detail/edit/delete` tienen `fn` definida directamente en `manager.component.ts`. Cuando el usuario pulsa "Edit network":

1. Se llama `fn(model)` directamente desde el menú de tres puntos
2. La `fn` hace un `GET /api/device` con `select: "network serialNumber name"` para asegurarse de tener los datos actualizados
3. Llama a `this.modal.open(AdvancedDeviceEditNetworkComponent, options, inputs, outputs)`
4. `BModalService` abre `MdDialog` con `AdvancedDeviceEditNetworkComponent` directamente, **sin pasar por BEditComponent**
5. El modal pasa los datos vía `inputs: { model: fullModel }` y suscribe los `@Output` via `outputs: { onSaveNetwork: ..., onRequestNetSync: ... }`

---

## Mapa de dependencias completo

```
Router
  └─ DeviceAdvancedManagerComponent (manager.component.ts)
       │  construye entityJson (schema + actions + options)
       │  templateUrl → manager.html → <bManager>
       │
       └─ BManagerComponent (b-manager.component.ts)
            │  interpreta entityJson
            │  asigna fn a actions detail/edit/delete
            │
            ├─ bList  ──── bGrid (tabla ag-Grid)
            │               └─ menú 3 puntos → action.fn(model)
            │
            ├─ bDetail  (panel lateral derecho)
            │
            └─ Al click "Edit" → router ?edit=id → BManagerComponent._openModal()
                  └─ BModalService.open(BEditComponent)
                        └─ BEditComponent (modal genérico)
                              │  getById → carga modelo
                              │  ComponentFactoryResolver → crea bEditComponent
                              └─ AdvancedDeviceEditComponent (renderizado dentro)

            └─ Al click "Edit network" → fn() directa en manager.component.ts
                  └─ GET /api/device (select network)
                        └─ BModalService.open(AdvancedDeviceEditNetworkComponent)
                              │  inputs: { model }
                              └─ outputs: { onSaveNetwork, onRequestNetSync }
```

---

## Cómo replicarlo para una funcionalidad nueva

Para añadir una acción nueva tipo "Edit network" en cualquier módulo:

1. **Crea el componente modal** (solo `@Input/@Output`, sin inyectar servicios que puedan dar problemas)
2. **Regístralo** en el `@NgModule` en `declarations` y `entryComponents`
3. **Añade la acción** en el array `actions` del `entityJson` con `type: "nombre-que-quieras"` y una `fn` que llame a `this.modal.open(...)`
4. **Pasa datos** vía el tercer argumento de `modal.open` (inputs) y suscríbete a outputs vía el cuarto argumento
5. Opcionalmente, usa `this.modal.confirm(...)` para pedir confirmación antes de ejecutar acciones destructivas

Para el **Edit estándar** (con formulario genérico), basta con:
- Crear un componente y asignarlo a `options.bEditComponent`
- `BEditComponent` hace todo el resto automáticamente (carga datos, Save, Close con confirmación)
