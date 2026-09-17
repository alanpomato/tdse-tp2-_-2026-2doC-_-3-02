# Análisis del Sistema de Tareas y Actuadores

El código fuente proporcionado implementa una arquitectura basada en eventos y máquinas de estados finitos (FSM) para un sistema embebido. La aplicación se divide en un sistema principal que procesa eventos a través de una cola circular no bloqueante y un actuador (como un LED) que reacciona a las órdenes de dicho sistema.

## Evolución de las variables de `task_system_dta_list`

La estructura `task_system_dta_list` almacena el estado de la tarea del sistema. Su evolución es la siguiente:

### Durante `task_system_init()`
* **`index`**: Itera desde `0` hasta `SYSTEM_DTA_QTY - 1` (donde `SYSTEM_DTA_QTY` vale `1`, correspondiente a `MODE_QTY`).
* **`task_system_dta_list[index].tick`**: No se inicializa explícitamente en el bucle de esta función, pero se asume su medición en milisegundos (mS) según el registro del sistema.
* **`task_system_dta_list[index].state`**: Se inicializa en `ST_SYS_IDLE`.
* **`task_system_dta_list[index].event`**: Se inicializa en `EV_SYS_IDLE`.
* **`task_system_dta_list[index].flag`**: Se inicializa en `false`.

### En sucesivas ejecuciones de `task_system_update()`
* Si hay eventos en la cola, `.flag` pasa a ser `true` y `.event` toma el valor del evento extraído de la cola.
* Dependiendo del estado actual y del evento, el `.state` alternará entre `ST_SYS_IDLE` y `ST_SYS_ACTIVE`.
* Tras procesar una transición válida, el `.flag` se restablece a `false`.
* Si el sistema cae en un estado por defecto (`default`), `.tick` se fuerza a `DEL_SYS_MIN` (0 mS), y se reinician `.state`, `.event` y `.flag` a sus valores inactivos.

---

## Comportamiento de la función `task_system_normal_statechart(void)`

> **Nota:** El código fuente define `void task_system_normal_statechart(void)` en lugar de recibir un `index` como parámetro.

Esta función implementa la máquina de estados del sistema:
1. Verifica si hay eventos pendientes utilizando `any_event_task_system()`.
2. Si existen, levanta la bandera `flag = true` y actualiza la variable `event` leyendo de la cola.
3. Evalúa el estado actual (`p_task_system_dta->state`):
   * **En `ST_SYS_IDLE`**: Si `flag` es verdadero y el evento es `EV_SYS_ACTIVE`, baja la bandera, envía el evento `EV_LED_ACTIVE` al actuador `ID_LED_A` mediante `put_event_task_actuator()`, y transiciona al estado `ST_SYS_ACTIVE`.
   * **En `ST_SYS_ACTIVE`**: Si `flag` es verdadero y el evento es `EV_SYS_IDLE`, baja la bandera, envía el evento `EV_LED_IDLE` al actuador `ID_LED_A`, y retorna al estado `ST_SYS_IDLE`.

---

## Evolución de la cola de eventos `event_task_system_queue`

Esta estructura maneja la recepción asíncrona de eventos para el sistema.

### Durante `task_system_init()` (vía `init_event_task_system()`)
* **`event_task_system_queue.head`, `.tail`, y `.count`**: Se inicializan en `0`.
* **`i`**: Itera desde `0` hasta `QUEUE_LENGTH - 1` (15).
* **`event_task_system_queue.queue[i]`**: Se inicializa con el valor `EMPTY` (255) en todas sus posiciones.

### En sucesivas ejecuciones de `task_system_update()`
* Al detectarse eventos, la función de estado llama a `get_event_task_system()`.
* **`.count`**: Decrece en `1`.
* El evento en la posición `queue[tail]` es leído y entregado al sistema.
* La posición leída `queue[tail]` se sobrescribe con `EMPTY`.
* **`.tail`**: Se incrementa en `1`; si alcanza `QUEUE_LENGTH`, vuelve a `0` (comportamiento circular).
> **Nota:** `.head` y `.count` solo se incrementan de forma externa cuando otras partes del código (como interrupciones u otras tareas) llaman a `put_event_task_system()`.

---

## Evolución de las variables de `task_actuator_dta_list`

Esta estructura almacena el estado de los actuadores del sistema.

### Durante `task_system_init()`
* El código proporcionado no modifica explícitamente las variables del actuador durante la inicialización del sistema.

### En sucesivas ejecuciones de `task_system_update()`
* Cuando la máquina de estados realiza una transición válida, llama a `put_event_task_actuator(event, ID_LED_A)`.
* **`identifier`**: Toma el valor enviado en la función, en este caso `ID_LED_A`.
* **`task_actuator_dta_list[identifier].event`**: Adopta el valor del evento enviado (`EV_LED_ACTIVE` o `EV_LED_IDLE`).
* **`task_actuator_dta_list[identifier].flag`**: Se impone en `true` forzosamente, indicándole a la tarea del actuador (no incluida en el update del sistema) que tiene un nuevo evento que procesar.
