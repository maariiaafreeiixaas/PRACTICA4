# Pràctica 4 : Sistemes operatius en temps real

## Objectiu
Comprendre el funcionament d'un sistema operativu en temps real.
Per fer-ho realitzarem una practica on generarem varies tasques i veurem com s'ejecuten dividint el temps d'us de la cpu.

## Excercici 1
El codi crea dues tasques independents que s'executen concurrentment en l'ESP32 utilitzant FreeRTOS.

Ambdues tasques utilitzen delay(1000) per esperar un segon entre impressions. El planificador de FreeRTOS gestiona l'execució d'aquestes tasques. Cada tasca entra en un bucle infinit (for(;;)), assegurant que s'executin contínuament.

Quan es crida delay(1000) en una tasca, FreeRTOS suspèn aquesta tasca i permet que altres tasques s'executin. Això resulta en l'alternança dels missatges impresos al port sèrie.

### Codi

```cpp

//setup(): Inicialitza la comunicació sèrie i crea una tasca anomenada anotherTask.
void setup() 
{ 
  Serial.begin(112500); 
  /* we create a new task here */ 
  xTaskCreate(
    anotherTask, /* Task function. */ 
    "another Task", /* name of task. */ 
    10000, /* Stack size of task */ 
    NULL, /* parameter of the task */ 
    1, /* priority of the task */ 
    NULL); /* Task handle to keep track of created task */ 
} 

//loop(): Aquesta és la tasca principal d'Arduino que s'executa per defecte. Imprimeix "this is ESP32 Task" al port sèrie cada segon.
void loop() 
{ 
  Serial.println("this is ESP32 Task"); 
  delay(1000); 
} 

//anotherTask(void * parameter): Aquesta és la funció de la tasca addicional. També imprimeix "this is another Task" al port sèrie cada segon.
void anotherTask(void * parameter) 
{ 
  /* loop forever */ 
  for(;;) 
  { 
    Serial.println("this is another Task"); 
    delay(1000); 
  } 
  /* delete a task when finish, this will never happen because this is infinity loop */ 
  vTaskDelete(NULL); 
}
```
### 1. Descriure la sortida pel port sèrie:

S'alternaran dos missatges cada segon, de la següent manera:
this is ESP32 Task
this is another Task
this is ESP32 Task
this is another Task
this is ESP32 Task
this is another Task
...

## Exercici 2 (casa)
Crear un programa que utilitzi dues tasques, una per encendre un LED i l'altra per apagar-lo. Les tasques han d'estar sincronitzades utilitzant un semàfor binari, assegurant que el LED s'encengui i apagui alternativament.

### Codi
```cpp
#include <Arduino.h>
#include <freertos/FreeRTOS.h>
#include <freertos/task.h>
#include <freertos/semphr.h>

// Pin del LED
const int ledPin = 2;

// Declarar el semàfor
SemaphoreHandle_t xSemaphore;

void setup() {
  // Inicialitzar el pin del LED
  pinMode(ledPin, OUTPUT);

  // Crear el semàfor binari
  xSemaphore = xSemaphoreCreateBinary();

  // Inicialitzar el semàfor per a que la tasca d'encès tingui el primer accés
  xSemaphoreGive(xSemaphore);

  // Crear les tasques
  xTaskCreate(toggleLEDon, "Toggle LED ON", 1000, NULL, 1, NULL);
  xTaskCreate(toggleLEDoff, "Toggle LED OFF", 1000, NULL, 1, NULL);
}

void loop() {
  // Deixar el loop buit
}

void toggleLEDon(void * parameter) {
  for(;;) {
    // Esperar fins que el semàfor estigui disponible
    if(xSemaphoreTake(xSemaphore, portMAX_DELAY)) {
      // Encendre el LED
      digitalWrite(ledPin, HIGH);
      // Esperar 500ms
      vTaskDelay(500 / portTICK_PERIOD_MS);
      // Donar el semàfor a la tasca d'apagat
      xSemaphoreGive(xSemaphore);
    }
  }
}

void toggleLEDoff(void * parameter) {
  for(;;) {
    // Esperar fins que el semàfor estigui disponible
    if(xSemaphoreTake(xSemaphore, portMAX_DELAY)) {
      // Apagar el LED
      digitalWrite(ledPin, LOW);
      // Esperar 500ms
      vTaskDelay(500 / portTICK_PERIOD_MS);
      // Donar el semàfor a la tasca d'encès
      xSemaphoreGive(xSemaphore);
    }
  }
}
```
