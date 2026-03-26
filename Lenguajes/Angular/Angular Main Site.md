
## Como identificar los modulos


## Regla práctica para este proyecto

En **este código**, casi siempre puedes orientarte así:

1. **mira la URL/pantalla**
2. busca una carpeta con ese nombre dentro de `src/app/main/`
3. empieza por `pages/manager/manager.component.ts`

En este proyecto, ese archivo suele ser el **centro de mando** de cada pantalla.

---

## Mapa mental rápido

Si estás en una página tipo:

- `plmn-list/management`
- `device-advanced/management`
- `firmware-update/management`

normalmente tienes esta estructura:

- `src/app/main/<modulo>/module.service.ts`
  - qué endpoint API usa
- `src/app/main/<modulo>/pages/manager/manager.component.ts`
  - columnas, acciones, schema, comportamiento general
- `src/app/main/<modulo>/pages/manager/detail.component.ts/html`
  - detalle personalizado, si existe
- `src/app/main/<modulo>/pages/manager/edit.component.ts/html`
  - formulario personalizado, si existe

---

## Qué archivo mirar según lo que ves en pantalla

### Si estás viendo la lista
Mira:

- `pages/manager/manager.component.ts`

Ahí suelen estar:

- columnas
- botones de acción
- condiciones `hide`
- campos del schema
- componentes custom de detalle/edición

---

### Si estás en “New” o “Edit”
Primero mira en `manager.component.ts` si existe:

- `bEditComponent`

Si existe, entonces el formulario real está en:

- `pages/manager/edit.component.ts`
- `pages/manager/edit.html`

Si **no** existe `bEditComponent`, entonces usa el formulario genérico y tienes que mirar:

- `src/app/buda-components/b-manager/b-edit/b-edit.component.ts`
- `src/app/buda-components/b-form/`

---

### Si estás viendo el panel derecho de detalle
Primero mira en `manager.component.ts` si existe:

- `bDetailComponent`

Si existe, el detalle real está en:

- `pages/manager/detail.component.ts`
- `pages/manager/detail.html`

Si no existe, usa el detalle genérico:

- `src/app/buda-components/b-manager/b-detail/`

---

### Si quieres saber qué API llama
Mira:

- `src/app/main/<modulo>/module.service.ts`

Ahí verás cosas como:

- `protected _apiUrl = "/api/plmn-list"`
- `protected _apiUrl = "/api/device"`

---

### Si quieres saber por qué aparece o no aparece un botón
Mira en:

- `pages/manager/manager.component.ts`

Busca:

- `"actions"`
- `"hide": (model) => ...`

Ahí suele estar exactamente la condición.

---

### Si quieres saber por qué aparece o no aparece un campo
Mira en:

- `pages/manager/manager.component.ts`

Busca:

- `"schema"`

Ahí se define:

- nombre del campo
- tipo
- si sale en lista
- si sale en detalle
- si sale en formulario

---

## En tu caso concreto: PLMN

Si estás en la página de **PLMN**, el recorrido bueno es este:

### 1) Comportamiento general
- `src/app/main/plmn-list/pages/manager/manager.component.ts`

### 2) Formulario “New/Edit”
- `src/app/main/plmn-list/pages/manager/edit.component.ts`
- `src/app/main/plmn-list/pages/manager/edit.html`

### 3) Panel de detalle
- `src/app/main/plmn-list/pages/manager/detail.component.ts`
- `src/app/main/plmn-list/pages/manager/detail.html`

### 4) API
- `src/app/main/plmn-list/module.service.ts`

---

## Cómo saberlo sin conocer Angular

Piensa así:

- **`manager.component.ts`** = qué hace la pantalla
- **`edit.component.*`** = cómo funciona el popup/formulario
- **`detail.component.*`** = cómo funciona la vista detalle
- **`module.service.ts`** = a qué API pega
- **`buda-components/...`** = comportamiento genérico compartido por todas las pantallas

No hace falta saber Angular al principio.  
Solo seguir el patrón de carpetas.

---

## Método de 30 segundos para cualquier pantalla

Cuando abras una pantalla nueva, haz esto:

1. identifica el nombre del módulo por la URL
2. abre `src/app/main/<modulo>/pages/manager/manager.component.ts`
3. revisa:
   - `schema`
   - `actions`
   - `options`
4. si en `options` ves:
   - `bEditComponent` → abre `edit.component.ts/html`
   - `bDetailComponent` → abre `detail.component.ts/html`
5. si no hay custom, mira los genéricos en:
   - `b-manager`
   - `b-form`
   - `b-grid`

---

## Truco para no perderte

Si quieres entender una pantalla, este es el orden más útil:

1. `manager.component.ts`
2. `module.service.ts`
3. `edit.component.ts/html` o `detail.component.ts/html`
4. componentes genéricos `buda-components`

---

## Aplicado a lo que tienes abierto ahora

Como tienes abierto:

- `src/app/main/plmn-list/pages/manager/detail.component.ts`

eso significa que estás mirando el **detalle personalizado** de PLMN.

Pero si quieres entender **toda la pantalla PLMN**, el archivo más importante sigue siendo:

- `src/app/main/plmn-list/pages/manager/manager.component.ts`

porque desde ahí ves:

- qué acciones salen
- qué schema usa
- qué detalle usa
- qué edit usa

---

## Si quieres, te hago una chuleta

Puedo darte una **chuleta de 8 líneas** del tipo:

- “si veo X, mira Y”
- “si quiero saber Z, abre este archivo”

para que la uses siempre en este proyecto.
