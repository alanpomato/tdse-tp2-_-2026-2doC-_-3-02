En esta actividad se adicionaron 3 botones externos al botón B1 de la placa, completando un total de 4 sensores.
Los botones se conectaron a Pines en el rango D0...D15, garantizando la disponibilidad de los mismos mediante el `.ioc` y su conexión física a través del esquemático de la placa.
La lógica se implementó en task_sensor.c, teniendo que modificar filas en cfg_list[] y definiendo los valores en task_sensor_attribute.h y board.h (a partir de la HAL configurada por .ioc). 

