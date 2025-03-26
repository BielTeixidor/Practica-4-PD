# Pràctica 4: SISTEMES OPERATIUS EN TEMPS REAL

## Introducció

Aquesta pràctica se centra en els sistemes operatius en temps real i la seva capacitat per gestionar múltiples tasques de manera eficient. Es defineix un temps determinat per a cada tasca, permetent repartir el temps de processament entre elles.

## Exercici pràctic *part 1*

En aquesta primera part, es crea un programa que implementa dues tasques en paral·lel utilitzant FreeRTOS a l'ESP32. Cada tasca imprimeix missatges diferents al port sèrie de manera alternada cada segon.

**Tasca principal:** S'executa a la funció `loop()`. Imprimeix "this is ESP32 Task" cada segon.

**Segona tasca:** Creada des de la funció `setup()` com a `anotherTask`. Imprimeix "this is another Task" també cada segon, executant-se de manera concurrent amb la tasca principal.

El resultat mostrat al port sèrie és:

```
- this is ESP32 Task  
- this is another Task  
- this is ESP32 Task  
- this is another Task  
```

### Codi

```cpp
#include<Arduino.h>

void anotherTask(void *parameter);

void setup() {
  Serial.begin(112500);

  // Creamos una nueva tarea aquí
  xTaskCreate(
      anotherTask,     // Función de la tarea
      "another Task",  // Nombre de la tarea
      10000,            // Tamaño de la pila de la tarea
      NULL,             // Parámetro de la tarea
      1,                // Prioridad de la tarea
      NULL              // Manejador de la tarea para realizar un seguimiento de la tarea creada
  );
}

// La función loop() es invocada por el loopTask de Arduino ESP32
void loop() {
  Serial.println("this is ESP32 Task");
  delay(1000);
}

// Esta función se invocará cuando se haya creado la tarea adicional (anotherTask)
void anotherTask(void *parameter) {
  // Bucle infinito
  for (;;) {
    Serial.println("this is another Task");
    delay(1000);
  }

  // Eliminar la tarea al finalizar (esto nunca ocurrirá porque es un bucle infinito)
  vTaskDelete(NULL);
}
```

---

## Exercici pràctic *part 2* - *Semàfor*

En aquesta segona part, es millora el control de dos LEDs mitjançant l'ús de semàfors en FreeRTOS. El semàfor garanteix que només una de les tasques controli els LEDs a la vegada, assegurant que un estigui encès mentre l'altre es manté apagat.

### Descripció del codi

El programa configura dos pins (LED1 i LED2) com a sortides i crea un semàfor binari. Es generen dues tasques:

- **TascaLed1:** Encén el LED1 i apaga el LED2.
- **TascaLed2:** Encén el LED2 i apaga el LED1.

Cada tasca espera obtenir el semàfor (`xSemaphoreTake`) per executar-se. En acabar, allibera el semàfor (`xSemaphoreGive`) perquè l'altra tasca prengui el control.

S'ha afegit `vTaskDelay(1000 / portTICK_PERIOD_MS)` per fer el retard més precís i compatible amb la configuració interna de FreeRTOS.

### Funcionament i sortida pel port sèrie

El programa alterna entre les dues tasques cada segon. A la terminal es mostrarà:

```
LED1 ON / LED2 OFF  
LED1 OFF / LED2 ON  
LED1 ON / LED2 OFF  
LED1 OFF / LED2 ON  
...  
```

### Codi

```cpp
#include <Arduino.h>
#include <FreeRTOS.h>
#include <task.h>
#include <semphr.h>

const int ledPin = 11;

SemaphoreHandle_t semaphore;

void setup() {
    Serial.begin(115200);
    pinMode(ledPin, OUTPUT);

    semaphore = xSemaphoreCreateBinary();
    
    xTaskCreate(encenderLED, "Encender LED", 1000, NULL, 1, NULL);
    xTaskCreate(apagarLED, "Apagar LED", 1000, NULL, 1, NULL);
}

void loop() {

}

void encenderLED(void *parameter) {
    for (;;) {
        digitalWrite(ledPin, HIGH);
        Serial.println("LED HIGH");
        delay(1000);
        xSemaphoreGive(semaphore); 
    }
}

void apagarLED(void *parameter) {
    for (;;) {
        digitalWrite(ledPin, LOW);
        Serial.println("LED LOW");
        delay(1000);
        xSemaphoreGive(semaphore); 
    }  
}
```

