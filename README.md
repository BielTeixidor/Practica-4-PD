# Practica 2 : INTERRUPCIONES

En esta segunda práctica se trabaja las interrupciones, dispondremos de 2 leds para llevarla a cabo, ademas deL microcontrolador ESP32.

La práctica incluye de 2 partes:

**- Práctica A:**
En esta parte se estudia las interrupciones por GPIO.

**- Práctica B:**
En esta otra parte las interrupciones se estuadian por timer.

# 1.-Practica A

```c++
struct Button {
const uint8_t PIN;
uint32_t numberKeyPresses;
bool pressed;
};
Button button1 = {18, 0, false};
void IRAM_ATTR isr() {
button1.numberKeyPresses += 1;
button1.pressed = true;
}
void setup() {
Serial.begin(115200);
pinMode(button1.PIN, INPUT_PULLUP);
attachInterrupt(button1.PIN, isr, FALLING);
}
void loop() {
if (button1.pressed) {
Serial.printf("Button 1 has been pressed %u times\n",
button1.numberKeyPresses);
button1.pressed = false;
}

//Detach Interrupt after 1 Minute
static uint32_t lastMillis = 0;
if (millis() - lastMillis > 60000) {
lastMillis = millis();
detachInterrupt(button1.PIN);
Serial.println("Interrupt Detached!");
}
}
```
Respecto al funcionamineto de esta parte de la práctica consiste en ---- poner el cable en G18 Y LA OTRA EN EL GND EL CONTACTO EN ESTE ULTIMO GENERA LAS INTERRUPCIONES

