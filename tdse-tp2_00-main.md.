# Análisis de Ejecución y Configuración: STM32F103

El código fuente proporcionado compone la estructura básica de inicialización y ejecución para un microcontrolador de la familia STM32F1. A continuación, se detalla el flujo de ejecución y la evolución de las variables `SystemCoreClock` y `SysTick`.

## 1. Fase de Inicio en Ensamblador (Boot/Reset)
El microcontrolador comienza su ejecución en el archivo `startup_stm32f103rbtx.s`[cite: 3].

* **Vector de Interrupciones:** Se define la tabla de vectores (`g_pfnVectors`), donde se establece que tras un reinicio, el sistema debe saltar a la etiqueta `Reset_Handler`[cite: 3].
* **Reset_Handler:** Es el punto de entrada real[cite: 3].
  * Llama primero a la función externa `SystemInit`[cite: 3]. En esta función externa, la variable global **`SystemCoreClock`** se inicializa por primera vez con el valor del oscilador interno por defecto (típicamente HSI a 8 MHz).
  * Copia los datos inicializados de la memoria Flash a la memoria SRAM (sección `.data`) y llena con ceros la sección de variables no inicializadas (`.bss`)[cite: 3].
  * Ejecuta `bl main` para transferir el control a la función principal en C[cite: 3].
* **Evolución del SysTick:** En esta etapa inicial, el temporizador de hardware SysTick se encuentra apagado y deshabilitado.

## 2. Inicialización del Hardware y HAL
La ejecución entra en el archivo `main.c` dentro de la función `main()`[cite: 1].

* **HAL_Init():** La primera acción del código C es llamar a `HAL_Init()`[cite: 1]. Esta función reinicia todos los periféricos e inicializa la interfaz de la Flash y el Systick[cite: 1].
* **Evolución del SysTick:** `HAL_Init()` configura el hardware del SysTick basándose en el valor actual de `SystemCoreClock` (8 MHz) para generar una interrupción cada 1 milisegundo exacto. A partir de este momento, la variable interna del HAL que cuenta los milisegundos comienza a incrementar.

## 3. Configuración del Reloj del Sistema
Tras configurar el HAL, se llama a la función `SystemClock_Config()` en `main.c`[cite: 1].

* **Configuración del Oscilador y PLL:** La función configura el oscilador interno (HSI) y enciende el PLL (`RCC_PLL_ON`)[cite: 1]. Se utiliza el HSI dividido por 2 como fuente del PLL (`RCC_PLLSOURCE_HSI_DIV2`) y se multiplica por 16 (`RCC_PLL_MUL16`)[cite: 1]. Considerando un HSI base de 8 MHz, esto eleva la frecuencia teórica a 64 MHz.
* **Evolución de SystemCoreClock:** Al ejecutarse `HAL_RCC_ClockConfig()`, los relojes de los buses AHB y APB se actualizan[cite: 1]. Internamente, se actualiza la variable global **`SystemCoreClock`** a su valor final de trabajo de **64,000,000 Hz** (64 MHz).
* **Ajuste del SysTick:** Al cambiar la velocidad del reloj central a 64 MHz, `HAL_RCC_ClockConfig()` reconfigura el divisor del hardware del SysTick en base al nuevo `SystemCoreClock`, asegurando que siga disparándose de forma precisa cada 1 ms bajo la nueva frecuencia.

## 4. Bucle Principal y Manejo de Interrupciones
Una vez configurado el reloj, se inicializan los puertos GPIO (`MX_GPIO_Init`) y el puerto serial USART2 (`MX_USART2_UART_Init`)[cite: 1]. Finalmente, se ejecuta `app_init()` y el código entra al bucle infinito `while (1)`, donde ejecuta repetidamente `app_update()`[cite: 1].

* **Evolución continua del SysTick:** Mientras el procesador ejecuta el bucle principal en `main.c`[cite: 1], el temporizador de hardware funciona en paralelo. Cada 1 milisegundo, el hardware interrumpe la ejecución y salta al archivo `stm32f1xx_it.c`, específicamente a la función **`SysTick_Handler(void)`**[cite: 2].
* Dentro de este handler, se ejecuta **`HAL_IncTick()`**[cite: 2], lo cual incrementa en +1 la base de tiempo del sistema. 
* Inmediatamente después, se retorna el control al bucle `while (1)`[cite: 1] para que la aplicación (`app_update()`) continúe su ejecución con una base de tiempo actualizada.
