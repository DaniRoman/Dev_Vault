
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


