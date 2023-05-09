<!-- Leave a blank line before the title -->

# Desarrollo del prototipo hardware

En este capítulo se describe con detalles a más bajo nivel sobre los periféricos y hardware utilizado para el desarrollo del proyecto, así como una breve introducción de la familia de microcontroladores de la que se ha hecho uso en el proyecto.




## Microcontroladores

En el desarrollo del proyecto, se han utilizado dos microcontroladores sobre los que se han probado el firmware. A continuación, se explica cada uno de ellos, así como una explicación general sobre el origen de esta familia de microcontroladores.



### Familia de microcontroladores AVR

Los microcontroladores AVR provienen del fabricante Atmel (actualmente perteneciente a Microchip Technology [@atmel-web]), están principalmente diseñados para el desarrollo de sistemas empotrados y aplicaciones relacionados con robótica, así como su uso a gran escala en entornos industriales. Su interfaz y diseño es relativamente sencillo, por lo que implementar aplicaciones que usen estos chips resulta realmente atractivo para programadores que quieran aprender a desarrollar software en el que se utilice, por ejemplo, el protocolo USB, entre otros muchos, contando con ello con un gran abanico de utilidades y periféricos.

Estos microcontroladores, entre los que se incluyen ATTiny85 y ATmega328p, se caracterizan por permitir implementaciones en las que se requieren pocas prestaciones, así como su bajo consumo energético, pero que a su vez ofrecen un gran rendimiento en proyectos o entornos de pruebas, como en el que se ha estado trabajando. El uso de una arquitectura RISC hace que sea una ventaja en el procesamiento de datos, aunque como veremos más adelante, en el ATTiny85 se cuenta con una limitación en cuanto a temporización, que en el caso del desarrollo de este proyecto ha sido un problema para determinados periféricos. Salvo esto último, la única complejidad que puede presentar el desarrollo de un firmware es la adquisición de los conocimientos propios de toda la API de USB. [@avr-products]

En cuanto a componentes hardware, estos chips cuentan con una memoria flash para el almacenamiento del firmware, una SRAM y, como habíamos mencionado antes para el caso del ATTiny85, cuentan también con temporizadores. Se puede hacer uso de un puerto serie (para el caso del ATmega) y, por ejemplo, un conversor ADC, para interactuar con dispositivos externos como un sensor de temperatura, y más componentes que se detallarán más adelante para los dos microchips que se han utilizado en este proyecto.



### ATTiny85

Este es el microcontrolador usado originalmente para el desarrolo del firmware con V-USB, en el que se ha implementado funcionalidad para controlar un anillo circular de leds junto a una pantalla LCD OLED. Debido a las limitaciones en cuanto a puertos y temporización que presenta, nos vimos obligados a buscar otras alternativas, entre ellas nos decantamos por el microchip ATmega328p que describiremos más adelante.

A continuación vamos a hablar de las características hardware de este chip, así como datos relacionados con su arquitectura. Al ser un chip bastante sencillo, está implementado con una arquitectura de 8 bits. Al igual que otros chips de la familia AVR, y en especial éste, cuenta con un bajo consumo al tener pocos pines de entrada/salida, solamente 6. Se cuenta con 8 kilobytes de memoria flash para almacenar el firmware, 512 bytes de EEPROM donde almacenamos el *bootloader*, y 512 bytes de SRAM.

Cuenta con 6 pines de I/O de propósito general (desde PB0 hasta PB5), una limitación bastante importante en el proyecto, ya que solamente se puede hacer uso de unos pocos periféricos conectados simultáneamente. Otro de los problemas encontrados en el desarrollo de este proyecto, tiene que ver con el oscilador interno, ya que resulta un problema en términos de temporización al ejecutar aplicaciones que requieran estar recibiendo datos de un dispositivo externo, en este caso se ha hecho uso de un sensor de temperatura. [@avr-attiny85] [@attiny85-datasheet]

![Chip ATTiny85](img/ATTiny85.jpg){width=30% #fig:label1}

La elección de este microcontrolador para el proyecto resultó originalmente ser una gran ventaja debido a la gran posibilidad de usos con el que se contaba, además de poder integrarlo con el entorno Arduido y con el que se realizaron distintas pruebas previas para entender el funcionamiento del bootloader de Micronucleus (originalmente cargado en el microchip y totalmente compatible con Arduino). Posteriormente, al desarrollar nuestro propio firmware basado en V-USB, decidimos dejar de usar este entorno para poder usar un bootloader propio de AVR, y de esta forma, mediante un programador, poder cargar el firmware V-USB dentro del ATTiny85. Así, dejamos de depender del entorno Arduino y pudimos trabajar en el desarrollo de V-USB puro, entendiendo toda la funcionalidad de este firmware y del propio protocolo USB a bajo nivel. Todo esto se explica en los siguientes apartados, cuando se hace uso del ATTiny85 con la placa Digispark que ya lo incorpora, por lo que utilizar Micronucleus con Arduino en Digispark nos resultó realmente sencillo.

![Integración del entorno Arduino con ATTiny85](img/attiny85_arduino.png){width=55% #fig:label2}

Un factor destacable de este microchip es el bajo consumo que requiere para funcionar, por lo que para futuras versiones del proyecto, se puede integrar una batería para demostrar que no necesita de una cantidad muy grande de energía y puede aprovechar al máximo su duración. Un ejemplo concreto podría ser una integración de batería y sensor de temperatura con pantalla OLED, para monitorizar la temperatura de manera constante (aunque con ATTiny85 el uso de un sensor de temperatura es limitado por lo comentado anteriormente de la temporización). [@attiny85-desc]

A continuación, se detalla el esquema de la configuración de cada uno de los pines para el chip.

![Detalle de los pines del ATTiny85](img/attiny85_pines.png){width=60% #fig:label3}



### ATmega328p

Tras estar trabajando con el ATTiny85, nos dimos cuenta de las limitaciones en cuanto a puertos de I/O y temporización que éste contaba. Por lo que decidimos buscar alternativas de chips dentro de la familia AVR, y nos llamó la atención este microchip al contar con un número bastante grande de puertos de entrada/salida, ya que cuenta con 23 pines. Además, utilizando el ATmega no contamos con las limitaciones de temporización (el ATmega cuenta con 3 *timers* o temporizadores), por lo que el uso de dispositivos que envíen datos al microchip (dispositivos de entrada) resulta muy adecuado al no presentar ninguna limitación que pueda provocar inestabilidad en el firmware o lectura de datos incorrecta.

![Chip ATmega](img/ATMEGA328P.jpg){width=55%}

A diferencia de los 512 KB del ATTiny85, éste cuenta con 1 KB de memoria EEPROM donde se almacena el bootloader, 2 KB de SRAM y, como hemos mencionado anteriormente, 23 pines I/O de propósito general, una de las grandes ventajas. 

Otros datos sobre el hardware son las 24 líneas de interrupción internas que tiene. Cuenta con una interfaz de puerto serie, dotándolo de funcionalidad en el firmware del proyecto para poder depurar errores en el código. Se hace uso conectando un cable mini-USB al host, y a través de una terminal puerto serie (en Linux, por ejemplo Minicom), se pueden ver los mensajes que lanza el firmware mediante la ejecución de funciones de depuración (similares a ```printf()```). Sobre el uso de esta forma de depuración en ATmega, se explica con detalle en el capítulo 5 de esta memoria.

El dispositivo puede operar entre 1.8 y 5.5 voltios, en función de distintos factores como el número de periféricos que contenga. Puede alcanzar una respuesta de 1 MIPS. [@nano-pinout]

![Placa Arduino NANO con ATmega328p](img/atmega_board.png){width=40%}





## Placas que utilizan estos microcontroladores

A continuación se detallan las dos placas sobre las que se han hecho pruebas en este proyecto, en las que se incluyen los microcontroladores mencionados anteriormente.



### Digispark con ATTiny85

Para el desarrollo original del firmware con el microchip ATTiny85, escogimos hacer uso de la placa Digispark, que además cuenta con una interfaz USB necesaria para el desarrollo de este proyecto.
Los detalles técnicos que cuenta Digispark son similiares al chip ATTiny, salvo la interfaz USB y los pines de entrada de corriente con los que cuenta. Puede funcionar con un voltaje de 5V.

![Pinout de la placa Digispark](img/Digispark_Pinout.png){width=55% #fig:label2}

Su tamaño de 25 mm x 18 mm, lo que resulta realmente manejable y portable. Incluye una interfaz USB para programación y comunicación con el host, así como un regulador de voltaje y LED de encendido. [@digispark-board]

![Placa de desarrollo Digispark](img/digispark_board.jpg){width=35% #fig:label2}

Una característica única de la placa Digispark es que viene preprogramada con un cargador de arranque que permite el flasheo de firmware a través del propio puerto USB usando el IDE de Arduino (llamado Micronucleous Bootloader). Con ello se puede programar código desde Arduino fácilmente a la placa sin necesidad de un programador independiente. Aunque para el desarrollo de nuestro proyecto, se ha optado por no depender de Arduino y utilizar la librería V-USB pura, entendiento todos los aspectos de bajo nivel del protocolo y comunicación USB, por lo que se requiere de un bootloader de AVR y un programador externo. [@setup-digispark]



### NANO con ATmega328p y puerto serie

Para el prototipo final del proyecto, hacemos uso de la placa Nano que incorpora el microchip ATmega328p. Las ventajas de utilizar esta placa en relación a Digispark son bastantes, además de poder contar con una interfaz puerto serie utilizada en el proyecto para depuración. En esta placa cabe destacar también el bajo consumo energético, y el amplio número de pines de E/S con el que se cuenta, en este caso 14. Una ventaja muy notable en comparación a la placa Digispark con ATTiny85.

![Pinout de la placa NANO con ATmega328p](img/nanopinout.png){width=50%}

Además de los pines de E/S mencionados anteriormente, contamos con 8 pines de entrada analógica y 6 pines PWM. Cuenta también con un puerto USB utilizado en este caso como salida de puerto serie (para depuración del firmware), y un botón de reset para reiniciar la placa. [@nano-info]

En cuanto a términos de programación, Nano es totalmente compatible con Arduino IDE, por lo que el desarrollo de pequeños y medianos proyectos utilizando esta herramienta hace que sea muy atractivo para programadores con poca experiencia. Por el contrario, nosotros desarrollamos y cargamos el firmware directamente sin utilizar este entorno, como se explica en otro apartados de esta memoria.

El uso de esta placa es una opción muy interesante para la creación de prototipos, ya que se puede integrar fácilmente en circuitos breadboard o perfboard, y poder realizar pruebas como las que se realizaron en este proyecto para implementar la interfaz USB, usando distintos diodos que posibilitaran el funcionamiento de este puerto.





## Periféricos con soporte en el proyecto

En este apartado se explica los distintos elementos hardware con los que se ha dado soporte en el proyecto. En este proyecto se ha contado con una placa inicial, la placa Bee 2.0, utilizada en la asignatura Arquitectura interna de Linux y Android, en la que se cuentan con distintos periféricos, como un display 7 segmentos o un buzzer, permitiendo la interconexión entre todas sus componentes. También se ha hecho uso de un anillo led circular y de una pantalla OLED.



### Anillo de LEDs de colores

Para probar el firmware, en la primera versión con ATTiny85 se ha hecho uso de la tira de leds circular fabricado por la empresa NeoPixel [@LED-strip]. Dichos leds se encuentran en disposición circular con colores RGB direccionables individualmente. Posteriormente, se han utilizado como un periférico para el prototipo final de este proyecto, utilizando ATmega328p.

![Tira de LEDs circular de NeoPixel](img/LED-Strip.jpg){width=45% #fig:label3}

Estos LEDs están conectados en cadena, y cada LED está controlado por una sola línea de datos. La línea de datos usa un protocolo de comunicación específico llamado *One-Wire Protocol*, es un protocolo software que controla el color y el brillo de cada LED individual. Su funcionamiento se explica en los siguientes apartados. De esta forma, conseguimos hacer iluminar todos los leds del anillo a la vez, o crear una serie de patrones para que se vayan encendiendo poco a poco, con distintos colores. 

La tira de LED circular se alimenta con una fuente de alimentación de 5V que proporciona la placa Digispark o Nano. Está dirigida por un microcontrolador, en nuestro caso originalmente el ATTiny85, posteriormente se usa el ATmega328p.



#### Smart Shift Registers en los LEDs de NeoPixel

Cada LED individual en la tira de LEDs circular de NeoPixel contiene un registro, con un pequeño microcontrolador y un LED RGB (Red, Green, Blue). El microcontrolador de cada LED es el responsable de controlar el color y el brillo, y de comunicarse con los otros LED de la cadena.

![Disposición de los pines y del controlador en cada LED](img/pines_neopixel.jpg){width=50% #fig:label3}

El *Smart Shift Register* utiliza un protocolo de comunicación específico llamado *One-Wire Protocol*, se usa para poder recibir datos del microcontrolador. Su funcionamiento se explica en el siguiente apartado. 

Dicho protocolo se basa en otro de más bajo nivel que hace uso de una sola línea de datos, para transmitir estos bits entre el microcontrolador y cada LED. Para enviar datos a cada LED utilizando cada registro, el microcontrolador envía una serie de bits, que el microcontrador del LED interpreta como una secuencia de comandos. Los comandos incluyen instrucciones para configurar el color y el brillo de cada LED individual, así como para configurar el tiempo y la sincronización de la cadena de LEDs.

![Animación sobre la disposición de los bits dentro del controlador de cada LED](img/onewire.png){width=70% #fig:label3}

Cada LED cuenta con 24 *Shift Registers* que se van rellenando según se vayan recibiendo datos del microcontrolador principal (ATTiny o ATmega). Cuando se reciben 24 bits, éstos se agrupan en 3 *data latches* que el controlador de cada LED interpreta para ajustar el color o brillo de cada LED (se utiliza un grupo de 8 bits para controlar el color rojo, otro grupo de 8 bits para el color verde y otro grupo de 8 bits para el color azul). De esta forma, los bits correspondientes a la configuración del siguiente LED de la cadena pasa por el registro de desplazamiento anterior, que éste lo reenvía al siguiente hasta que se desplazan a los *data latches*. Cuando el registro de desplazamiento recibe la señal de reset, quiere decir que se pueden iluminar los LEDs al haber terminado el envío de datos de toda la cadena, se empezaría a partir de ese momento otra ronda de envío de datos. [@smsreg-video]



#### Protocolo One-Wire

El protocolo One-Wire (o también conocido como 1-Wire) es utilizado en la tira de LEDs circular de Neopixel para poder transmitir  los 1's o 0's lógicos a cada LEDs, ya que se cuenta con una única línea de datos. 

![Flujo de datos usando 1-Wire](img/onewire.png){width=50%}

El funcionamiento de este protocolo es el siguiente: para poder determinar el dato, se mide los tiempos entre cada flanco de subida y de bajada, para cada transmisión. Si el tiempo entre el flanco de subida y el flanco de bajada es mayor que el tiempo transcurrido hasta el siguiente flanco de subida, el resultado es un voltaje alto (TH) o un 1 lógico. Por el contrario, si el tiempo entre el primer flanco de subida y el siguiente flanco de bajada es menor que el tiempo transcurrido hasta el el siguiente flanco de subida, el resultado es un voltaje bajo (LW) o un 0 lógico. En la siguiente figura se ilustra este funcionamiento.

![Cálculo de cada dato según los flancos de subida o bajada en 1-Wire](img/1wireTHTL.png){width=50%}



### Display 7 segmentos

Este periférico con soporte en el firmware es capaz de representar un número del 0 al 9, mediante segmentos verticales y horizontales que, según el número a mostrar, se encienden o apagan mostrando el número en cuestión. También cuenta con un punto a la derecha de los segmentos. La configuración de este display es de cátodo común, es decir, cuando se escribe un 1 se enciende el segmento deseado, y cuando se escribe un 0, se apaga.

![Diagrama del circuito del display 7 segmentos de la placa Bee 2.0](img/d7seglin.png){width=50%}

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

[COMPLETAR explicación I2C]

En los siguientes apartados se explica la funcionalidad implementada en el firmware y también los distintos drivers que hacen uso de este dispositivo.



### Buzzer

Este buzzer utiliza una señal PWM para su funcionamiento. Cuando se recibe un 1 por dicho pin, éste empieza a emitir un sonido constante, cuando se deja de transmitir, no se emite ningún sonido. La explicación sobre la implementación de la funcionalidad de este periférico en el firmware y drivers se explica en los siguientes apartados de la memoria. 

Para poder entender el funcionamiento de este periférico, es preciso entender cómo funciona la señal PWM (Pulse Width Modulation, en castellano, modulación por ancho de pulso). El objetivo es convertir una señal digital (un 1 lógico) en una señal analógica, lo que se traduce en sonido para el Buzzer (una determinada frecuencia, generalmente entre los 2 KHz ~ 5 KHz). Para ello, se hace pasar dicha corriente por una componente interna que es la que produce el sonido.



### Placa Bee 2.0

Como punto de partida en el proyecto, se ha estudiado el funcionamiento e interconexión de los distintos elementos hardware de la placa Bee 2.0 diseñada para la asignatura Arquitectura Interna de Linux y Android.

![Placa Bee 2.0 de la asignatura Arquitectura interna de Linux y Android](img/bee20gen.png){width=40%}

La idea de este proyecto es la posible fusión del uso de la librería V-USB con esta asignatura, así como el diseño de prácticas relacionadas con el uso de este firmware, para que la interacción y funcionalidad con los elementos hardware de la placa sea mucho más variada, y que la modificación de funcionalidades a implementar por los alumnos sea cada vez de una forma distinta, lo que ayuda a la adquisición de los conocimientos de bajo nivel sobre el funcionamiento, entre otras cosas, del protocolo USB.