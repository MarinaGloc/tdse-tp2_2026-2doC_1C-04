# TP2-00 - Análisis de arranque y programa principal

## Secuencia desde el reset

1. El Cortex-M3 toma de la tabla de vectores el valor inicial del puntero de pila y la dirección de `Reset_Handler`.
2. `Reset_Handler` llama a `SystemInit()`, copia `.data` desde Flash a RAM, limpia `.bss`, ejecuta los constructores de la biblioteca C y llama a `main()`.
3. `main()` ejecuta `HAL_Init()`, configura el reloj mediante `SystemClock_Config()`, inicializa GPIO y USART2, y llama a `app_init()`.
4. Finalmente entra en `while (1)`, donde invoca continuamente `app_update()`.

`stm32f1xx_it.c` contiene los manejadores de excepciones. El relevante para la aplicación es `SysTick_Handler()`: incrementa el tick de HAL con `HAL_IncTick()` y luego llama a `HAL_SYSTICK_IRQHandler()`. Este último termina ejecutando `HAL_SYSTICK_Callback()`, definido por la aplicación.

## Evolución de `SystemCoreClock`

- Después del reset el STM32F103 usa HSI, cuyo valor nominal es **8 MHz**.
- `SystemInit()` deja el sistema en su configuración inicial segura y `SystemCoreClock` representa inicialmente 8 MHz.
- `SystemClock_Config()` mantiene HSI, divide sus 8 MHz por 2 como entrada del PLL y aplica un multiplicador por 16.
- Tras aplicar esa configuración, `SystemCoreClock` pasa a **64 MHz** y permanece con ese valor durante el lazo principal, salvo que el programa vuelva a modificar el RCC. HCLK queda en 64 MHz, PCLK1 en 32 MHz y PCLK2 en 64 MHz.

## Evolución de `SysTick`

`SysTick` es un periférico descendente de 24 bits, no una variable escalar. Sus registros principales son `CTRL`, `LOAD` y `VAL`.

- Tras el reset está detenido.
- `HAL_Init()` configura una interrupción periódica de 1 ms usando el reloj disponible en ese momento.
- Cuando `SystemClock_Config()` cambia HCLK, HAL vuelve a configurar la base de tiempo para conservar el período de 1 ms.
- Con `SystemCoreClock = 64 MHz`, el contador recorre aproximadamente 64 000 ciclos por milisegundo (`LOAD = 63999` cuando la fuente es HCLK).
- Al llegar a cero recarga `LOAD`, produce `SysTick_Handler()` y continúa contando.

Por lo tanto, en el `while (1)` el valor instantáneo de `SysTick->VAL` cambia continuamente de `LOAD` hacia cero, mientras el tick de HAL y `g_app_tick_cnt` avanzan una vez por milisegundo.

## Verificación sugerida

Colocar puntos de interrupción en `Reset_Handler`, después de `HAL_Init()`, después de `SystemClock_Config()` y dentro de `while (1)`. Observar `SystemCoreClock`, `SysTick->CTRL`, `SysTick->LOAD`, `SysTick->VAL` y `uwTick`.
