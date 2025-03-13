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
