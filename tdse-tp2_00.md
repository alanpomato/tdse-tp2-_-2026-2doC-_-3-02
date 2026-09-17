# Trabajo Práctico: Codificación en C de Diagramas de Estado

Para codificar un diagrama de estados en C, el método más estructurado y universal es implementar una **Máquina de Estados Finitos (FSM)** utilizando enumeraciones (`enum`) y una estructura de control `switch-case`. 

A continuación, se detalla el patrón estándar paso a paso.

## 1. Definir los Estados y los Eventos
El primer paso es mirar el diagrama y listar todos los estados y eventos (transiciones). En C, esto se hace utilizando `enum` para que el código sea legible y fácil de depurar.

*   **Estados:** Representan la situación actual del sistema (ej. Apagado, Encendido, Esperando).
*   **Eventos:** Representan las entradas o acciones que provocan que el sistema salte de un estado a otro (ej. Botón presionado, Temporizador agotado).

## 2. Implementación de Referencia
El esqueleto principal consta de una variable que almacena el estado actual y una función que recibe los eventos y decide la transición.

Ejemplo: **Molinete de Subte**.
*   **Estados:** `BLOQUEADO` y `DESBLOQUEADO`.
*   **Eventos:** `INSERTAR_MONEDA` y `EMPUJAR_BARRERA`.

### `main.c`

```c
#include <stdio.h>

// 1. Definimos todos los estados posibles
typedef enum {
    ESTADO_BLOQUEADO,
    ESTADO_DESBLOQUEADO
} EstadoMolinete;

// 2. Definimos todos los eventos que pueden ocurrir
typedef enum {
    EVENTO_INSERTAR_MONEDA,
    EVENTO_EMPUJAR_BARRERA
} EventoMolinete;

// 3. Variable para rastrear en qué estado estamos (inicia bloqueado)
EstadoMolinete estadoActual = ESTADO_BLOQUEADO;

// 4. La función central de la máquina de estados
void procesarEvento(EventoMolinete evento) {
    switch (estadoActual) {
        
        case ESTADO_BLOQUEADO:
            if (evento == EVENTO_INSERTAR_MONEDA) {
                printf("Moneda aceptada. Desbloqueando...\n");
                estadoActual = ESTADO_DESBLOQUEADO; // Transición de estado
            } else if (evento == EVENTO_EMPUJAR_BARRERA) {
                printf("No podes pasar. El molinete esta bloqueado.\n");
                // El estado no cambia
            }
            break;

        case ESTADO_DESBLOQUEADO:
            if (evento == EVENTO_EMPUJAR_BARRERA) {
                printf("Persona paso. Bloqueando molinete...\n");
                estadoActual = ESTADO_BLOQUEADO; // Transición de estado
            } else if (evento == EVENTO_INSERTAR_MONEDA) {
                printf("El molinete ya esta desbloqueado. Moneda rechazada.\n");
                // El estado no cambia
            }
            break;
            
        default:
            printf("Estado desconocido.\n");
            break;
    }
}

int main() {
    printf("--- Iniciando simulacion del Molinete ---\n");
    
    procesarEvento(EVENTO_EMPUJAR_BARRERA);   // Intenta pasar sin pagar
    procesarEvento(EVENTO_INSERTAR_MONEDA);   // Paga
    procesarEvento(EVENTO_INSERTAR_MONEDA);   // Intenta pagar de nuevo
    procesarEvento(EVENTO_EMPUJAR_BARRERA);   // Pasa
    
    return 0;
}

3. Alternativas más avanzadas
Para sistemas embebidos complejos o programación avanzada, se pueden utilizar métodos que evitan que el switch-case escale de manera inmanejable:

Tabla de Transiciones (Matriz): Una matriz bidimensional donde las filas son estados, las columnas son eventos, y las celdas contienen el estado siguiente.

Punteros a Funciones: Cada estado es una función independiente, y la variable de estado actual es un puntero a la función que debe ejecutarse.
