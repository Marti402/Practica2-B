# Practica2_Alexandre_Martí
Participants: Alexandre Pascual / Martí Vila

# _Timer Interrupt_ en ESP32

Este proyecto utiliza un **timer hardware** en un **ESP32** para generar interrupciones periódicas cada 1 segundo y contar el número total de interrupciones.

##  Descripción del Código

El código configura un temporizador de hardware en el ESP32 utilizando la API de **FreeRTOS** y las funciones de temporización del ESP32. La interrupción generada por el temporizador se maneja mediante una función especial en la **RAM interna** (`IRAM_ATTR`).

##  Funcionamiento

1. Se inicializa el **serial monitor** para la depuración.
2. Se configura un **temporizador de hardware** que genera interrupciones cada 1 segundo.
3. En cada interrupción, se incrementa un contador (`interruptCounter`).
4. En el `loop()`, si ha ocurrido una interrupción, se reduce `interruptCounter` y se incrementa `totalInterruptCounter`, mostrando el total en el monitor serie.

##  Explicación de Componentes Clave

###  Variables Globales

- `volatile int interruptCounter;` → Contador de interrupciones.
- `int totalInterruptCounter;` → Contador total de interrupciones.
- `hw_timer_t * timer = NULL;` → Puntero al temporizador de hardware.
- `portMUX_TYPE timerMux = portMUX_INITIALIZER_UNLOCKED;` → Mutex para evitar condiciones de carrera.

###  Función de Interrupción (`onTimer`)

```cpp
void IRAM_ATTR onTimer() {
    portENTER_CRITICAL_ISR(&timerMux);
    interruptCounter++;
    portEXIT_CRITICAL_ISR(&timerMux);
}
```

- Esta función se ejecuta cuando el temporizador genera una interrupción.
- Usa `portENTER_CRITICAL_ISR()` y `portEXIT_CRITICAL_ISR()` para proteger la variable compartida `interruptCounter`.
- Se almacena en la **RAM interna** (`IRAM_ATTR`) para una ejecución más rápida y segura.

###  Configuración en `setup()`

```cpp
void setup() {
    Serial.begin(115200);
    timer = timerBegin(0, 80, true);
    timerAttachInterrupt(timer, &onTimer, true);
    timerAlarmWrite(timer, 1000000, true);
    timerAlarmEnable(timer);
}
```

- Inicia la comunicación serial a **115200 baudios**.
- Configura un temporizador (`timerBegin`):
  - `0` → Usa el temporizador 0.
  - `80` → Divide el reloj de la CPU (80 MHz / 80 = 1 MHz, o 1 µs por tick).
  - `true` → Cuenta en modo ascendente.
- Se **adjunta la interrupción** con `timerAttachInterrupt()`.
- Se **configura la alarma** para que se dispare cada **1.000.000 de ticks** (1 segundo).
- Se habilita la alarma con `timerAlarmEnable()`.

###  Bucle Principal (`loop()`)

```cpp
void loop() {
    if (interruptCounter > 0) {
        portENTER_CRITICAL(&timerMux);
        interruptCounter--;
        portEXIT_CRITICAL(&timerMux);
        totalInterruptCounter++;
        Serial.print("An interrupt has occurred. Total number: ");
        Serial.println(totalInterruptCounter);
    }
}
```

- Comprueba si ha ocurrido una interrupción.
- Reduce `interruptCounter` dentro de una **sección crítica** para evitar condiciones de carrera.
- Aumenta `totalInterruptCounter` y muestra el total en **Serial Monitor**.

##  Requisitos

- **ESP32**
- **Arduino IDE** con las librerías de ESP32 instaladas

##  Notas

- La función `IRAM_ATTR` es necesaria para que la interrupción se ejecute en **RAM interna** y no en memoria flash.
- Se usa `portMUX_TYPE` para evitar errores en el acceso a `interruptCounter` en un sistema de múltiples núcleos.

##  Salida esperada en Serial Monitor

```
An interrupt has occurred. Total number: 1
An interrupt has occurred. Total number: 2
An interrupt has occurred. Total number: 3
...
```

Cada línea aparecerá cada 1 segundo.

---

### 📌 Autor

Desarrollado para **ESP32** usando **interrupciones de temporizador**. 🚀

