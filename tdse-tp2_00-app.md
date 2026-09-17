# Análisis de Variables del Sistema ETS (Event-Triggered Systems)

A continuación se detalla el comportamiento de las variables desde el inicio del programa en `app_init()` y durante el bucle principal en `app_update()`:

| Variable | Unidad | Evolución y Comportamiento |
| :--- | :--- | :--- |
| `g_app_tick_cnt` | Numeral | Se inicializa en `0` en `app_it_init()`[cite: 2]. Se incrementa de forma asíncrona cada vez que se ejecuta `HAL_SYSTICK_Callback()` mediante una interrupción de hardware[cite: 2]. En el flujo principal de `app_update()`, si esta variable es mayor a `0`, se decrementa para indicar que se debe procesar un nuevo ciclo (tick)[cite: 1]. |
| `g_app_runtime_us` | Microsegundos | Es una variable acumuladora que se reinicia a `0` al comienzo de cada ciclo de actualización dentro de `app_update()`[cite: 1]. Posteriormente, suma el tiempo de ejecución (`LET`) de cada una de las tareas que se ejecutan en ese ciclo[cite: 1]. |
| `index` | Numeral | Es la variable local utilizada como índice de control en los bucles `for` de `app_init()` y `app_update()`[cite: 1]. Inicia en `0` y se incrementa de a `1` hasta ser igual a `TASK_QTY - 1` (iterando sobre las 3 tareas definidas)[cite: 1]. |
| `task_dta_list[index].NOE` | Numeral | Representa el número de ejecuciones de la tarea[cite: 1]. Comienza en `0` al configurarse en `app_init()` y aumenta en `1` durante cada ejecución de la tarea correspondiente dentro de `app_update()`[cite: 1]. |
| `task_dta_list[index].LET` | Microsegundos | Representa el último tiempo de ejecución de la tarea[cite: 1]. Se inicializa en `0` en `app_init()` y se sobreescribe en cada ciclo de `app_update()` con el tiempo exacto que tomó ejecutar la tarea, medido por `cycle_counter_get_time_us()`[cite: 1, 6]. |
| `task_dta_list[index].BCET` | Microsegundos | Representa el mejor tiempo de ejecución registrado[cite: 1]. Comienza con un valor inicial de `1000` (`TASK_X_BCET_INI`) en `app_init()`[cite: 1]. En `app_update()`, adopta el valor de `LET` únicamente si el nuevo tiempo de ejecución es estrictamente menor al `BCET` almacenado actualmente[cite: 1]. |
| `task_dta_list[index].WCET` | Microsegundos | Representa el peor tiempo de ejecución registrado[cite: 1]. Inicia en `0` (`TASK_X_WCET_INI`) en `app_init()`[cite: 1]. Durante `app_update()`, adopta el valor de `LET` si el nuevo tiempo de ejecución es estrictamente mayor al `WCET` histórico[cite: 1]. |

---

## Impacto de utilizar `LOGGER_INFO()`

La macro `LOGGER_INFO()` funciona deshabilitando las interrupciones del sistema, procesando una cadena de texto mediante `snprintf`, llamando a una función de impresión (`logger_log_print_`) y finalmente rehabilitando las interrupciones[cite: 4]. Si la impresión utiliza semihosting, este proceso de entrada/salida es significativamente lento[cite: 3, 4].

Si se invoca `LOGGER_INFO()` dentro de la función de actualización de una tarea, el tiempo requerido para el formateo e impresión quedará comprendido entre el reinicio del contador de ciclos y la medición de tiempo efectuada por `cycle_counter_get_time_us()`[cite: 1, 6]. 

**Consecuencias directas:**

* **Aumento del WCET:** El valor de `task_dta_list[index].WCET` de esa tarea aumentará drásticamente, ya que el prolongado tiempo de ejecución (`LET`) superará con seguridad los tiempos medidos en iteraciones anteriores[cite: 1].
* **Aumento del Runtime:** El valor de `g_app_runtime_us` también sufrirá un incremento notable durante ese ciclo, puesto que debe sumar directamente este nuevo `LET` prolongado[cite: 1].





-----------

Luego de varias ejecuciones de app_update() , leer y almacenar los valores de task_dta_list [index] ( indicar unidad de medida), de cada tarea en : …\ tdse_workspace \ tdse-tp2_00-model_integration \ tdse-tp2_00-app.md .

<img width="669" height="539" alt="image" src="https://github.com/user-attachments/assets/4830778b-42a9-4af2-a7bf-f0611b7caed2" />
