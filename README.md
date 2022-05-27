# Practica_2
Esta práctica nos introduce y explica las interrupciones hardware que, a rasgos generales, son señales que interrumpen la actividad normal de nuestro microprocesador y saltan a atenderla. En nuestro caso con la ESP32, podemos definir una función de rutina de servicio de interrupción que se llamará cuando un pin GPIO cambie el valor de su señal. Todos los pines de la ESP pueden ser configurados de esta manera.
## Codigo_2.1_Interrupción_GPIO

```
#include <Arduino.h>
 
struct Button {
  const uint8_t PIN;
  uint32_t numberKeyPresses;
  bool pressed;
};
 
Button button1 = {5, 0, false};
 
void IRAM_ATTR isr() {
  button1.numberKeyPresses += 1;
  button1.pressed = true;
}
 
void setup()
{
  Serial.begin(115200);
  pinMode(button1.PIN, INPUT_PULLUP);
  attachInterrupt(button1.PIN, isr, FALLING);
}
 
void loop()
{
  if (button1.pressed) {
    Serial.printf("Button 1 has been pressed %u times\n", button1.numberKeyPresses);
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
## Funcionamiento
Respecto al codigo primero se crea un ``` struct Button ``` con el pin de entrada, un contador y un booleano. Lo necesario para saber si se esta pulsando y cuantas veces se ha pulsado el mismo. A continuación tenemos ``` vodi IRAM_ATTR isr() ```, que es la funcion que se llamara cada vez que se dispare la interrupción. Dentro encontramos que la funcion augmenta en 1 el contador del button1 y pone su booleano pressed a true para indicar que se ha pulsado. En el setup simplemente declaramos todo lo que necesitamos en el loop y añadimos la interrupción con la funcion ``` attachInterrupt(button1.PIN, isr, FALLING) ```. Los tres parámetros son el pin para establecer la clavija de interrupción, la función que se ira repitiendo y el modo que define cuando se debe disparar la interrupción. Usamos FALLING, lo que nos indica que nuestra interrupción ocurre cuando el pin va de HIGH a LOW.
