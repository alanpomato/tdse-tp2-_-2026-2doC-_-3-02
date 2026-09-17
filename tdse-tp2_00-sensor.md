# Análisis de funcionamiento: Sistema de Tareas y Sensores

Este documento describe el funcionamiento y la evolución de las variables principales del código fuente compuesto por los archivos `task_sensor_attribute.h`, `task_system_attribute.h`, `task_sensor.c` y `task_system_interface.c`.

## 1. Evolución de las variables del Sensor

Las variables relacionadas al sensor pertenecen a la estructura `task_sensor_dta_list` y se comportan de la siguiente manera a lo largo del tiempo:

*   **`index`**:
    *   **Inicio (`task_sensor_init`)**: Se inicializa en `0` y se incrementa en un bucle `for` hasta llegar a `SENSOR_DTA_QTY - 1` (que en este caso es 1, por lo que solo toma el valor 0).
    *   **Loop principal (`task_sensor_update`)**: En cada iteración del loop de actualización de tareas, el `index` vuelve a iterar desde `0` hasta `SENSOR_DTA_QTY - 1` para recorrer todos los sensores configurados.
*   **`task_sensor_dta_list[index].tick`**:
    *   **Unidad de medida**: Milisegundos (ms), deducido a partir de los mensajes de log de inicialización (`Tick [mS]`).
    *   **Evolución**: Sorprendentemente, en este código específico, la variable `tick` **no se inicializa explícitamente** dentro de `task_sensor_init()`. Durante el loop principal, tampoco se incrementa ni se modifica, a menos que el estado del sensor caiga en el caso por defecto (`default`) de la máquina de estados, donde se le asigna el valor `DEL_BTN_MIN` (0).
*   **`task_sensor_dta_list[index].state`**:
    *   **Inicio**: Se inicializa forzosamente en el estado `ST_BTN_IDLE`.
    *   **Loop principal**: Cambia entre `ST_BTN_IDLE` (reposo) y `ST_BTN_ACTIVE` (presionado) dependiendo de los eventos físicos registrados por el botón.
*   **`task_sensor_dta_list[index].event`**:
    *   **Inicio**: Se inicializa asumiendo que el botón no está presionado, asignándose `EV_BTN_UP`.
    *   **Loop principal**: En cada ejecución de `task_sensor_statechart`, su valor se actualiza dinámicamente a `EV_BTN_DOWN` si la lectura del hardware (`HAL_GPIO_ReadPin`) indica que el botón está presionado, o a `EV_BTN_UP` si no lo está.

## 2. Comportamiento de `task_sensor_statechart(uint32_t index)`

La función `task_sensor_statechart` es el núcleo lógico del sensor y funciona como una máquina de estados finitos (FSM):

1.  **Lectura del Hardware**: Primero, lee el estado físico del pin configurado para el sensor mediante la función `HAL_GPIO_ReadPin`.
2.  **Actualización de Eventos**: Compara la lectura con el estado de "presionado" (`pressed`) definido en la configuración (`task_sensor_cfg_list`). Si coinciden, actualiza el evento actual a `EV_BTN_DOWN`; en caso contrario, a `EV_BTN_UP`.
3.  **Máquina de Estados (Switch)**: Evalúa el estado actual del sensor (`p_task_sensor_dta->state`):
    *   **Caso `ST_BTN_IDLE`**: Si el evento es `EV_BTN_DOWN` (alguien acaba de presionar el botón), la función envía una señal a la cola del sistema llamando a `put_event_task_system(signal_down)` y transita al estado `ST_BTN_ACTIVE`.
    *   **Caso `ST_BTN_ACTIVE`**: Si el evento es `EV_BTN_UP` (alguien soltó el botón), envía la señal contraria a la cola del sistema mediante `put_event_task_system(signal_up)` y vuelve al estado `ST_BTN_IDLE`.
    *   **Caso `default`**: Funciona como un mecanismo de seguridad (fallback). Si la memoria se corrompe y cae en un estado no reconocido, resetea el contador de `tick` a `DEL_BTN_MIN`, el estado a `ST_BTN_IDLE` y el evento a `EV_BTN_UP`.

## 3. Evolución de la cola `event_task_system_queue`

La cola (buffer circular) gestiona los eventos que el sensor le envía al sistema central (definido en `task_system_interface.c`). Las variables evolucionan así:

*   **Variables en el Inicio (`init_event_task_system()`)**:
    *   `event_task_system_queue.head`: Se inicializa en `0`.
    *   `event_task_system_queue.tail`: Se inicializa en `0`.
    *   `event_task_system_queue.count`: Se inicializa en `0`.
    *   `event_task_system_queue.queue[i]`: Se llena completamente con el macro `EMPTY` (`255ul`) desde la posición `i=0` hasta `i=15` (`QUEUE_LENGTH - 1`).

*   **En el Loop Principal (`task_sensor_update` -> `put_event_task_system()`)**:
    * Cuando el usuario interactúa con el botón (presiona o suelta), se ejecuta la función `put_event_task_system(event)`.
    *   **`count`**: Se incrementa en `1` cada vez que ingresa un nuevo evento a la cola.
    *   **`queue[i]`**: En la posición indicada por `head`, se sobreescribe el valor `EMPTY` por el evento generado (ej. `EV_SYS_ACTIVE` o `EV_SYS_IDLE`).
    *   **`head`**: Se incrementa en `1` apuntando al siguiente espacio disponible. Si `head` alcanza el tamaño máximo (`QUEUE_LENGTH` = 16), vuelve a `0` (comportamiento de buffer circular).
    
*   **Al procesar eventos (`get_event_task_system()`)**:
    * Cuando el sistema consume los eventos de la cola, llama a `get_event_task_system()`.
    *   **`count`**: Disminuye en `1`.
    *   **`queue[i]`**: Se extrae el evento de la posición `tail` y dicha posición se vuelve a marcar con el valor `EMPTY`.
    *   **`tail`**: Se incrementa en `1` y, de forma similar al `head`, si alcanza `QUEUE_LENGTH`, se reinicia a `0`.
