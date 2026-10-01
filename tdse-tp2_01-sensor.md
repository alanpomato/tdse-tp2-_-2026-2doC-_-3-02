En esta actividad se procedió a modificar la lógica del Botón B1 color azul de la placa, implementada en task_sensor.c.
La lógica anterior no contemplaba el anti-rebote, por lo que se agregaron los estados correspondientes:


<img width="858" height="379" alt="image" src="https://github.com/user-attachments/assets/2f66616f-65bb-41f4-844f-c5e3fa088b84" />


Al detectar un cambio en el pin se aguardan 50 ticks y si se sostiene el cambio se modifica el estado del sensor y se emite la señal al sistema.

El anti-rebote dura DEL_BTN_MAX = 50ms.


<img width="622" height="454" alt="image" src="https://github.com/user-attachments/assets/08b39d2b-a327-4d66-9b20-9c00c04f8999" />


<img width="607" height="446" alt="image" src="https://github.com/user-attachments/assets/65487379-19e2-4d61-b014-bcf5ee4c1434" />
