# P4

## Objectiu
L'objectiu de la pràctica és comprendre el funcionament d'un sistema operatiu en temps real mitjançant la creació de diverses tasques i observar com s'executen compartint el temps d'ús de la CPU.

## Materials
- ESP32  
- 2 LEDS  

## Codi

```cpp
void setup()
{
    Serial.begin(112500);
    /* Creem una nova tasca aquí */
    xTaskCreate(
        anotherTask, /* Funció de la tasca */
        "another Task", /* Nom de la tasca */
        10000, /* Mida de la pila de la tasca */
        NULL, /* Paràmetre de la tasca */
        1, /* Prioritat de la tasca */
        NULL /* Handle per fer seguiment de la tasca creada */
    );
}

/* La funció loop() es crida contínuament per l'ESP32 */
void loop()
{
    Serial.println("this is ESP32 Task");
    delay(1000);
}

/* Aquesta funció s'executa quan es crea anotherTask */
void anotherTask(void * parameter)
{
    /* Bucle infinit */
    for(;;)
    {
        Serial.println("this is another Task");
        delay(1000);
    }
    /* Eliminació de la tasca quan finalitzi (aquí mai passa perquè el bucle és infinit) */
    vTaskDelete(NULL);
}
```

## Sortida esperada

```
    this is another Task
    this is ESP32 Task
    this is another Task
    this is ESP32 Task
```

## Descripció
Aquest codi mostra com crear una tasca addicional per permetre l'execució concurrent de diverses tasques. La tasca addicional imprimeix un missatge periòdicament per sèrie mentre el bucle principal (`loop()`) realitza una acció similar. Aquesta tasca es crea mitjançant la funció `xTaskCreate` i s'executa a `anotherTask`.

---

## Codi amb semàfors

```cpp
#include <Arduino.h>
#include <FreeRTOS.h>
#include <task.h>
#include <semphr.h>

// Definició de pins
const int ledPin = 23;

// Declaració de semàfors
SemaphoreHandle_t semaforEncendre;
SemaphoreHandle_t semaforApagar;

// Prototips de funcions
void tascaEncendre(void *parameter);
void tascaApagar(void *parameter);

void setup() {
    Serial.begin(115200);

    // Inicialització de semàfors
    semaforEncendre = xSemaphoreCreateBinary();
    semaforApagar = xSemaphoreCreateBinary();

    // Creació de tasques
    xTaskCreate(tascaEncendre, "Encendre LED", 1000, NULL, 1, NULL);
    xTaskCreate(tascaApagar, "Apagar LED", 1000, NULL, 1, NULL);

    // Inicialització del pin del LED
    pinMode(ledPin, OUTPUT);
}

void loop() {
    // No s’utilitza en aquest exemple
}

void tascaEncendre(void *parameter) {
    for (;;) {
        // Espera fins que es desbloquegi el semàfor per encendre el LED
        xSemaphoreTake(semaforEncendre, portMAX_DELAY);

        Serial.println("Encenent LED");
        digitalWrite(ledPin, HIGH);
        delay(1000); // Espera 1 segon

        // Allibera el semàfor perquè l'altra tasca pugui apagar el LED
        xSemaphoreGive(semaforApagar);
    }
}

void tascaApagar(void *parameter) {
    for (;;) {
        // Espera fins que es desbloquegi el semàfor per apagar el LED
        xSemaphoreTake(semaforApagar, portMAX_DELAY);

        Serial.println("Apagant LED");
        digitalWrite(ledPin, LOW);
        delay(1000); // Espera 1 segon

        // Allibera el semàfor perquè l'altra tasca pugui encendre el LED
        xSemaphoreGive(semaforEncendre);
    }
}
```

## Descripció
En aquest codi es creen dues tasques (`tascaEncendre` i `tascaApagar`) que s'executen en bucles infinits. Cada tasca espera que es desbloquegi un semàfor específic abans de realitzar la seva acció (encendre o apagar el LED). Després d’executar l’acció corresponent, la tasca allibera el semàfor adequat perquè l'altra tasca pugui executar-se. Això garanteix una sincronització correcta entre les dues tasques.

