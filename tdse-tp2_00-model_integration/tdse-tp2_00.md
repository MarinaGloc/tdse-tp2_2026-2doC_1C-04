# TP2-00 - Diagramas de Estado y codificación en C

## Enfoque de trabajo

Este trabajo práctico implementa un sistema embebido como un conjunto de máquinas de estados finitos cooperativas. Cada máquina conserva su estado entre invocaciones y se actualiza rápidamente, sin esperas bloqueantes. La aplicación base separa el problema en tres tareas:

1. **Sensor:** lee el pulsador y convierte el nivel eléctrico en eventos semánticos.
2. **System:** recibe los eventos mediante una cola y decide la acción requerida.
3. **Actuator:** recibe la orden y modifica la salida asociada al LED.

El flujo funcional es:

```text
B1 USER -> task_sensor -> cola de eventos -> task_system -> task_actuator -> LD2
```

El `SysTick` genera la base de tiempo de 1 ms. `app_update()` consume los ticks pendientes y ejecuta, en orden, las funciones de actualización de Sensor, System y Actuator. Esto permite implementar el comportamiento de manera no bloqueante y facilita la extensión a varios sensores o actuadores.

## Criterios de codificación

- Representar estados y eventos mediante enumeraciones.
- Separar configuración constante (`cfg`) de datos variables (`dta`).
- Ejecutar una transición únicamente cuando se cumple su evento o guarda.
- Realizar las acciones de salida en el cuerpo de la transición.
- Comunicar las máquinas mediante interfaces y eventos, sin acoplarlas directamente al hardware de las otras tareas.
- Evitar demoras activas dentro de las máquinas de estados.
- Mantener el código dependiente de la placa concentrado en `board.h` y en la inicialización generada por STM32CubeMX.

## Resultado esperado del proyecto base

Al mantener presionado B1 USER, la máquina Sensor produce `EV_SYS_ACTIVE`; System ordena `EV_LED_ACTIVE` y Actuator enciende LD2. Al soltar el pulsador se propagan `EV_SYS_IDLE` y `EV_LED_IDLE`, apagando LD2.

> La comprobación eléctrica y temporal final debe realizarse sobre una placa NUCLEO-F103RB mediante STM32CubeIDE.
