
## 1. Introducción
En esta práctica se ha desarrollado un controlador de máquina expendedora utilizando Arduino UNO.
El sistema permite gestionar tanto un modo SERVICIO, donde los clientes pueden seleccionar el café que quiere, como un modo ADMIN, destinado al mantenimiento y configuración del sistema.
El objetivo principal es integrar diversos sensores y actuadores, 
implementando un software robusto que utilice threads, interrupciones y watchdog, garantizando el funcionamiento estable incluso ante posibles fallos.

Componentes utilizados:
-Arduino UNO
-Pantalla LCD 16×2
-Sensor de temperatura y humedad DHT11
-Sensor ultrasonidos HC-SR04
-Joystick XY con botón
-Botón
-LEDs

Para más información de los objetivos y el resultado esperado ver enunciado:
[Descargar PDF del enunciado](Practica3.pdf)

## 2. Hardware y conexiones
El montaje físico del proyecto se corresponde al siguiente diagrama:
![conexiones](https://raw.githubusercontent.com/Marcox300/Controlador-Maquina-Expendedora/main/img/emp_espendedora.png)

Los colores correponden a:
| Color        | Dispositivo  |
| ------------ | ------------ |
| **Rojo**     | 5V           |
| **Negro**    | GND          |
| **NARANJA**  | LCD          |
| **VERDE**    | Joystick     |
| **AMARILLO** | Ultrasonidos |
| **ROSA**     | LEDs         |
| **MARRÓN**   | Botón        |
| **MORADO**   | DHT11        |
En la imagen exportada, la conexión al +5 V no aparece por error, aunque en el montaje real sí debe existir.

- LCD 16×2 (NARANJA)

RS → Pin 12

E → Pin 11

D4 → Pin 5

D5 → Pin 4

D6 → Pin A4

D7 → Pin A0

VSS → GND

VDD → 5V

RW → GND

Backlight A → 5V (a través de resistencia serie 220)

Backlight K → GND

- Sensor de Temperatura y Humedad DHT11 (MORADO)

DATA → Pin 7

VCC → 5V

GND → GND

- Sensor Ultrasonidos HC-SR04 (AMARILLO)

TRIG → Pin 9

ECHO → Pin 10

VCC → 5V

GND → GND

- Joystick (VERDE)

VRx → A1

VRy → A2

SW → 3

VCC → 5V

GND → GND

Botón ADMIN (MARRÓN)

Señal → Pin 2 (modo INPUT_PULLUP)

El botón se conecta a GND al pulsar.

- LEDs indicadores (ROSA)

LED1 → A3

LED2 → 6

Cada LED lleva en serie una resistencia de ≈220 Ω.

Cátodo de cada LED → GND

## 3. Software y lógica de la máquina

### 3.1. Arduino Threads

### 3.2. Interrupciones

### 3.3. Watchdog

## 4. Mejoras propuestas

## 5. Vídeo de la práctica
[Ver vídeo en OneDrive](https://urjc-my.sharepoint.com/:v:/g/personal/m_morenop_2023_alumnos_urjc_es/EXXsT2KYz1pNpV0Lx879DR4ButtTfbX1o6npgNSIb82OpQ?e=c3xxGh)

## 6. Repositorio

El código fuente del proyecto se hará público una vez concluida la asignatura.
