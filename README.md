# README - Reproducción de audio AAC con ESP32

## Descripción del proyecto

Este ejercicio consiste en reproducir un archivo de audio en formato AAC almacenado en memoria PROGMEM utilizando un ESP32 y la salida I2S. Para ello se utilizan las librerías de audio proporcionadas en el enunciado.

El código utilizado es prácticamente el mismo que el proporcionado por el profesor, sin modificaciones importantes en el funcionamiento.

---

## Modificaciones realizadas respecto al código original

No se han realizado modificaciones funcionales respecto al código proporcionado en el enunciado.

Únicamente:
- se ha añadido `#include <Arduino.h>`
- se ha mantenido el formato y la indentación del código para mejorar la legibilidad

El funcionamiento sigue siendo exactamente el mismo.

---

# 1. Salida por el puerto serie

La comunicación serie se inicializa mediante:

```cpp
Serial.begin(115200);
```

Esto configura el puerto serie a una velocidad de 115200 baudios.

Durante la ejecución del programa, cuando el audio termina de reproducirse, aparece el siguiente mensaje en el monitor serie:

```text
Sound Generator
```

Este mensaje se imprime continuamente cada segundo debido a:

```cpp
Serial.printf("Sound Generator\n");
delay(1000);
```

Por tanto, la salida serie sirve para indicar que la reproducción del audio ha finalizado y que el generador de sonido ya no está en ejecución.

---

# 2. Funcionamiento

## Inclusión de librerías

```cpp
#include "AudioGeneratorAAC.h"
#include "AudioOutputI2S.h"
#include "AudioFileSourcePROGMEM.h"
#include "sampleaac.h"
```

Estas librerías permiten:
- decodificar audio AAC
- enviar audio mediante I2S
- leer audio almacenado en memoria PROGMEM

El archivo `sampleaac.h` contiene el audio AAC almacenado como un array de datos.

---

## Creación de objetos

```cpp
AudioFileSourcePROGMEM *in;
AudioGeneratorAAC *aac;
AudioOutputI2S *out;
```

Se crean tres objetos:
- `in`: fuente de audio almacenada en memoria
- `aac`: decodificador AAC
- `out`: salida de audio I2S

---

## Función setup()

En `setup()` se inicializa todo el sistema de audio.

### Inicialización del puerto serie

```cpp
Serial.begin(115200);
```

Permite visualizar mensajes por el monitor serie.

---

### Carga del audio en memoria

```cpp
in = new AudioFileSourcePROGMEM(sampleaac, sizeof(sampleaac));
```

Se carga el archivo AAC almacenado en `sampleaac.h`.

---

### Creación del decodificador AAC

```cpp
aac = new AudioGeneratorAAC();
```

Este objeto se encarga de decodificar el audio AAC.

---

### Configuración de la salida I2S

```cpp
out = new AudioOutputI2S();
```

Se crea la salida de audio digital I2S.

---

### Ajuste del volumen

```cpp
out->SetGain(0.125);
```

Se configura el volumen de salida al 12.5%.

---

### Configuración de pines I2S

```cpp
out->SetPinout(26, 25, 22);
```

Se configuran los pines:
- GPIO 26 → BCLK
- GPIO 25 → LRCLK / WS
- GPIO 22 → DATA

Estos pines se utilizan para enviar el audio digital al DAC o amplificador I2S.

---

### Inicio de la reproducción

```cpp
aac->begin(in, out);
```

Se inicia la reproducción del audio AAC.

---

## Función loop()

```cpp
if (aac->isRunning()) {
    aac->loop();
}
```

Mientras el audio se está reproduciendo, se ejecuta continuamente `aac->loop()`, encargada de mantener activa la reproducción y procesar el flujo de audio.

---

Cuando el audio termina:

```cpp
aac->stop();
Serial.printf("Sound Generator\n");
delay(1000);
```

- se detiene el reproductor
- se muestra el mensaje por el puerto serie
- se espera 1 segundo antes de repetir el proceso en el `loop()`