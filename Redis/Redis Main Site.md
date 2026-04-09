
---

# Conceptos de Redis 


Redis  se usa como **almacén temporal rápido** para representar una cosa muy simple:

> “este dispositivo sigue vivo hasta tal momento”

La forma de representarlo es con una **key con TTL**.

---

## 2. Qué es una key

En Redis todo gira alrededor de claves.

Ejemplo:

```text
device:errcom:12345
```

Esa sería una key que representa el estado temporal del device `12345`.

---

## 3. Qué es TTL

TTL significa **Time To Live**.

Es el tiempo que una key puede vivir antes de borrarse sola.

Ejemplo:

- guardas `device:errcom:12345`
- le pones TTL de 54000 segundos
- cuando pasa ese tiempo, Redis la elimina automáticamente
    

En esta arquitectura, eso significa:

> si no ha llegado ningún mensaje nuevo antes de que expire la key, consideramos que el device ha dejado de comunicar

---

## 4. Qué significa “renovar la key”

Cada vez que llega un mensaje válido del device:

- se vuelve a escribir la key
- o se resetea su TTL

Eso equivale a decir:

> “el dispositivo sigue vivo, vuelve a contar desde cero”

Ejemplo conceptual:

```ts
redis.set("device:errcom:12345", valor, "EX", ttl)
```

Cada mensaje válido vuelve a ejecutar esto y mueve la expiración hacia adelante.

---

## 5. Qué es “expiración”

Cuando el TTL llega a 0:

- Redis borra la key
- y puede emitir un evento diciendo que esa key expiró
    

En tu caso:

- si expira `device:errcom:12345`
- significa que el device no ha enviado ningún mensaje válido dentro del tiempo esperado
- entonces toca generar `errorcomm`
    

---

## 6. Qué es `__keyevent@0__:expired`

Eso es un **canal especial de Redis** que publica eventos internos de expiración de keys.

### Traducido

- `keyevent` = evento sobre una key
- `@0` = base de datos Redis 0
- `expired` = tipo de evento: la key ha expirado
    

Entonces un worker puede suscribirse a ese canal y enterarse cuando Redis borra una key por expiración.

## Idea mental

Redis dice algo como:

> “ha expirado `device:errcom:12345`”

y el worker escucha eso y reacciona.

---

## 7. Qué hace el worker con esa expiración

Cuando detecta que expiró una key de errcom:

1. saca el `deviceId` del nombre de la key
2. verifica si el device sigue online o si no se recuperó ya
3. actualiza Mongo:
    
    - `device.status = "error"`
    - crea/actualiza documento en `ErrorComm`
        
4. emite evento `errorcomm activation` a `events-12830`
    

---

## 8. Por qué Redis encaja bien aquí

Porque este problema se parece mucho a un **heartbeat**.

Cada mensaje válido es como decir:

- “sigo vivo”
- “reinicia mi temporizador”
    

Redis con TTL hace esto muy bien:

- es rápido
- evita cálculos constantes
- evita recorrer todos los devices todo el rato
- convierte el problema en algo reactivo
    

---


### Heartbeat

Señal periódica de vida de un dispositivo.  
Aquí, cualquier mensaje válido salvo `link`.

### TTL

Tiempo que una key vive en Redis antes de desaparecer sola.

### Expiración

Momento en que Redis borra una key por haber vencido el TTL.

### Worker

Proceso especializado que escucha eventos y ejecuta una tarea automática.

### Keyspace notifications

Funcionalidad de Redis para emitir eventos cuando pasan cosas con keys, por ejemplo expiraciones.

### Polling

Revisar periódicamente en bucle si algo ha vencido.  
Es lo que hace el micro antiguo.

### Enfoque reactivo

En vez de revisar cada 10 minutos, esperas a que Redis te avise cuando ocurra algo.

---


    

---

