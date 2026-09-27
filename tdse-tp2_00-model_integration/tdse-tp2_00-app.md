# TP2-00 - Análisis de la aplicación

## Organización

`app.c` implementa un planificador cooperativo activado por tiempo. `task_cfg_list` contiene tres pares de funciones `init/update`, ejecutados en este orden: Sensor, System y Actuator.

`app_it.c` define `g_app_tick_cnt`. `HAL_SYSTICK_Callback()` lo incrementa cada 1 ms. `app_update()` extrae un tick de forma atómica y, por cada tick pendiente, ejecuta las tres tareas. Si durante su ejecución llegan nuevos ticks, también los procesa antes de regresar.

El contador DWT mide los ciclos empleados por cada `task_update()`. `cycle_counter_get_time_us()` convierte esa medición a microsegundos usando `SystemCoreClock`.

## Evolución de variables

En `app_init()`:

- `g_app_cnt = 0`.
- `g_app_tick_cnt = 0` al ejecutar `app_it_init()`.
- Para cada índice `0..2`: `NOE = 0`, `LET = 0 us`, `BCET = 1000 us` y `WCET = 0 us`.

En cada período de 1 ms procesado por `app_update()`:

- `g_app_tick_cnt` disminuye en uno; la interrupción SysTick puede incrementarlo nuevamente.
- `g_app_cnt` aumenta en uno.
- `index` recorre `0` (Sensor), `1` (System) y `2` (Actuator).
- `NOE` aumenta en uno para la tarea correspondiente.
- `LET` toma el tiempo de la última ejecución, en microsegundos.
- `BCET` conserva el menor `LET` observado.
- `WCET` conserva el mayor `LET` observado.
- `g_app_runtime_us` se reinicia y luego acumula los tres valores `LET`; representa el tiempo total de cómputo de esa activación, en microsegundos.

## Registro de valores medidos

Los valores temporales dependen de la placa, la optimización, la instrumentación y el uso del depurador; no pueden deducirse exactamente mediante análisis estático.

| Tarea (`index`) | NOE | LET [us] | BCET [us] | WCET [us] |
|---:|---:|---:|---:|---:|
| 0 - Sensor | medir | medir | medir | medir |
| 1 - System | medir | medir | medir | medir |
| 2 - Actuator | medir | medir | medir | medir |

`g_app_runtime_us`: **medir en la placa** [us].

## Impacto de `LOGGER_INFO()`

Las llamadas realizadas durante la inicialización aumentan fuertemente su duración. Si se incorporan dentro de una región medida por DWT, elevan `LET`, `WCET` y, por suma, `g_app_runtime_us`. Con semihosting el efecto puede ser especialmente grande porque la CPU se detiene para que el depurador atienda la salida. Por eso una medición temporal representativa debe hacerse sin logging dentro del camino medido, o debe documentar expresamente que la instrumentación está incluida.
