<!-- Leave a blank line before the title -->

# Desarrollo del prototipo hardware { #cap:hw }

En este capítulo se describen en detalle los microcontroladores y periféricos utilizados para el desarrollo del proyecto, así como su arquitectura interna. Previamente se hace una introducción de los microcontroladores en general, con una breve descripción de sus componentes. 




## Microcontroladores

Los microcontroladores son pequeños dispositivos fabricados para realizar tareas específicas, contando con todos los elementos hardware necesarios en el mismo chip, como la memoria o el procesador, así como otros componentes esenciales, por ejemplo puertos de E/S. Los microcontroladores cuentan con un núcleo central de procesamiento, que denominamos CPU, implementada con una arquitectura de 8, 16 o 32 bits. Se encarga de realizar todas las operaciones de procesamiento lógico, así como ejecutar instrucciones almacenadas en su memoria. Relacionado con la memoria del chip, podemos distinguir dos categorías:

- **Memoria del programa** (ROM/Flash): almacena código ejecutable, es decir, el firmware, que contiene las instrucciones que el microcontrolador debe seguir para realizar su tarea. Existen varios tipos: las memorias ROM, las cuales no permiten modificar su contenido con una escritura normal, son programadas durante el proceso de fabricación. Las memorias Flash, por el contrario, sí pueden reprogramarse posteriormente. Existe un tipo especial de memoria ROM, la EEPROM, que permiten ser modificadas mediante un procedimiento especial.

  Para poder ejecutar el código de la memoria Flash, se sigue un proceso en el que se ve involucrado otro tipo de memoria volátil: la DRAM. Antes de lanzar el código del programa principal (residente en la memoria Flash) a ejecución, se carga un tipo de programa que inicializa el sistema y lo prepara para ejecutar el código principal, llamado *bootloader*. El código correspondiente al *bootloader* reside en una región específica de la memoria Flash. Posteriormente, se carga el código del programa principal a la DRAM, tarea que ejecuta también el *bootloader*. Una vez cargado, éste transfiere el control del procesador al programa principal, ejecutándose a partir de ahora desde la DRAM.

- **Memoria de datos**: se utiliza para almacenar datos y otras variables temporales durante la ejecución del programa. A su vez podemos distinguir varios tipos, entre los cuales destacamos los siguientes:

  a) RAM (Random Access Memory): Este tipo de memoria es volátil, es decir, se borra cuando el microcontrolador se apaga. La RAM se utiliza para incrementar la velocidad en la ejecución del programa, ya que almacena datos que requieren ser accedidos y modificados rápidamente. 

  b) EEPROM (Electrically Erasable Programmable Read-Only Memory). A diferencia de la memoria RAM, ésta no pierde información aunque se interrumpa la corriente. La información de esta memoria se escribe mediante pulsos eléctricos especiales, que gestiona el microcontrolador. 

Otros de los elementos de un chip son los puertos de entrada/salida (GPIOs). Estos puertos permiten al microcontrolador comunicarse con dispositivos externos, ya sean de entrada o salida, como botones, actuadores o pantallas. Además, un chip puede incluir módulos especiales como conversores analógico-digital (ADC) y PWM, o interfaces de comunicación tipo UART, SPI, I2C, etc. También suelen tener controladores de temporización, utilizados para la gestión de estos dispositivos E/S. [@chip-description]

A continuación, se presenta una breve introducción de la familia de controladores AVR, seguida de una descripción detallada de cada uno de los microcontroladores utilizados en el proyecto. 



### Familia de microcontroladores AVR

Los microcontroladores AVR provienen del fabricante Atmel (actualmente perteneciente a Microchip Technology [@atmel-web]), son chips con pocas prestaciones, bajo costo, bajo consumo y buen rendimiento. Su interfaz y diseño es relativamente sencillo, por lo que implementar aplicaciones que usen estos chips resulta atractivo para programadores que quieran iniciarse en el desarrollo software utilizando protocolos más complejos, como puede ser USB. Su desarrollo en sistemas empotrados puede variar desde pequeñas aplicaciones relacionadas con robótica, hasta su uso en entornos industriales.

Los componentes más importantes de esta familia de controladores son los siguientes:

- **CPU**: cada chip de esta familia de microcontroladores cuenta con un núcleo central de procesamiento con arquitectura tipo RISC de 8 bits, permitiendo operaciones con una frecuencia de reloj desde los 8 MHz, 16 MHz, hasta los 20 MHz. 
- **Memorias**: para almacenar el código del firmware, éstos microcontroladores utilizan una memoria Flash reprogramable. Para el almacenamiento de datos, utilizan una memoria RAM, permitiendo el acceso rápido a datos y variables temporales en tiempo de ejecución. 
- **Registros de propósito general**: utilizados para el procesamiento de operaciones aritméticas y lógicas, así como el almacenamiento de datos temporales. Suelen ser de 8 bits.
- **Periféricos integrados**: dependiendo del tipo de microcontrolador, puede contar con un número determinado de puertos de E/S de propósito general (GPIO), así como un puerto para la comunicación I2C, SPI, o temporizadores/contadores. Los microcontroladores más avanzados pueden tener generadores PWM (modulación por ancho de pulso) o conversores ADC (analógico - digital), entre otros.
- **Sistema de interrupciones y oscilador**: permite responder a eventos externos (si se hace uso, por ejemplo, de periféricos de entrada), y cuentan con un oscilador para la sincronización de las operaciones que se están ejecutando.

Esta familia de microcontroladores, entre los que se incluyen ATTiny85 y ATmega328p utilizados en este proyecto, se caracterizan también por su relativo bajo coste, y su facilidad de adquisición [@avr-products]. A continuación, se explica con más detalle cada uno de estos microcontroladores.



### ATTiny85

Este microcontrolador se ha utilizado para el desarrollo del primer prototipo del proyecto. A continuación, se describen cada uno de sus componentes hardware:

- **CPU**: Este microcontrolador cuenta con una arquitectura RISC de 8 bits (característico de los microcontroladores AVR), a una velocidad de reloj de hasta 20MHz (operando con un voltaje de, como máximo, 5,5V).
- **Memoria**: El chip ATTiny85 cuenta con una memoria Flash de 8KB para almacenar el firmware, y una memoria de datos tipo RAM de 512 bytes.
- **Registros de propósito general y especiales**: Este chip cuenta con 32 registros de propósito general, cada uno con 8 bits. También incluye registros especiales, como registros de estado (SREG) e interrupciones. 
- **Periféricos**: Cuenta con 6 pines configurables para E/S de propósito general (GPIOs). También cuenta con dos temporizadores (Timers) de 8 bits, y 4 salidas de PWM de 8 bits para generar señales analógicas. Una característica a mencionar es que tiene soporte para comunicación serie, a través de un módulo USART (Universal Synchronous/Asynchronous Receiver/Transmitter).
- **Interrupciones y oscilador interno**: Este chip tiene soporte para interrupciones sobre eventos externos, también cuenta con un oscilador (calibrado de fábrica) de 8 MHz. 

Un factor destacable de este microchip es que necesita muy poca energía para funcionar, por lo que diseñar un sistema en el que se haga uso de una batería puede permitirnos aprovechar al máximo su duración. La figura \ref{fig:label41} detalla el esquema con la posición de cada pin en el ATTiny85. [@avr-attiny85] [@attiny85-datasheet] [@attiny85-desc]

![Pinout ATTiny85 [@pinoutatiny-page]](img/attiny85_pines.png){width=90% #fig:label41}

La placa elegida que integra este microchip es la placa Digispark [@digispark-board]. Su tamaño de 25 mm x 18 mm, incorpora un regulador de voltaje y un LED de encendido. Está basada en el microcontrolador ATTiny85, y cuenta con una interfaz USB. Una característica importante de esta placa es que viene preprogramada con un cargador de arranque que permite programar firmware directamente al chip a través del propio puerto USB usando el IDE de Arduino (llamado Micronucleous, basado en USB HID). Con ello se puede programar código desde el entorno de Arduino fácilmente a la placa sin necesidad de un programador independiente [@setup-digispark]. En la figura \ref{fig:label42} se muestran la disposición de los pines en la placa Digispark.

![Pinout de la placa Digispark [@digisparkpin-image]](img/Digispark_Pinout.png){width=55% #fig:label42}



### ATmega328p {#sec:nano}

Para el prototipo final de este proyecto, se ha optado por el uso del microcontrolador ATmega328p, que es más potente y no por ello más caro. A continuación se describen sus componentes hardware:

- **CPU**: Este chip cuenta con una unidad central de procesamiento con arquitectura RISC de 8 bits, y una frecuencia de reloj de hasta 20MHz. (Características similares al ATTiny85, operando con un voltaje de hasta 5,5V).
- **Memoria**: Cuenta con 32 KB de memoria Flash, 2 KB de RAM y 1 KB de EEPROM, esta última para el almacenamiento no volátil de datos.
- **Registros de propósito general**: El ATmega328p tiene un conjunto de 32 registros de propósito general de 8 bits, utilizados para transferencia de datos entre registros y memoria, y operaciones aritméticas.
- **Periféricos**: Este chip cuenta con 14 pines configurables de E/S. También incorpora 3 timers, y se da soporte a los siguientes protocolos de comunicación: UART (Universal Asynchronous Receiver/Transmitter), SPI (Serial Peripheral Interface) y I2C (Inter-Integrated Circuit).
- **Interrupciones y oscilador interno**: El ATmega328p admite interrupciones externas gestionadas por software. Cuenta también con un oscilador funcionando a una frecuencia base de 8MHz, ajustable mediante bits de configuración de fusibles.

Para nuestro proyecto, las dos ventajas fundamentales del ATmega328p frente al ATiny85 son:

- Mayor frecuencia de reloj, lo que permite realizar tareas más complicadas sin entorpecer el protocolo USB.
- Mayor capacidad de pines de E/S, que nos permiten manejar un mayor número de periféricos simultáneamente.

La figura \ref{fig:44} muestra la posición de los pines en el chip.

![Pinout ATmega328p [@atmegapinout-image]](img/atmega_pinout.png){width=55% #fig:44}

Para el prototipo final del proyecto, hacemos uso de la placa NANO que incorpora el microchip ATmega328p. Además de los 14 pines de E/S mencionados anteriormente, contamos con 8 pines de entrada analógica y 6 pines PWM. Cuenta también con un puerto USB utilizado en este proyecto como salida de puerto serie (para depuración del firmware), y un botón de reset para reiniciar la placa [@nano-info]. El uso de esta placa es una opción muy interesante para la creación de prototipos, ya que se puede integrar fácilmente en circuitos breadboard o perfboard [@nano-pinout]. La figura \ref{fig:60} muestra la disposición de los pines en la placa NANO.

![Pinout de la placa NANO [@nanopinout-image]](img/nanopinout.png){width=70% #fig:60}



## Periféricos soportados por el proyecto

En este apartado se describen los distintos elementos hardware a los que se ha dado soporte en el proyecto. Se ha partido de una placa inicial, la placa Bee 2.0, utilizada en la asignatura LIN que cuenta con distintos periféricos, entre ellos un display 7 segmentos o un buzzer. También se ha hecho uso de un anillo led circular y de una pantalla OLED, externos a esta placa.




### Placa Bee 2.0

Como punto de partida en el proyecto, se han estudiado los componentes y el funcionamiento de los distintos elementos hardware de la placa *Bee 2.0*, diseñada para la asignatura *Arquitectura Interna de Linux y Android*. La placa puede observarse en la figura \ref{fig:45}.

![Placa Bee 2.0 de la asignatura Arquitectura interna de Linux y Android [@bee-board]](img/bee20gen.png){width=70% #fig:45}

La idea de este proyecto es impulsar todo el potencial que ofrece esta placa, al utilizar un firmware de base que permite implementar cualquier funcionalidad utilizando todos los periféricos que la componen. Sus componentes son: un diodo LED RGB con sus correspondientes resistencias, tres pulsadores, tres diodos de color rojo (también interconectados mediante resistencias, para evitar posibles daños), un display 7 segmentos, un zumbador *buzzer* pasivo, conversores ADC (analógico-digital) y DAC (digital-analógico), y salida UART (puerto serie). Dispone de varios pines para poder conectar periféricos externos, así como dos salidas de voltaje a 3.3V y 5V. Originalmente, esta placa se utiliza junto a una Raspberry PI en la asignatura de LIN, utilizando para el desarrollo de sus prácticas los siguientes periféricos: Display 7 segmentos, zumbador *buzzer*, los tres diodos y un pulsador. El utilizar conjuntamente esta placa con nuestro firmware posibilita la elaboración de prácticas con cualquier periférico de los mencionados anteriormente, y no sólo limitando su funcionalidad a estos últimos.




### Anillo de LEDs de colores

Este periférico (externo a la placa Bee 2.0) cuenta con 12 LEDs conectados en serie dispuestos de forma circular, con comunicación unidireccional. La empresa encargada de su diseño es NeoPixel [@LED-strip]. Dichos LEDs están compuestos microcontrolador y un registro asociado a cada uno, con colores RGB direccionables individualmente. Cada controlador está compuesto por un registro especial, que gestiona la configuración de color e intensidad de cada LED. El dispositivo, probado originalmente con ATTiny85, se puede observar en la figura \ref{fig:46}:

![Tira de LEDs circular de NeoPixel](img/LED-Strip.jpg){width=75% #fig:46}

Para transmitir información de color e intensidad a cada LED, se utiliza una sola línea de datos en todo el dispositivo. Esta línea de datos usa un protocolo de comunicación específico llamado *One-Wire Protocol*. Mediante este protocolo, se puede implementar funcionalidad para poder iluminar todos los LEDs del anillo a la vez, o crear una serie de patrones para que se vayan encendiendo en distinto orden y con distintos colores. 

La tira circular de LEDs se alimenta con una fuente de alimentación de 5V que proporciona la propia placa a la que está conectada. El dispositivo está gestionado por el microcontrolador de la placa a la que está conectado.



#### Smart Shift Register en los LEDs de NeoPixel

Cada LED RGB individual contiene un un pequeño microcontrolador y un registro. Dicho microcontrolador es el responsable de controlar el color y el brillo, y de comunicarse con los otros LEDs de la cadena. Todos los chips que componen cada LED de este dispositivo utilizan el protocolo *One-Wire* para su comunicación, y tienen un componente clave llamado *Smart Shift Register*, que se explica a continuación. 

![Arquitectura de cada LED [@smsreg-video]](img/pines_neopixel.jpg){width=50% #fig:47}

Cada microcontrolador se comporta como registro de desplazamiento, recibiendo los bits por el pin DIN y sacando los bits de acarreo por el pin DOUT. La arquitectura de cada LED se puede observar en la figura \ref{fig:47}. De este modo, al recibir datos por uno de los pines, los bits que salen son los transmitidos al siguiente controlador de la cadena. Cada registro *Smart Shift Register* está compuesto por los siguientes elementos:

- **Bit de control de datos**: Este bit está dedicado a controlar la transferencia de datos. Mediante este bit, el controlador puede gestionar la información sobre configuración de colores e intensidad recibida.
- **Registros de color**: Dentro de cada *Smart Shift Register*, existen subregistros dedicados a cada color (R-G-B). Su función es almacenar los valores de brillo y determinar el color resultante del LED.
- **Protocolo de comunicación**: El registro está orquestado por un protocolo llamado *1-Wire* (*One-Wire*), que rige una secuencia de bits específica para determinar los datos.
- **Sincronización**: Su implementación permite operar con la señal de reloj del microcontrolador principal, para evitar fallos y desfases en la comunicación.
- **Multiplexación/Almacenamiento**: Este registro también se encarga de transmitir los bits de cada LED secuencialmente en la cadena a la siguiente componente, almacenando la información y transmitiéndola en el instante adecuado. 

Cuando el *smart shift register* recibe la información en forma de bits, se encarga de ajustar los registros internos para poder establecer los colores del LED en cuestión. Este proceso se repite para los 12 LEDs que componen el anillo circular, como se puede observar en la figura \ref{fig:48}.

![Flujo de bits dentro del controlador de cada LED [@smsreg-video]](img/onewire.png){width=70% #fig:48}

Cada registro en la cadena de LEDs cuenta con una posible configuración de 24 bits, agrupándose en grupos de 8 bits recogidos en 3 *data latches* o registros, que el controlador interpreta para ajustar el color o brillo de cada LED (se utiliza un grupo de 8 bits para controlar el color rojo, otro grupo de 8 bits para el color verde y otro grupo de 8 bits para el color azul). Cuando se han recibido todos los datos, el microcontrolador de cada LED se encarga de excitarlo para iluminarlo. Cuando el registro de desplazamiento recibe la señal de *reset*, quiere decir que se pueden iluminar los LEDs al haber terminado el envío a toda la cadena. A partir de ese momento, se empieza otra secuencia de datos. [@smsreg-video]



#### Protocolo One-Wire

El protocolo *One-Wire* (o también conocido como *1-Wire*) permite la transmisión de 1's o 0's en cadena, pudiendo transmitir la información a cada LED contando con una única línea de datos.

El funcionamiento de este protocolo es el siguiente: para poder determinar el dato, se mide los tiempos de transmisión entre el flanco de subida y bajada. Si el tiempo entre el flanco de subida y el flanco de bajada es mayor que el tiempo transcurrido hasta el siguiente flanco de subida, el resultado es un voltaje alto (TH) o un 1 lógico. Por el contrario, si el tiempo entre el primer flanco de subida y el flanco de bajada es menor que el tiempo transcurrido hasta el el siguiente flanco de subida, el resultado es un voltaje bajo (LW) o un 0 lógico. En la figura \ref{fig:49} se ilustra este funcionamiento.

![Cálculo de cada dato según los flancos de subida o bajada en 1-Wire [@smsreg-video]](img/1wireTHTL.png){width=50% #fig:49}



### Display 7 segmentos

Este periférico se encuentra en la placa Bee 2.0, es capaz de representar un número del 0 al 9 mediante segmentos verticales y horizontales que, según el número a mostrar, se encienden o apagan mostrando el número en cuestión. También cuenta con un punto en la parte inferior derecha. La configuración hardware de este display es de cátodo común, es decir, cuando se escribe un 1 se enciende el segmento deseado, y cuando se escribe un 0, se apaga.

![Diagrama del circuito del display 7 segmentos de la placa Bee 2.0 [@sevenseg]](img/d7seglin.png){width=50% #fig:d7s}

La figura \ref{fig:d7s} ilustra el circuito interno del display 7 segmentos. Como se puede observar, cuenta con dos registros, controlados por 3 pines de la placa Bee 2.0. Esto es una enorme ventaja, ya que si tuviéramos 8 pines de entrada para el display, limitaría la posible interacción con otros dispositivos de E/S, al tener que estar constantemente enviando un 1 lógico en aquellos pines correspondientes a los segmentos que se quieren encender. Para ello, el sistema emplea dos registros, uno de desplazamiento y otro de salida, que se explican a continuación.

Entradas del registro de desplazamiento: 

- **SDI** (Serial Data Input): es la entrada serie del registro, de 8 bits. En cada ciclo de reloj se propaga un bit que controla el estado de un segmento distinto. El bit menos significativo corresponde al punto, el más significativo al segmento horizontal superior.
- **SRCLK** (Shift Register Clock): señal del reloj correspondiente al registro de desplazamiento. 
- **Enable**. Esta señal debe estar a 1 para que el sistema realice la acción de desplazamiento y carga paralela, mediante los dos registros.

Entradas del registro de salida:

- **RCLK** (Register Clock): señal del reloj correspondiente al registro de salida.
- **Load**. Al igual que la señal de enable, tiene que estar a 1 para el sistema realice las acciones de desplazamiento y carga paralela.

Para poder generar la salida deseada, con establecer un 1 en la señal de RCLK (que debe estar a 0 durante todo el proceso previo), se cargaría inmediatamente el número en el registro, y por lo tanto, en la salida de los segmentos del display. La alteración de las señales SDI, SRCLK y RCLK se realiza mediante *bit banging*, es decir, es la propia CPU la que establece el valor del pin de forma temporizada permitiendo que la señal se mantenga un cierto tiempo en el valor lógico deseado.  [@bee-board]



### Pantalla LCD OLED

Este periférico es externo a la paca Bee 2.0, consta de una pantalla LCD tipo OLED (formado por un cristal líquido), en el que se permite mostrar cadenas de texto. Para poder ser gestionada mediante el microcontrolador principal, se utiliza el protocolo I2C [@attiny-oled]. El dispositivo en cuestión es el mostrado en la figura \ref{fig:50}:

![Pantalla OLED utilizada en el proyecto](img/LCD_OLED.png){width=40% #fig:50}

El protocolo I2C permite la comunicación serie mediante dos cables para transferencias entre el microcontrolador (o maestro) y el periférico (o esclavo). Para poder establecer la comunicación, es necesario tener en cuenta los siguientes puntos en la conexión y comunicación utilizando I2C:

- **Conexión**: En nuestro caso, para permitir transferencias de información a la pantalla, se deben conectar las salidas I2C del dispositivo a la placa NANO, mediante las líneas SDA (Serial Data Line) y SCL (Serial Clock Lline). En nuestro caso, ATmega328p será el maestro, y la pantalla OLED será el esclavo. 
- **Reloj (SCL)**: Esta línea se utiliza para sincronizar las transferencias entre el microcontrolador y el periférico, generando las correspondientes señales de reloj. Estas señales son gestionadas por el maestro, ya que es quién determina la velocidad de transferencia de datos.
- **Línea de datos (SDA)**: Los datos se transmiten de forma bidireccional entre maestro y esclavo (cada componente puede enviar y recibir). Las transmisiones se dividen en secuencias, formadas por paquetes de datos. 
- **Dirección I2C del dispositivo**: La pantalla OLED debe tener asignada una dirección única, para poder ser reconocido por el microcontrolador y comunicarse con él. 
- **Comandos/datos**: Para enviar información a la pantalla, se realiza a través de la línea SDA mediante comandos y datos. Estos comandos configuran, entre otras cosas, el brillo y contraste. Los datos sirven para transmitir información que la pantalla mostrará.
- **Flujo de datos en I2C**: Para iniciar la comunicación, el maestro envía una señal de inicio o *start* junto a la dirección asignada a la pantalla. Después, se envían los comandos correspondientes y los datos al periférico. El esclavo, una vez recibida la información, responde con un ACK para confirmar la recepción. Una vez finalizada la comunicación, el maestro envía una señal de *stop*, indicando el fin de la transmisión.



### Zumbador pasivo *Buzzer*

Este periférico se encuentra en la placa Bee 2.0, utiliza una señal PWM para su funcionamiento. PWM es una técnica que requiere de la manipulación del ancho de una señal periódica, para gestionar la energía que se envía al dispositivo. Para poder hacer sonar el buzzer, se requiere enviar pulsos de señal a una frecuencia constante. El hecho de modificar la amplitud de las ondas crea diferentes niveles de volumen. 

En el microcontrolador, existen registros específicos para general la señal PWM. Se debe configurar un timer adecuado, estableciendo la frecuencia de conteo y el modo de funcionamiento. Aplicado al ATmega328p, existe un módulo en el chip que se encarga de generar estas señales PWM. El encargado de generar la señal periódica es el propio timer, para ello se debe configurar una frecuencia adecuada, ya que sino el volumen podría ser muy bajo o muy alto. Para hacer sonar el buzzer, en cada ciclo se comienza con un 1 en la salida PWM, y se conmuta a 0 cuando el valor del contador del timer baja de un determinado umbral. Configurando este umbral, se establece el *duty cycle*, seleccionando así el volumen/tono emitido por el zumbador. 

Para conectar el buzzer de la placa Bee 2.0 a la placa NANO, se debe unir la salida positiva de la placa Bee 2.0 (PWM1) a uno de los pines PWM de la placa NANO, y las dos salidas de tierra (GND).