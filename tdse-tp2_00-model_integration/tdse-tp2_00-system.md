# TP2-00 - Análisis de Task System

## Inicialización

Existe un único modo, `NORMAL`, por lo que `SYSTEM_DTA_QTY = 1` e `index` sólo toma el valor 0. `task_system_init()` inicializa la cola y establece:

- `g_task_system_mode = NORMAL`.
- `tick = 0 ms` por inicialización estática.
- `state = ST_SYS_IDLE`.
- `event = EV_SYS_IDLE`.
- `flag = false`.

## Máquina de estados

`task_system_update()` selecciona el modo y llama a `task_system_normal_statechart()`.

- Si la cola contiene un evento, éste se extrae, se guarda en `event` y se activa `flag`.
- En `ST_SYS_IDLE`, la combinación `flag == true` y `event == EV_SYS_ACTIVE` limpia la bandera, envía `EV_LED_ACTIVE` a `ID_LED_A` y cambia a `ST_SYS_ACTIVE`.
- En `ST_SYS_ACTIVE`, `EV_SYS_IDLE` limpia la bandera, envía `EV_LED_IDLE` y vuelve a `ST_SYS_IDLE`.
- Un estado inválido restaura los valores iniciales.

`tick` no interviene en las transiciones de esta versión y permanece en 0 ms.

## Cola de eventos

Al inicializar: `head = tail = count = 0` y cada `queue[i] = EMPTY`. `put_event_task_system()` escribe en `head`; `get_event_task_system()` lee desde `tail`. Ambos índices son circulares, con longitud 16.

Por el orden Sensor -> System, un evento generado por Sensor normalmente se consume en la misma ejecución de `app_update()`. Después del consumo, `head == tail`, `count == 0` y la celda leída vuelve a `EMPTY`.

## Comunicación con Actuator

`put_event_task_actuator(event, identifier)` selecciona directamente `task_actuator_dta_list[identifier]`, copia el evento y establece `flag = true`.

| Transición de System | identifier | event del actuador | flag |
|---|---|---|---|
| IDLE -> ACTIVE | `ID_LED_A` | `EV_LED_ACTIVE` | `true` hasta que Actuator lo consume |
| ACTIVE -> IDLE | `ID_LED_A` | `EV_LED_IDLE` | `true` hasta que Actuator lo consume |

Como Actuator se ejecuta después de System, la orden también puede aplicarse dentro de la misma activación de 1 ms.
