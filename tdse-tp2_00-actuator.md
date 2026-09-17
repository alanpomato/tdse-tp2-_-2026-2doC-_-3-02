# Análisis del Funcionamiento: Task Actuator

El siguiente documento detalla el análisis del funcionamiento del código fuente que implementa una máquina de estados finitos (Statechart) no bloqueante para el control de un actuador (LED). Se analizan los archivos: `task_actuator_attribute.h`, `task_actuator.c` y `task_actuator_interface.c`.

## 1. Evolución de variables: `task_actuator_init()` y `task_actuator_update()`

Al iniciar la aplicación, se ejecuta por única vez `task_actuator_init()`. Luego, el sistema invoca de forma periódica a `task_actuator_update()` dentro del loop principal.

La evolución de las variables dentro de la estructura `task_actuator_dta_list` es la siguiente:

* **`index`**: 
  * En `task_actuator_init()`, se utiliza en un bucle `for` para iterar sobre la cantidad de actuadores configurados (`ACTUATOR_DTA_QTY`). Como solo hay un actuador definido (`ID_LED_A`), `index` toma el valor de `0`. 
  * En `task_actuator_update()`, vuelve a iterar de la misma manera para procesar la máquina de estados de cada actuador.
* **`task_actuator_dta_list[index].tick`**: 
  * Su unidad de medida son los **milisegundos (ms)** (evidenciado por las constantes de tiempo como `DEL_LED_MAX = 500ul` y las funciones de HAL). 
  * No se inicializa explícitamente en el inicio ni cambia en los estados funcionales normales. Solo adquiere el valor `DEL_LED_MIN` (0) si la máquina de estados cae en el caso de error (`default`).
* **`task_actuator_dta_list[index].state`**: 
  * Se inicializa forzosamente en `ST_LED_IDLE` durante `task_actuator_init()`.
* **`task_actuator_dta_list[index].event`**: 
  * Se configura inicialmente con el evento de reposo `EV_LED_IDLE`.
* **`task_actuator_dta_list[index].flag`**: 
  * Se inicializa en `false`, indicando que no hay eventos nuevos pendientes por procesar.

---

## 2. Comportamiento de la función `task_actuator_statechart(uint32_t index)`

Esta función es el núcleo del módulo. Es llamada continuamente por `task_actuator_update()` y evalúa el estado actual (`state`) mediante una estructura `switch`:

* **Caso `ST_LED_IDLE`**: 
  Si la variable `flag` es `true` y el evento recibido es `EV_LED_ACTIVE`:
  1. Baja la bandera (`flag = false`).
  2. Escribe en el puerto GPIO correspondiente para encender el actuador (`led_on`).
  3. Transiciona al nuevo estado cambiando `state` a `ST_LED_ACTIVE`.

* **Caso `ST_LED_ACTIVE`**: 
  Si la variable `flag` es `true` y el evento recibido es `EV_LED_IDLE`:
  1. Baja la bandera (`flag = false`).
  2. Escribe en el puerto GPIO para apagar el actuador (`led_off`).
  3. Transiciona al estado de reposo cambiando `state` a `ST_LED_IDLE`.

* **Caso `default`**: 
  Actúa como un mecanismo de seguridad (fail-safe) por si el estado se corrompe. Restablece el actuador a condiciones iniciales:
  1. `tick = DEL_LED_MIN`
  2. `state = ST_LED_IDLE`
  3. `event = EV_LED_IDLE`
  4. `flag = false`

---

## 3. Interfaz e Inyección de Eventos

El archivo `task_actuator_interface.c` expone la función `put_event_task_actuator(task_actuator_ev_t event, task_actuator_id_t identifier)`, la cual permite a otras tareas interactuar con el actuador.

Su comportamiento y la evolución de las variables asociadas es:

* **`identifier`**: 
  Se recibe como parámetro y funciona como el índice (ej. `0`) para apuntar al actuador correcto dentro del arreglo global `task_actuator_dta_list`.
* **`task_actuator_dta_list[identifier].event`**: 
  Se sobrescribe inmediatamente con el nuevo evento solicitado por la aplicación (`EV_LED_ACTIVE` o `EV_LED_IDLE`).
* **`task_actuator_dta_list[identifier].flag`**: 
  Se fuerza a `true`, notificando que hay un evento fresco listo para ser procesado.

**Ciclo completo de ejecución:**
Cuando otra tarea llama a `put_event_task_actuator()`, se actualiza el `event` y el `flag` pasa a `true`. En la siguiente iteración del loop principal, `task_actuator_update()` llamará a `task_actuator_statechart()`. La máquina de estados leerá el `flag` en `true`, ejecutará la acción sobre el hardware (encender/apagar el LED), cambiará de estado y volverá a poner el `flag` en `false` para indicar que el evento fue consumido correctamente.
