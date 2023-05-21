<!-- Leave a blank line before the title -->

# Desarrollo del prototipo hardware { #cap:hw }

En este capítulo se describen con detalles a bajo nivel los microcontroladores y periféricos utilizados para el desarrollo del proyecto, así como su arquitectura interna. Previamente se hace una introducción de los microcontroladores en general, con una breve descripción de sus componentes. 




## Microcontroladores

Los microcontroladores son pequeños dispositivos utilizados en sistemas empotrados para realizar tareas específicas, contando con todos los elementos hardware necesarios en un solo chip, como la memoria o el procesador, así como otros componentes esenciales, por ejemplo puertos de E/S. Los microcontroladores cuentan con un núcleo central de procesamiento, la CPU, que puede tener una arquitectura de 8, 16 o 32 bits. Se encarga de realizar todas las operaciones de procesamiento lógico, así como ejecutar instrucciones almacenadas en su memoria. Relacionado con la memoria del chip, podemos distinguir dos categorías:

- **Memoria del programa** (ROM/Flash): almacena código ejecutable, es decir, el firmware, que contiene las instrucciones que el microcontrolador debe seguir para realizar su tarea. Pueden ser de varios tipos: ROM, el código de esta memoria no se puede cambiar, únicamente puede grabarse durante el proceso de fabricación. Las memorias Flash, por el contrario, sí pueden reprogramarse posteriormente.

- **Memoria de datos**: se utiliza para almacenar datos y otras variables temporales durante la ejecución del programa. A su vez podemos distinguir varios tipos, entre los cuales destacamos los siguientes:

  a) RAM (Random Access Memory): Este tipo de memoria es volátil, es decir, se borra cuando el microcontrolador se apaga. La RAM se utiliza para incrementar la velocidad en la ejecución del programa, ya que almacena datos que requieren ser accedidos y modificados rápidamente. 

  b) EEPROM (Electrically Erasable Programmable Read-Only Memory). A diferencia de la memoria RAM, ésta no pierde información aunque interrumpa la corriente. La información en esta memoria se escribe mediante pulsos eléctricos especiales, que gestiona el microcontrolador. 

Otros de los elementos de un chip son los puertos de entrada/salida (GPIOs). Estos puertos permiten al microcontrolador comunicarse con dispositivos externos, ya sean de entrada o salida, como botones, actuadores o pantallas. Además, un chip puede incluir módulos especiales como conversores analógico-digital (ADC), o interfaces de comunicación tipo UART, SPI, I2C, etc. También suelen tener controladores de temporización, utilizados para la gestión de estos dispositivos E/S. [@chip-description]

A continuación, se presenta una breve introducción de la familia de controladores AVR, seguida de una descripción detallada de cada uno de los microcontroladores utilizados en el proyecto. 



### Familia de microcontroladores AVR

Los microcontroladores AVR provienen del fabricante Atmel (actualmente perteneciente a Microchip Technology [@atmel-web]), son chips que cuentan con un gran rendimiento en entornos con pocas prestaciones. Su interfaz y diseño es relativamente sencillo, por lo que implementar aplicaciones que usen estos chips resulta atractivo para programadores que quieran iniciarse en el desarrollo software utilizando protocolos más complejos, como puede ser USB. Su desarrollo en sistemas empotrados puede variar desde pequeñas aplicaciones relacionadas con robótica, hasta su uso en entornos industriales.

Los componentes más importantes de esta familia de controladores son los siguientes:

- **CPU**: cada chip de esta familia de microcontroladores cuenta con un núcleo central de procesamiento con arquitectura tipo RISC de 8 bits, permitiendo operaciones con una frecuencia de reloj desde los 8 MHz, 16 MHz, hasta los 20 MHz. 
- **Memorias**: para almacenar el código del firmware, éstos microcontroladores utilizan una memoria Flash reprogramable. Para el almacenamiento de datos, utilizan una memoria RAM, permitiendo el acceso rápido a datos y variables temporales en tiempo de ejecución. 
- **Registros de propósito general**: utilizados para el procesamiento de operaciones aritméticas y lógicas, así como el almacenamiento de datos temporales. Suelen ser de 8 bits.
- **Periféricos integrados**: dependiendo del tipo de microcontrolador, puede contar con un número determinado de puertos de E/S de propósito general (GPIO), así como un puerto para la comunicación I2C, SPI, o temporizadores/contadores. Los microcontroladores más avanzados pueden tener generadores PWM (modulación por ancho de pulso) o conversores ADC (analógico - digital), entre otros.
- **Sistema de interrupciones y oscilador**: permite responder a eventos externos (si se hace uso, por ejemplo, de periféricos de entrada), y cuentan con un oscilador para la sincronización de las operaciones que se están ejecutando.

Esta familia de microcontroladores, entre los que se incluyen ATTiny85 y ATmega328p utilizados en este proyecto, se caracterizan también por su relativo bajo coste, y su facilidad de adquisición [@avr-products]. A continuación, se explica con más detalle cada uno de estos microcontroladores en concreto.



### ATTiny85

Este microcontrolador se ha utilizado para el desarrollo del primer prototipo del proyecto. A continuación, se describen cada uno de sus componentes hardware:

- **CPU**: Este microcontrolador cuenta con una arquitectura RISC de 8 bits (característico de los microcontroladores AVR), a una velocidad de reloj de hasta 20MHz (operando con un voltaje de, como máximo, 5,5V).
- **Memoria**: El chip ATTiny85 cuenta con una memoria Flash de 8KB para almacenar el firmware, y una memoria de datos tipo RAM de 512 bytes.
- **Registros de propósito general y especiales**: Este chip cuenta con 32 registros de propósito general, cada uno con 8 bits. También incluye registros especiales, como registros de estado (SREG) e interrupciones. 
- **Periféricos**: Cuenta con 6 pines configurables para E/S de propósito general (GPIOs). También cuenta con dos temporizadores (Timers) de 8 bits, y 4 salidas de PWM de 8 bits para general señales analógicas. Una característica a mencionar es que tiene soporte para comunicación serie, a través de un módulo USART (Universal Synchronous/Asynchronous Receiver/Transmitter).
- **Interrupciones y oscilador interno**: Este chip tiene soporte para interrupciones sobre eventos externos, también cuenta con un oscilador (calibrado de fábrica) de 8 MHz. 

Un factor a destacar de este microchip es el bajo consumo que requiere, por lo que la inclusión de una batería en el sistema puede demostrar su bajo consumo, y además puede aprovechar al máximo la duración de la batería. A continuación, se detalla el esquema con la posición de cada pin en el ATTiny85. [@avr-attiny85] [@attiny85-datasheet] [@attiny85-desc]

![Pinout ATTiny85 [@pinoutatiny-page]](img/attiny85_pines.png){width=90% #fig:label3}

La placa elegida que integra este microchip es la placa Digispark [@digispark-board]. Su tamaño de 25 mm x 18 mm, incorpora un regulador de voltaje y un LED de encendido. Está basada en el microcontrolador ATTiny85, cuenta para ello con una interfaz USB. Una característica importante de esta placa es que viene preprogramada con un cargador de arranque que permite programar firmware directamente al chip a través del propio puerto USB usando el IDE de Arduino (llamado Micronucleous, basado en USB HID). Con ello se puede programar código desde el entorno de Arduino fácilmente a la placa sin necesidad de un programador independiente [@setup-digispark]. A continuación se muestran la disposición de los pines en la placa Digispark:

![Pinout de la placa Digispark [@digisparkpin-image]](img/Digispark_Pinout.png){width=55% #fig:label2}



### ATmega328p

Para el prototipo final de este proyecto, se ha optado por el eso de este microcontrolador, que es más potente y no por ello más caro. A continuación se describen sus componentes hardware:

- **CPU**: Este chip cuenta con una unidad central de procesamiento con arquitectura RISC de 8 bits, y una frecuencia de reloj de hasta 20MHz. (Características similares al ATTiny85, operando con un voltaje de hasta 5,5V).
- **Memoria**: Cuenta con 32 KB de memoria Flash, 2 KB de RAM y 1 KB de EEPROM, esta última para el almacenamiento de datos  no volátiles.
- **Registros de propósito general**: El ATmega328p tiene un conjunto de 32 registros de propósito general de 8 bits, utilizados para transferencia de datos entre registros y memoria, y operaciones aritméticas.
- **Periféricos**: En cuanto a pines de E/S, este chip cuenta con un número bastante mayor que el ATTiny85, con 14. También incorpora 3 timers, y se da soporte a los siguientes protocolos de comunicación: UART (Universal Asynchronous Receiver/Transmitter), SPI (Serial Peripheral Interface) y I2C (Inter-Integrated Circuit).
- **Interrupciones y oscilador interno**: El ATmega328p admite interrupciones externas y gestionadas por software. Cuenta también con un oscilador funcionando a una frecuencia base de 8MHz, ajustable mediante bits de configuración de fusibles.

La ventaja de usar este microchip es el mayor número de pines de E/S que posee, además de poder ajustar la frecuencia de reloj mediante software, ya que éste fue un problema en el desarrollo del firmware utilizando ATTiny85. A continuación, se muestra la posición de los pines en el chip:

![Pinout ATmega328p [@atmegapinout-image]](img/atmega_pinout.png){width=55%}

Para el prototipo final del proyecto, hacemos uso de la placa NANO que incorpora el microchip ATmega328p. Las ventajas de utilizar esta placa en relación a Digispark son bastantes, además de poder contar con una interfaz puerto serie utilizada en el proyecto para depuración. En esta placa cabe destacar también el bajo consumo energético, y el amplio número de pines de E/S con el que se cuenta, en este caso 14. Una ventaja muy notable en comparación a la placa Digispark con ATTiny85. Además de los pines de E/S mencionados anteriormente, contamos con 8 pines de entrada analógica y 6 pines PWM. Cuenta también con un puerto USB utilizado en este proyecto como salida de puerto serie (para depuración del firmware), y un botón de reset para reiniciar la placa [@nano-info]. El uso de esta placa es una opción muy interesante para la creación de prototipos, ya que se puede integrar fácilmente en circuitos breadboard o perfboard. A continuación se muestra la disposición de los pines en la placa NANO: [@nano-pinout]

![Pinout de la placa NANO [@nanopinout-image]](img/nanopinout.png){width=70%}



## Periféricos soportados por el proyecto

En este apartado se describen los distintos elementos hardware a los que se ha dado soporte en el proyecto. Se ha partido de una placa inicial, la placa Bee 2.0, utilizada en la asignatura LIN, que cuenta con distintos periféricos, entre ellos un display 7 segmentos o un buzzer. También se ha hecho uso de un anillo led circular y de una pantalla OLED, externos a esta placa.




### Placa Bee 2.0

[WORK IN PROGRESS] Como punto de partida en el proyecto, se ha estudiado el funcionamiento e interconexión de los distintos elementos hardware de la placa Bee 2.0 diseñada para la asignatura Arquitectura Interna de Linux y Android.

![Placa Bee 2.0 de la asignatura Arquitectura interna de Linux y Android [@bee-board]](img/bee20gen.png){width=70%}

La idea de este proyecto es la posible fusión del uso de la librería V-USB con esta asignatura, así como el diseño de prácticas relacionadas con el uso de este firmware, para que la funcionalidad e interacción con los elementos hardware de la placa sea mucho más variada, y que la modificación de la implementación software por parte de los alumnos sea bastante más amplia que con la que se cuenta actualmente con el Blinkstick, lo que ayudaría a la adquisición de los conocimientos de bajo nivel sobre el funcionamiento, entre otras cosas, del protocolo USB.




### Anillo de LEDs de colores

Para probar el firmware, en una primera versión con ATTiny85 se ha hecho uso de la tira de LEDs circular fabricado por la empresa NeoPixel [@LED-strip]. Dichos LEDs se encuentran en disposición circular con colores RGB direccionables individualmente. Posteriormente, ha sido incorporado como periférico para el prototipo final de este proyecto, utilizando ATmega328p.

![Tira de LEDs circular de NeoPixel](img/LED-Strip.jpg){width=75% #fig:label3}

Estos LEDs están conectados en serie. Todos los LEDs cuentan con un registro y un pequeño microcontrolador, y para su configuración se utiliza una sola línea de datos. Esta línea de datos usa un protocolo de comunicación específico llamado *One-Wire Protocol*, es un protocolo software que configura el color y el brillo de cada LED individual. Su funcionamiento se explica en los siguientes apartados. De esta forma, conseguimos hacer iluminar todos los LEDs del anillo a la vez, o crear una serie de patrones para que se vayan encendiendo poco a poco, con distintos colores. 

La tira circular de LEDs se alimenta con una fuente de alimentación de 5V que proporciona la placa Digispark o Nano. Está gestionada por un microcontrolador, en nuestro caso originalmente el ATTiny85, posteriormente se usa el ATmega328p.



#### Smart Shift Registers en los LEDs de NeoPixel

Cada LED individual en la tira de LEDs circular de NeoPixel contiene un registro, con un pequeño microcontrolador y un LED RGB (Red, Green, Blue). El microcontrolador de cada LED es el responsable de controlar el color y el brillo, y de comunicarse con los otros LED de la cadena.

![Disposición de los pines y del controlador en cada LED [@smsreg-video]](img/pines_neopixel.jpg){width=50% #fig:label3}

El *Smart Shift Register* utiliza un protocolo de comunicación específico llamado *One-Wire Protocol*, se usa para poder recibir datos del microcontrolador. Su funcionamiento se explica en el siguiente apartado. 

Dicho protocolo se basa en otro de más bajo nivel que hace uso de una sola línea de datos, para transmitir estos bits entre el microcontrolador y cada LED. Para enviar datos a cada LED utilizando cada registro, el microcontrolador envía una serie de bits, que el microcontrador del LED interpreta como una secuencia de comandos. Los comandos incluyen instrucciones para configurar el color y el brillo de cada LED individual, así como para configurar el tiempo y la sincronización de la cadena de LEDs.

![Posición de los bits dentro del controlador de cada LED [@smsreg-video]](img/onewire.png){width=70% #fig:label3}

Cada LED cuenta con 24 *Shift Registers* que se van rellenando según se vayan recibiendo datos del microcontrolador principal (ATTiny o ATmega). Cuando se reciben 24 bits, éstos se agrupan en 3 *data latches* que el controlador de cada LED interpreta para ajustar el color o brillo de cada LED (se utiliza un grupo de 8 bits para controlar el color rojo, otro grupo de 8 bits para el color verde y otro grupo de 8 bits para el color azul). De esta forma, los bits correspondientes a la configuración del siguiente LED de la cadena pasa por el registro de desplazamiento anterior, que éste lo reenvía al siguiente hasta que se desplazan a los *data latches*. Cuando el registro de desplazamiento recibe la señal de reset, quiere decir que se pueden iluminar los LEDs al haber terminado el envío de datos de toda la cadena, se empezaría a partir de ese momento otra ronda de envío de datos. [@smsreg-video]



#### Protocolo One-Wire

El protocolo One-Wire (o también conocido como 1-Wire) es utilizado en la tira de LEDs circular de Neopixel para poder transmitir  los 1's o 0's lógicos a cada LED, ya que se cuenta con una única línea de datos. 

El funcionamiento de este protocolo es el siguiente: para poder determinar el dato, se mide los tiempos entre cada flanco de subida y de bajada, para cada transmisión. Si el tiempo entre el flanco de subida y el flanco de bajada es mayor que el tiempo transcurrido hasta el siguiente flanco de subida, el resultado es un voltaje alto (TH) o un 1 lógico. Por el contrario, si el tiempo entre el primer flanco de subida y el siguiente flanco de bajada es menor que el tiempo transcurrido hasta el el siguiente flanco de subida, el resultado es un voltaje bajo (LW) o un 0 lógico. En la siguiente figura se ilustra este funcionamiento.

![Cálculo de cada dato según los flancos de subida o bajada en 1-Wire [@smsreg-video]](img/1wireTHTL.png){width=50%}



### Display 7 segmentos

Este periférico con soporte en el firmware es capaz de representar un número del 0 al 9, mediante segmentos verticales y horizontales que, según el número a mostrar, se encienden o apagan mostrando el número en cuestión. También cuenta con un punto a la derecha de los segmentos. La configuración de este display es de cátodo común, es decir, cuando se escribe un 1 se enciende el segmento deseado, y cuando se escribe un 0, se apaga.

![Diagrama del circuito del display 7 segmentos de la placa Bee 2.0 [@sevenseg]](img/d7seglin.png){width=50%}

El diagrama anterior ilustra el circuito interno del display 7 segmentos. Se puede observar que se cuenta con dos registros, y para ello el sistema tiene 3 pines de entrada. Esto es una enorme ventaja, ya que si tuviéramos 8 pines de entrada para el display, a parte de la complejidad en el código que se añadiría, limitaría la posible interacción con otros dispositivos de E/S, al tener que estar constantemente enviando un 1 lógico en aquellos pines correspondientes a los segmentos que se quieren encender. Para ello, se ha implementado el sistema con dos registros, uno de desplazamiento y otro de salida, que se explican a continuación.

Entradas del registro de desplazamiento: 

- **SDI** (Serial Data Input): es la entrada serie del registro, de 8 bits. Cada bit corresponde a un segmento. El bit menos significativo corresponde al punto.
- **SRCLK** (Shift Register Clock): señal del reloj correspondiente al registro de desplazamiento. 
- **Enable**. Esta señal debe estar a 1 para que el sistema realice la acción de desplazamiento y carga paralela, mediante los dos registros.

Entradas del registro de salida:

- **RCLK** (Register Clock): señal del reloj correspondiente al registro de salida.
- **Load**. Al igual que la señal de enable, tiene que estar a 1 para el sistema realice las acciones de desplazamiento y carga paralela.

Para poder generar la salida deseada, con establecer un 1 en la señal de RCLK (que debe estar a 0 durante todo el proceso previo), se cargaría inmediatamente el número en el registro, y por lo tanto, en la salida de los segmentos del display. [@bee-board]



### Pantalla LCD OLED

Otro de los periféricos con soporte en el firmware del proyecto es una pantalla LCD tipo OLED, en el que se muestran cadenas de texto. Para el desarrollo de la funcionalidad correspondiente a este periférico, nos hemos basado en otro proyecto que también hace uso de V-USB para implementar funcionalidad con una pantalla de las mismas características, en concreto para utilizarla para mostrar mensajes que se reciben a través del protocolo I2C. [@attiny-oled]

![Pantalla OLED utilizada en el proyecto](img/LCD_OLED.png){width=40%}

Para la conexión del periférico con el microcontrolador, a través de las placas Digispark o NANO, se hace a través de los pines de I2C de ambas placas, en concreto a través de los pines de SDA y SCL.

En los siguientes apartados se explica la funcionalidad implementada en el firmware y también los distintos drivers que hacen uso de este dispositivo. Este periférico utiliza el ReportID 3 del firmware para su funcionamiento.



### Buzzer

Este buzzer utiliza una señal PWM para su funcionamiento. Cuando se recibe un 1 por dicho pin, éste empieza a emitir un sonido constante, cuando se deja de transmitir, no se emite ningún sonido. La explicación sobre la implementación de la funcionalidad de este periférico en el firmware y drivers se explica en los siguientes apartados de la memoria. 

Para poder entender el funcionamiento de este periférico, es preciso entender cómo funciona la señal PWM (Pulse Width Modulation, en castellano, modulación por ancho de pulso). El objetivo es convertir una señal digital (un 1 lógico) en una señal analógica, lo que se traduce en sonido para el Buzzer (una determinada frecuencia, generalmente entre los 2 KHz ~ 5 KHz). Para ello, se hace pasar dicha corriente por una componente interna que es la que produce el sonido.