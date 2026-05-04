
microservicio de **detección y gestión de error de comunicación** (`ErrcomMicro`). Su trabajo es decidir cuándo un dispositivo ha dejado de comunicar, mantener el estado temporal de esa situación en BD y disparar eventos de activación/desactivación de la alarm

**Responsabilidad principal**  
Trabaja sobre dos ejes:

1. **Reactivo**: cuando entra un `status` o cambia una configuración, recalcula la siguiente ventana esperada de comunicación.
2. **Periódico**: cada cierto tiempo revisa qué dispositivos deberían haber comunicado ya y, si no lo han hecho, activa error de comunicación

## Flujo del microservicio 

[[AKO 44 - ErrorComm workFlow.canvas]]


