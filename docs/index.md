
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
Aquí se encuentran las funcionalidades básicas que se esperan de cada sensor.

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

El software se organiza como una máquina de estados finita:

1. ARRANQUE: parpadeo del LED1 y mensaje "CARGANDO..."
2. SERVICIO: espera de cliente (subestado b), muestra de temperatura/humedad, navegación de productos con joystick y preparación de café con LED2 progresivo (subestado a).
3. ADMIN: menú de administración con lectura de sensores y edición de precios.

Los espacios y sub espacios usados los definimos:

```cpp
enum class State { ARRANQUE, SERVICIO, ADMIN };
enum class Service_substate { ESPERANDO_CLIENTE, MOSTRANDO_PRODUCTOS };
enum class Admin_substate { MENU, SHOW_TEMP, SHOW_DISTANCE, SHOW_COUNTER, EDIT_PRICES };
```

Teniendo en cuenta lo que se espera de cada componente (p1 Introducción). La implementación se explica a continuación:

#### **LCD**

Para mostrar información el reto de este componente es no sobrecargarlocon escrituras constantes.
Actualizar una pantalla LCD demasiado rápido provoca parpadeos y reduce la legibilidad.
Para evitarlo, la solución implementada consiste en comparar el contenido nuevo con el previo y solo actualizar cuando exista un cambio.
Aun así no en todos los casos ha sido posible, por ejemplo:

- La lectura de temperatura y humedad, que se refresca cada cierto intervalo para evitar saturar el sensor DHT.

- La visualización de segundos, que necesariamente cambia cada segundo.

Como añadido para el simbolo del euro lo he generado a mano.

#### **Sensor Ultrasonido HC-SR04**

El sensor ultrasónico se usa para medir distancias y detectar la presencia de un usuario frente a la máquina.
Inicialmente se valoró integrarlo dentro de un Arduino Thread para realizar mediciones constantes, pero estas rutinas requieren pequeños delays internos para el envío y recepción del pulso, lo que podría introducir ralentizaciones innecesarias en el resto del sistema.

#### **Joystick analógico**

Lectura del eje X / Y utilizados para navegar menús. El joystick tiene ruido en las mediciones por eso le ajustamos un rango que conocemos. (m<300 o m>700).
La lectura del botón se explica más adelante.

#### **LEDs**

Los LEDs muestran estados en el sistema. Debido a las limitaciónes propuestas surgen retos:

Evitar bloquear el programa con funciones tipo delay().

En algunos casos es necesario un efecto PWM progresivo (cafetera/tiempo).

Su solución:
Control por 
```cpp
millis()
```
sin bloquear ejecución compruebo en cada ciclo si ha pasado el tiempo para encenderse o apagarse.

Mapeo de valores analógicos a intensidad PWM (cuando el pin lo permite).

#### Menciones faltantes

Nos falta por mencionar el uso de los botones, el sensor de temperatura y el mecanismo que utilizamos para medir el tiempo. A su vez, la solución de cómo medir el tiempo constantemente.

### 3.1. Arduino Threads

Se utilizan threads (Estos threads no son paralelos librería ArduinoThread / ThreadController) para tareas que deben ejecutarse de forma periódica sin bloquear el sistema, por ejemplo:
- Sensor de Temperatura y Humedad DHT11
- Contador de tiempo

 ¿Por qué utilizamos threads en estos sensores?
Porque su naturaleza es periódica y no depende del estado. El ultrasonido se usa solo en momentos puntuales (cambio de estado),
pero la temperatura y humedad, y los tiempos necesitan medirse SIEMPRE durante el uso:

En SERVICIO para mostrar condiciones ambientales
En ADMIN para registrar datos
En la parte de café para controlar temporizadores

Además, nos permiten modularizar la lógica. Cada thread actúa como un “módulo independiente”.
Por esto, y por las limitaciones de tiempo de lectura del DHT11 es que se decidio no implementar la lectura de temperatura y humedad en una función cómo el ultrasonido.

### 3.2. Interrupciones
El proyecto utiliza interrupciones externas de Arduino (INT0 e INT1) para gestionar eventos críticos procedentes de botones físicos:
- Botón ADMIN → Pin 2 (INT0)
- Botón del Joystick (SW) → Pin 3 (INT1)

Ambos botones están configurados en modo INPUT_PULLUP.

Motivos para usar interrupciones en lugar de lectura por sondeo:
- Perder pulsaciones rápidas del usuario.
- Aumentar la latencia en menús interactivos.

Por ello, las interrupciones garantizan que la placa reacciona inmediatamente.

#### Botón ADMIN

El botón principal se usa para acceder al modo administrador desde cualquier punto del programa. Además de resetear el servicio.
La interrupción permite detectar el evento incluso durante procesos largos y garantizar que siempre se entra al menú ADMIN sin latencia.
Comportamiento implementado:

1. Cuando el usuario pulsa el botón (flanco descendente), se dispara INT0.
2. Guarda el tiempo que ha estado activado y si se ha completado.

En resumen, las interrupciones garantizan que las pulsaciones se detecten aunque el microcontrolador esté ejecutando otras tareas, lo cual no sería fiable usando solo `digitalRead()` periódicos.

#### Botón del Joystick
Usar interrupción permite validar selección en menús sin retraso perceptible y simplifica la detección de la pulsación solo modificando un flag.

### 3.3. Watchdog
El Watchdog Timer (WDT) es un mecanismo de seguridad del microcontrolador que reinicia automáticamente el sistema en caso de que el programa se bloquee o quede atrapado en un bucle infinito.
En este proyecto, el watchdog se utiliza para mejorar la fiabilidad del sistema ante posibles bloqueos inesperados.

En nuestro proyecto se inicializa el watchdog al arrancar el sistema, configurándolo para que resetee el Arduino cada 2 segundos.
En el loop principal se llama periódicamente a ```wdt_reset()``` para indicar que el programa sigue funcionando correctamente y resetear el watchdog.

De esta forma se evita que la máquina expendedora quede congelada sin responder y protege el sistema ante errores de hardware o software.

## 4. Mejoras propuestas

A partir del funcionamiento actual del sistema, se plantean varias mejoras que podrían implementarse en futuras versiones para ampliar las capacidades y mejorar la usabilidad del dispositivo.

4.1. Inclusión de un Motor para Accionamiento Real

Actualmente la máquina expendedora simula el proceso de entrega del producto únicamente mediante indicadores visuales.
Una mejora significativa sería integrar un motor (servo) que realice la acción de apertura de la máquina expendedora, permitiendo simular de forma más realista el comportamiento de una expendedora real.

La estructura de software actual soporta fácilmente esta ampliación al disponer de estados bien definidos. Por tanto solo faltariía un acople.

4.2. Añadir un Potenciómetro para Controlar el Contraste del LCD

El LCD funciona correctamente, pero no dispone de control físico de contraste.
Incorporar un potenciómetro dedicado al pin `V0` del LCD permitiría ajustar el contraste en función de la iluminación ambiente.

4.3. Mejora de la Organización de Software

Se debería reorganizar las funciones repetitivas en módulos más independientes para facilitar la modularización y los cambios.

## 5. Vídeo de la práctica

En el siguiente vídeo se muestra el funcionamiento completo del sistema incluyendo los estados principales y la interacción con los sensores.

[Ver vídeo en OneDrive](https://urjc-my.sharepoint.com/:v:/g/personal/m_morenop_2023_alumnos_urjc_es/EXXsT2KYz1pNpV0Lx879DR4ButtTfbX1o6npgNSIb82OpQ?e=c3xxGh)

## 6. Repositorio

El código fuente del proyecto se hará público una vez concluida la asignatura.
