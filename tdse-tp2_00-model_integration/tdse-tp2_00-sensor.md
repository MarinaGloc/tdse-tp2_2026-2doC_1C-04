# TP2-00 - Análisis de Task Sensor

## Configuración e inicialización

El proyecto contiene un único sensor, `ID_BTN_A`, conectado al pulsador B1 USER. La configuración guarda puerto, pin, nivel de pulsación, tiempo máximo y los eventos destinados a System.

`task_sensor_init()` recorre un solo índice (`index = 0`) e inicializa:

- `tick = 0 ms`, debido a la inicialización estática del arreglo global.
- `state = ST_BTN_IDLE`.
- `event = EV_BTN_UP`.

## Máquina de estados

En cada `task_sensor_update()`, `index` toma el valor 0 y se llama a `task_sensor_statechart(0)`:

1. Se lee el GPIO.
2. Si coincide con `BTN_A_PRESSED`, `event = EV_BTN_DOWN`; de lo contrario, `event = EV_BTN_UP`.
3. En `ST_BTN_IDLE`, `EV_BTN_DOWN` encola `EV_SYS_ACTIVE` y cambia a `ST_BTN_ACTIVE`.
4. En `ST_BTN_ACTIVE`, `EV_BTN_UP` encola `EV_SYS_IDLE` y vuelve a `ST_BTN_IDLE`.

Mientras el pulsador permanece estable no se generan eventos repetidos, porque sólo se emite uno al cambiar entre los dos estados. En esta versión `tick` no participa de las transiciones y permanece en 0 ms, salvo recuperación desde un estado inválido, que también lo fija en cero.

## Evolución de la cola de System

`init_event_task_system()` establece `head = 0`, `tail = 0`, `count = 0` y llena las 16 posiciones con `EMPTY` (255).

Al detectar una pulsación, `put_event_task_system(EV_SYS_ACTIVE)` escribe en `queue[head]`, incrementa `head` con retorno circular al llegar a 16 e incrementa `count`. Al detectar la liberación ocurre lo mismo con `EV_SYS_IDLE`.

En la misma activación de la aplicación, System se ejecuta después de Sensor: detecta que `head != tail`, obtiene el evento, incrementa `tail`, decrementa `count` y vuelve a escribir `EMPTY` en la posición consumida. En funcionamiento normal la cola suele quedar vacía al terminar cada `app_update()`.

| Momento | head | tail | count | Contenido relevante |
|---|---:|---:|---:|---|
| Después de inicializar | 0 | 0 | 0 | Todas las posiciones = 255 |
| Sensor encola primer evento | 1 | 0 | 1 | `queue[0] = EV_SYS_ACTIVE` |
| System lo consume | 1 | 1 | 0 | `queue[0] = 255` |

La implementación no comprueba desbordamiento; si se produjeran más de 16 escrituras sin consumo, `count` dejaría de representar una cola válida.
