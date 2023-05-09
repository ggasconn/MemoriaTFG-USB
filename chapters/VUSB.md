<!-- Leave a blank line before the title -->

# Proyecto V-USB

V-USB es la librería que se ha utilizado en el proyecto para emular un dispositivo USB usando un microcontrolador sin esta capacidad. 

En este capítulo, se describen los detalles de la librería, así como algunas alternativas que se podrían haber usado para la implementación del dispositivo


## Descripción

El proyecto V-USB [@objective-development] distribuye una implementación de uso libre que permite la creación de dispositivos USB low-speed vía software, capaz de funcionar en los microcontroladores *AVR* de la empresa *Atmel*.

De esta manera, nos brinda la posibilidad de construir dispositivos USB por muy poco coste ya que hace uso únicamente del hardware del microcontrolador sin ningún chip adicional, por lo que es una alternativa económica y sencilla frente a las otras opciones que hay para construir este tipo de dispositivos.

Las limitaciones que pone en los chips que soporta son extremandamente bajas, por lo que prácticamente cualquier microcontrolador Atmel será compatible. Algunas de las características que deben de tener los chips son:

- 2kB de memoria flash
- 128B de memoria RAM
- Reloj de al menos 12MHz. Soportando frecuencias de hasta 20Mhz

Además, no requiere de ningún puerto UART, timer o hardware especial para funcionar. Siendo esto una ventaja ya que en caso de que el microcontrolador elegido tenga estas características, quedan completamente disponibles para su uso.

La libería está escrita en C y consta de diferentes ficheros donde se emula el comportamiento de un controlador USB hardware, así como diferentes ficheros de configuración que permiten exprimir al máximo el microcontrolador en uso gracias a todos los parámetros que se pueden ajustar.

Durante el proyecto, hemos hecho uso de la librería tanto en el microcontrolador *ATTiny85* como en el *ATMega328P* con completo éxito, siendo este último mucho mejor opción debido a la frecuencia de reloj y el número de puertos I/O, así como del hardware adicional que trae como timers o generadores de señales.


## Alternativas a esta librería

V-USB es una excelente opción para los escenarios que se tratan en los apartados anteriores, esencialmente nos permite emular un dispositivo USB en microcontroladores con pocos recursos utilizando simplemente código C.

Pero es posible que para otros proyectos necesitemos, por ejemplo, una mayor capacidad de procesamiento paralelo a la gestión de la comunicación USB, o mayor velocidad en la propia comunicación, ya que V-USB solo implementa dispositivos *USB 1.0* o lo que es lo mismo *Low Speed USB*.

Como alternativa para resolver los escenarios propuestos de ejemplo podríamos usar diferentes microcontroladores que ya ofrecen una gestión USB implementada por hardware. Gracias a esto, liberamos mucho a la CPU de calcular toda la comunicación, ya que todo este proceso se trata por un hardware a parte dedicado exclusivamente a ello.

Dentro de esta gama de microcontroladores y sin salirnos de la familia *ATMega*, encontramos el *ATMega32u4*. Este microcontrolador ofrece, entre otras funcionalidades, la comunicación USB por hardware con las siguientes características:

- Trabaja bajo el estandar USB 2.0 Full-speed/Low Speed y permite usar interrupciones
- Soporta transferencias de datos de hasta 12Mb/s y 1.5Mb/s
- 6 Endpoints programables tanto de tipo IN como OUT y funcionando en todos los modos
- 832 bytes de memoria USB DPRAM completamente independiente para uso de los endpoints
- Mejor eficiencia de ciclos cuando se trabaja en modo Low Speed

Saliendo de la marca *ATmel*, otra de las grandes alternativas debida a su potencia, y muy usada hoy en día, es el *STM32* de la marca ST [@ST-Electronic]. Este microcontrolador también nos ofrece la implementación USB por hardware con las siguientes características:

- Trabaja con los estándares USB 2.0/1.0
- Permite velocidades de transmisión de hasta 480 Mb/s y 12 Mb/s
- Tiene 6 endpoints completamente configurables
- Tiene un cristal dedicado que trabaja a una frecuencia de 48MHz
- Posee un DMA interno para USB
