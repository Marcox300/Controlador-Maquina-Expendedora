
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

> **Nota:** En la imagen exportada falta accidentalmente la conexión al pin **5V**, pero en el montaje real sí está presente.

#### Conexiones por dispositivo

- LCD 16×2 (NARANJA)

| Señal LCD   | Pin Arduino                            |
| ----------- | -------------------------------------- |
| RS          | 12                                     |
| E           | 11                                     |
| D4          | 5                                      |
| D5          | 4                                      |
| D6          | A4                                     |
| D7          | A0                                     |
| VSS         | GND                                    |
| VDD         | 5V                                     |
| RW          | GND                                    |
| Backlight A | 5V (con resistencia de 220 Ω en serie) |
| Backlight K | GND                                    |


- Sensor de Temperatura y Humedad DHT11 (MORADO)

| Señal | Pin |
| ----- | --- |
| DATA  | 7   |
| VCC   | 5V  |
| GND   | GND |


- Sensor Ultrasonidos HC-SR04 (AMARILLO)

| Señal | Pin |
| ----- | --- |
| TRIG  | 9   |
| ECHO  | 10  |
| VCC   | 5V  |
| GND   | GND |


- Joystick (VERDE)

| Señal | Pin |
| ----- | --- |
| VRx   | A1  |
| VRy   | A2  |
| SW    | 3   |
| VCC   | 5V  |
| GND   | GND |

- Botón (MARRÓN)

| Señal | Pin |
| ----- | --- |
| Señal | 2   |
| Común | GND |


- LEDs indicadores (ROSA)

| LED  | Pin | Notas                                 |
| ---- | --- | ------------------------------------- |
| LED1 | A3  | Salida digital normal                 |
| LED2 | 6   | **PWM**, usado para control de brillo |

Ambos llevan resistencia de 220 Ω en serie y cátodo a GND.

#### Capacidades especiales de los pines y motivos de su elección
- Pin 6 — Salida PWM

Se utiliza para el LED indicador LED2, permitiendo variar su intensidad mediante ```analogWrite()``` como requiere la práctica.

- Pines 2 y 3 — Interrupciones externas (INT0 e INT1)

Los dos botones (ADMIN y joystick SW) se conectan aquí para 
detectar pulsaciones incluso si el micro está en otras tareas garantizando la detección en eventos importantes, se explicará más adelante.

## 3. Software y lógica de la máquina

### 3.1. Arduino Threads

### 3.2. Interrupciones

### 3.3. Watchdog

## 4. Mejoras propuestas

## 5. Vídeo de la práctica
[Ver vídeo en OneDrive](https://urjc-my.sharepoint.com/:v:/g/personal/m_morenop_2023_alumnos_urjc_es/EXXsT2KYz1pNpV0Lx879DR4ButtTfbX1o6npgNSIb82OpQ?e=c3xxGh)

## 6. Repositorio

El código fuente del proyecto se hará público una vez concluida la asignatura.
