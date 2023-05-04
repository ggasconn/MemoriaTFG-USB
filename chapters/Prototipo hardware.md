<!-- Leave a blank line before the title -->

# Desarrollo del prototipo hardware

En este capítulo se describe con detalles a más bajo nivel sobre los periféricos y hardware utilizado para el desarrollo del proyecto, así como una breve introducción de la familia de microcontroladores de la que se ha hecho uso en el proyecto.




## Microcontroladores

En el desarrollo del proyecto, se han utilizado dos microcontroladores sobre los que se han probado el firmware. A continuación, se explica cada uno de ellos, así como una explicación general sobre el origen de esta familia de microcontroladores.



### Familia de microcontroladores AVR

Esta familia de microcontroladores AVR provienen del fabricante Atmel (actualmente perteneciente a Microchip Technology [@atmel-web]), principalmente diseñados para el desarrollo de sistemas empotrados y aplicaciones relacionados con robótica, así como su uso en entornos industriales. Su interfaz y diseño es relativamente sencillo, por lo que implementar aplicaciones que hagan uso de estos dispositivos resulta realmente atractivo para programadores que quieran aprender a desarrollar software en el que se utilice, por ejemplo, el protocolo USB, contando con ello con un gran abanico de utilidades y periféricos.

Estos microcontroladores, entre los que se incluyen ATTiny85 y ATmega, los caracterizamos por su bajo consumo energético. El uso de una arquitectura RISC hace que sea una ventaja en el procesamiento de datos, aunque como veremos más adelante, en el ATTiny85 se cuenta con una limitación en cuanto a temporización, que en el caso del desarrollo de este proyecto ha sido un problema para determinados periféricos. Salvo esto último, la única complejidad que puede presentar el desarrollo de un firmware es la adquisición de los conocimientos propios de toda la API de USB. [@avr-products]

En cuanto a componentes hardware, estos chips cuentan con una memoria flash para el almacenamiento del firmware, una SRAM y, como habíamos mencionado antes para el caso del ATTiny85, cuentan también con temporizadores. Se puede hacer uso de un puerto serie (para el caso del ATmega) y, por ejemplo, un conversor ADC, para interactuar con dispositivos externos como un sensor de temperatura, y más componentes que se detallarán más adelante para los dos microchips que se han utilizado en este proyecto. [@avr-info]



### ATTiny85

Este es el microcontrolador usado originalmente para el desarrolo del firmware con V-USB, que controla un anillo circular de leds junto a una pantalla LCD OLED. Debido a las limitaciones en cuanto a puertos y temporización que presenta, nos vimos obligados a buscar otras alternativas, entre ellas nos decantamos por el microchip ATmega328p que describiremos más adelante.

A continuación vamos a hablar de las características hardware de este chip, así como datos relacionados con su arquitectura. Al ser un chip bastante sencillo, está implementado con una arquitectura de 8 bits. Al igual que otros chips de la familia AVR, y en especial éste, cuenta con un bajo consumo al tener pocos pines de entrada/salida, solamente 6. Se cuenta con 8 kilobytes de memoria flash para almacenar el firmware, 512 bytes de EEPROM donde almacenamos el *bootloader*, y 512 bytes de SRAM. Como hemos mencionado anteriormente, cuenta con 6 pines de I/O de propósito general (desde PB0 hasta PB5), una limitación bastante importante en el proyecto, ya que solamente se puede hacer uso de unos pocos periféricos conectados simultáneamente. Otro de los problemas encontrados en el desarrollo de este proyecto, tiene que ver con el oscilador interno, ya que resulta un problema en términos de temporización al ejecutar aplicaciones que requieran estar recibiendo datos de un dispositivo externo, en este caso se ha hecho uso de un sensor de temperatura. [@avr-attiny85] [@attiny85-datasheet]

![Chip ATTiny85](img/ATTiny85.jpg){width=30% #fig:label1}

La elección de este microcontrolador para el proyecto resultó originalmente ser una gran ventaja debido a la gran posibilidad de usos con el que se contaba, además de poder integrarlo con el entorno Arduido y con el que se realizaron distintas pruebas previas para entender el funcionamiento del bootloader de Micronucleus (originalmente cargado en el microchip y totalmente compatible con Arduino). Posteriormente, al desarrollar nuestro propio firmware basado en V-USB, decidimos dejar de usar este entorno para poder cargar un bootloader propio de AVR a la EEPROM, y de esta forma, mediante un programador, poder flashear el firmware V-USB dentro del ATTiny85. Así, dejamos de depender del entorno Arduino y pudimos trabajar en el desarrollo de V-USB puro, entendiendo toda la funcionalidad de este firmware y del propio protocolo USB a bajo nivel. Todo esto se explica en los siguientes apartados, cuando se hace uso del ATTiny85 con la placa Digispark que ya lo incorpora, por lo que utilizar Micronucleus con Arduino en Digispark nos resultó realmente sencillo.

![Integración del entorno Arduino con ATTiny85](img/attiny85_arduino.png){width=55% #fig:label2}

Un factor a destacar de este microchip es el bajo consumo que requiere para funcionar, por lo que para futuras versiones del proyecto, se puede integrar una batería para demostrar que no necesita de una cantidad muy grande de energía y puede aprovechar al máximo su duración. Un ejemplo concreto podría ser una integración de batería y sensor de temperatura con pantalla OLED, para monitorizar la temperatura de manera constante (aunque con ATTiny85 el uso de un sensor de temperatura es limitado por lo comentado anteriormente de la temporización). [@attiny85-desc]

A continuación, se detalla el esquema de la configuración de cada uno de los pines para el chip.

![Detalle de los pines del ATTiny85](img/attiny85_pines.png){width=60% #fig:label3}



### ATmega328p

Tras estar trabajando con el ATTiny85, nos dimos cuenta de las limitaciones en cuanto a puertos de I/O y temporización que éste contaba. Por lo que decidimos buscar alternativas de chips dentro de la familia AVR, y nos llamó la atención este microchip al contar con un número bastante grande de puertos de entrada/salida, ya que cuenta con 23 pines. Además, utilizando el ATmega no contamos con las limitaciones de temporización (el ATmega cuenta con 3 *timers* o temporizadores), por lo que el uso de dispositivos que envíen datos al microchip (dispositivos de entrada) resulta muy adecuado al no presentar ninguna limitación que pueda provocar inestabilidad en el firmware o lectura de datos incorrecta.

![Chip ATmega](img/ATMEGA328P.jpg){width=55%}

A diferencia de los 512 KB del ATTiny85, éste cuenta con 1 KB de memoria EEPROM donde se almacena el bootloader, 2 KB de SRAM y, como hemos mencionado anteriormente, 23 pines I/O de propósito general, una de las grandes ventajas. 

Otros datos sobre el hardware son las 24 líneas de interrupción internas que tiene. Cuenta con una interfaz de puerto serie de 2 cables, desarrollado en el firmware para poder depurar errores en el código. Se hace uso conectando un cable mini-USB al host, y a través de una terminal puerto serie (en Linux, por ejemplo Minicom), se pueden ver los mensajes que lanza el firmware. Sobre el uso de esta forma de depuración en ATmega, se explica con detalle en el capítulo 5 de esta memoria.

El dispositivo puede operar entre 1.8 y 5.5 voltios, en función de distintos factores como el número de periféricos que contenga. Puede alcanzar una respuesta de 1 MIPS. [@nano-pinout]

![Placa Arduino NANO con ATmega328p](img/atmega_board.png){width=40%}





## Placas que utilizan estos microcontroladores

A continuación se detallan las dos placas sobre las que se han hecho pruebas en este proyecto.



### Digispark con ATTiny85

Para el desarrollo original del firmware con el microchip ATTiny85, escogimos hacer uso de la placa Digispark, que además cuenta con una interfaz USB necesaria para el desarrollo de este proyecto.
Los detalles técnicos que cuenta Digispark son similiares al chip ATTiny, salvo la interfaz USB y los pines de entrada de corriente con los que cuenta. Puede funcionar con un voltaje de 5V.

![Pinout de la placa Digispark](img/Digispark_Pinout.png){width=55% #fig:label2}

Su tamaño de 25 mm x 18 mm, lo que resulta realmente manejable y portable. Incluye una interfaz USB para programación y comunicación con el host, así como un regulador de voltaje y LED de encendido. [@digispark-board]

![Placa de desarrollo Digispark](img/digispark_board.jpg){width=35% #fig:label2}

Una característica única de la placa Digispark es que viene preprogramada con un cargador de arranque que permite programarla a través de USB usando el IDE de Arduino (llamado Micronucleous Bootloader). Con ello se puede programar código desde Arduino fácilmente a la placa sin necesidad de un programador independiente. Aunque para el desarrollo de nuestro proyecto, se ha optado por no depender de Arduino y utilizar la librería V-USB pura, entendiento todos los aspectos de bajo nivel del protocolo y comunicación USB, por lo que se requiere de un bootloader de AVR y un programador externo. [@setup-digispark]



### NANO con ATmega328p y puerto serie

Para el prototipo final del proyecto, hacemos uso de la placa Nano que incorpora el microchip ATmega328p. Las ventajas de utilizar esta placa en relación a Digispark son bastantes, además de poder contar con una interfaz puerto serie utilizada en el proyecto para depuración. En esta placa se destaca también el bajo consumo energético que realiza, y el amplio número de pines de E/S con el que se cuenta, en este caso 14.

Esta placa también incluye diversos periféricos, como convertidores analógico-digitales o salidas de modulación por ancho de pulsos (PWM).

![Pinout de la placa NANO con ATmega328p](img/nanopinout.png){width=50%}

Además de los pines de E/S mencionados anteriormente, contamos con 8 pines de entrada analógica y 6 pines PWM. Cuenta también con un puerto USB utilizado en este caso como salida de puerto serie (para depuración del firmware), y un botón de reset para reiniciar la placa. [@nano-info]

Una de las características más atractivas de la placa Nano es su compatibilidad con Arduino IDE, aunque nosotros desarrollamos y cargamos el firmware directamente sin utilizar este entorno, como se explica en otro apartados.

El uso de esta placa es una opción muy interesante para la creación de prototipos y pruebas, ya que se puede integrar fácilmente en circuitos breadboard o perfboard. Para realizar las pruebas con USB, se utilizó esta placa con una breadboard para conectar distintos diodos que posibilitaran el funcionamiento de este puerto.





## Periféricos con soporte en el proyecto

COMPLETAR: placa de expansión genérica, integra algunos periféricos y permite la conexión de otros. 



### Anillo de LEDs de colores

Para probar el firmware, en la primera versión se ha hecho uso de la tira de leds circular fabricado por la empresa NeoPixel [@LED-strip]. Dichos leds se encuentran en disposición circular con colores RGB direccionables individualmente.

![Tira de LEDs circular de NeoPixel sobre una placa Arduino](img/LED-Strip.jpg){width=45% #fig:label3}

Estos LEDs están conectados en cadena, y cada LED está controlado por una sola línea de datos. La línea de datos usa un protocolo de comunicación específico llamado *One-Wire Protocol*, es un protocolo software usado para controlar el color y el brillo de cada LED individual. Su funcionamiento se explica en los siguientes apartados. De esta forma, conseguimos hacer iluminar todos los leds del anillo a la vez, o crear una serie de patrones para que se vayan encendiendo poco a poco, con distintos colores. 

La tira de LED circular se alimenta con una fuente de alimentación de 5V que proporciona la placa Digispark o Nano. Está dirigida por un microcontrolador, en nuestro caso originalmente el ATTiny85, posteriormente en la versión definitiva del proyecto se usa el ATmega328p.



#### Smart Shift Registers en los LEDs de NeoPixel

Cada LED individual en la tira de LEDs circular de NeoPixel contiene un registro, con un pequeño microcontrolador y un LED RGB (Red, Green, Blue). El microcontrolador de cada LED es el responsable de controlar el color y el brillo, y de comunicarse con los otros LED de la cadena.

![Disposición de los pines y del controlador en cada LED](img/pines_neopixel.jpg){width=50% #fig:label3}

El *Smart Shift Register* utiliza un protocolo de comunicación específico llamado *One-Wire Protocol*, se usa para poder recibir datos del microcontrolador. Su funcionamiento se explica en el siguiente apartado. Dicho protocolo se basa en otro protocolo serie de bajo nivel que utiliza una sola línea de datos para transmitirlos entre el microcontrolador y cada LED.

Para enviar datos a cada LED utilizando cada *Smart Shift Register*, el microcontrolador envía una serie de pulsos, que el microcontrador del LED interpreta como una secuencia de comandos. Los comandos incluyen instrucciones para configurar el color y el brillo de cada LED individual, así como para configurar el tiempo y la sincronización de la cadena de LEDs.

![Animación sobre la disposición de los bits dentro del controlador de cada LED](img/onewire.png){width=70% #fig:label3}

Cada LED cuenta con 24 *Shift Registers* que se van rellenando según se vayan recibiendo datos del microcontrolador principal (ATTiny o ATmega). Cuando se reciben 24 bits, éstos se agrupan en 3 *data latches* que el controlador de cada LED interpreta para ajustar el color o brillo de cada LED (se utiliza un grupo de 8 bits para controlar el color rojo, otro grupo de 8 bits para el color verde y otro grupo de 8 bits para el color azul). De esta forma, los bits correspondientes a la configuración del siguiente LED de la cadena pasa por el registro de desplazamiento del anterior, que éste lo reenvía al siguiente LED hasta que los desplaza a los *data latches*. Cuando el registro de desplazamiento recibe la señal de reset, quiere decir que se pueden iluminar los leds al haber terminado el envío de datos de toda la cadena, se empezaría a partir de ese momento otra ronda de envío de datos. [@smsreg-video]



#### Protocolo One-Wire

El protocolo One-Wire (o también conocido como 1-Wire) es utilizado en la tira de LEDs circular de Neopixel para poder transmitir  los 1's o 0's lógicos a cada LEDs, ya que se cuenta con una única línea de datos. 

![Flujo de datos usando 1-Wire](img/onewire.png){width=50%}

El funcionamiento de este protocolo es el siguiente: para poder determinar el dato, se mide los tiempos entre cada flanco de subida y de bajada, para cada transmisión. Si el tiempo entre el flanco de subida y el flanco de bajada es mayor que el tiempo transcurrido hasta el siguiente flanco de subida, el resultado es un voltaje alto (TH) o un 1 lógico. Por el contrario, si el tiempo entre el primer flanco de subida y el siguiente flanco de bajada es menor que el tiempo transcurrido hasta el el siguiente flanco de subida, el resultado es un voltaje bajo (LW) o un 0 lógico. En la siguiente figura se ilustra este funcionamiento.

![Cálculo de cada dato según los flancos de subida o bajada en 1-Wire](img/1wireTHTL.png){width=50%}



### Display 7 segmentos

COMPLETAR



### Pantalla LCD OLED

Otro de los periféricos usado para el desarrollo del firmware, es una pantalla LCD tipo OLED, en el que se muestran cadenas de texto. Para el desarrollo del firmware correspondiente a este periférico, nos hemos basado en un proyecto que también hace uso de V-USB para implementar funcionalidad con una pantalla de las mismas características, en concreto para utilizarla como salida de la consola de puerto serie. [@attiny-oled]

![Pantalla OLED utilizada en el proyecto](img/LCD_OLED.png){width=40%}



### Sensor de temperatura

Completar



### Buzzer

Completar



### Placa Bee 2.0

COMPLETAR: Relación con LIN