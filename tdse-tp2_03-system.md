En esta actividad se implementó la lógica del sistema siguiendo la máquina de estados de la imagen:

<img width="685" height="496" alt="task_system" src="https://github.com/user-attachments/assets/f659320a-ed11-4cba-b01e-5c51c8ba76b1" />

<img width="852" height="420" alt="image" src="https://github.com/user-attachments/assets/b075a513-91a6-4a68-9260-97ca1f8436f9" />

Los eventos son EV_SYS_CAMERA, EV_SYS_BUTTON y EV_SYS_SENSOR_COIL, que para poder probar el sistema en IDE se debieron asociar a la señal emitida por los sensores.
Adicionalmente se colocó EV_SYS_IDLE que existe solamente como valor de relleno que usan los sensores al soltar (signal_down())

El sistema además posee una cola de 16 lugares ubicada en task_system_interface.c, en donde se almacenan las señales de los sensores y luego se van consumiendo por el sistema.


