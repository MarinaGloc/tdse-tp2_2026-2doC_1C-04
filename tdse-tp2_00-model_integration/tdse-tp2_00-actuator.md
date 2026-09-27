# TP2-00 - Análisis de Task Actuator

## Configuración e inicialización

Existe un único actuador, `ID_LED_A`, asociado a LD2. Para la NUCLEO-F103RB el LED es activo en nivel alto: `LED_A_ON = GPIO_PIN_SET` y `LED_A_OFF = GPIO_PIN_RESET`.

`task_actuator_init()` recorre solamente `index = 0` y establece:

- `tick = 0 ms`, por inicialización estática.
- `state = ST_LED_IDLE`.
- `event = EV_LED_IDLE`.
- `flag = false`.
- salida física en `LED_A_OFF`.

## Interfaz de eventos

`put_event_task_actuator(event, identifier)` accede al elemento indicado, guarda `event` y establece `flag = true`. Para este proyecto `identifier` siempre debe ser `ID_LED_A` (valor 0).

## Máquina de estados

`task_actuator_update()` invoca `task_actuator_statechart(0)`:

- En `ST_LED_IDLE`, si `flag == true` y `event == EV_LED_ACTIVE`, limpia `flag`, escribe el nivel de encendido y cambia a `ST_LED_ACTIVE`.
- En `ST_LED_ACTIVE`, si `flag == true` y `event == EV_LED_IDLE`, limpia `flag`, escribe el nivel de apagado y vuelve a `ST_LED_IDLE`.
- Ante un estado inválido recupera `tick = 0 ms`, estado y evento IDLE, y `flag = false`.

`tick` no se usa para temporización en esta implementación y permanece en 0 ms.

## Evolución típica

| Situación | event | flag antes de actualizar | state después de actualizar | LD2 |
|---|---|---:|---|---|
| Inicialización | `EV_LED_IDLE` | false | `ST_LED_IDLE` | apagado |
| System ordena encender | `EV_LED_ACTIVE` | true | `ST_LED_ACTIVE` | encendido |
| Pulsador continúa presionado | `EV_LED_ACTIVE` | false | `ST_LED_ACTIVE` | encendido |
| System ordena apagar | `EV_LED_IDLE` | true | `ST_LED_IDLE` | apagado |

La comprobación final debe hacerse sobre la placa: B1 presionado debe mantener LD2 encendido y B1 liberado debe mantenerlo apagado.
