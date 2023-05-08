<!-- Leave a blank line before the title -->

# Proyecto V-USB

En este capítulo se describe el funcionamiento a bajo nivel de esta librería utilizada para el desarrollo del firmware, así como otras alternativas contempladas.


## Descripción

El proyecto V-USB [@objective-development] se utiliza para implementar firmware en dispositivos USB de baja velocidad con pocos recursos, entre ellos se encuentran los microcontroladores AVR (como los ATTiny o Atmel, utilizados en este proyecto para realizar las pruebas y desarrollar el firmware final). Al ser una librería que no requiere de mucho tamaño para almacenar su código, ni del funcionamiento de muchos recursos del chip, hace que sea compatible sobre dispositivos con prestaciones hardware limitadas y con arquitecturas más bien sencillas.

Esta librería ha sido probada en los chips ATTiny85 y posteriormente en ATmega328p, las ventajas han sido evidentes en el uso de un firmware basado en V-USB para estos dispositivos. El requisito más importante que se tiene en ambos chips, es el número de periféricos que se desean conectar a ellos, ya que dependiendo del alcance del proyecto, ésto puede llegar a ser una limitación. En ATmega, como se explica más adelante, esta limitación se reduce notablemente al disponer de pines suficientes para el alcance de este proyecto. 

El proyecto V-USB utiliza la interfaz USB para la comunicación entre el host (con un driver cargado que lo controle) y el dispositivo conectado, por lo que utiliza todos los elementos de la comunicación USB (endpoints, mensajes URBs, etc) que se detallarán más adelante. También hablaremos de la estructura interna del proyecto, ya que se hace uso de Report-IDs, característico de V-USB.

En cuanto a la comunicación USB, esta librería hace uso de dos endpoints en el chip ATTiny85: el endpoint 0 y endpoint 1. El endpoint 0 es usado para las transferencias de control, es decir, inicializar y configurar la interfaz USB. El endpoint 1, se usa para la transferencia de datos, es decir, para enviar y recibir los informes USB HID. [@vusb-library]

Para la configuración de los endpoints, están definidos en el archivo cabecera `usbconfig.h`. En el caso del ATTiny85, dicha configuración se puede encontrar en el directorio `usbdrv`.


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
