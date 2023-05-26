<!-- Leave a blank line before the title -->

# Proyecto V-USB

V-USB es un framework que permite crear dispositivos USB con microcontroladores que no disponen de soporte hardware para este fin. Usa una implementación 100% software para emular la comunicación permitiendo así crear dispositivos con esta funcionalidad a un coste inferior y con menor complejidad que los chips que tienen esta característica implementada en hardware.

En este capítulo, se describen sus detalles y su arquitectura interna, así como algunas alternativas que se podrían haber usado para la creación de la infraestructura diseñada.


## Framework V-USB

El proyecto V-USB [@objective-development] distribuye una implementación de uso libre que permite la creación de dispositivos USB *low-speed* a través de software, capaz de funcionar en los microcontroladores *AVR* de la empresa *Atmel*. De esta manera, nos brinda la posibilidad de construir dispositivos USB por muy poco coste ya que hace uso únicamente del hardware del microcontrolador sin ningún chip adicional, por lo que es una alternativa económica y sencilla frente a otras opciones que hay para construir este tipo de dispositivos.

La manera en la que V-USB implementa esta funcionalidad es a través de la técnica *Bit Banging* [@bit-banging]. *Bit Banging* es la manera de implementar en software la lógica de un protocolo usando pines genéricos de *E/S*, en lugar de usar un controlador hardware específicamente diseñado para ello. Estas señales son generadas activando o desactivando las líneas de datos y ajustando los tiempos de espera a los específicos del protocolo. Aunque es una técnica ampliamente usada, también presenta algunos incovenientes como la fiabilidad en la transmisión o la cantidad de procesamiento que conlleva lo que deriva en una pérdida de rendimiento. Aun así, es una buena alternativa en escenarios como en el que nos encontramos donde buscamos un microcontrolador de coste bajo y poco hardware complementario.

Las limitaciones que pone en los chips que soporta son extremandamente bajas, por lo que prácticamente cualquier microcontrolador *Atmel* será compatible. Algunas de las características que deben de tener los chips son:

- 2kB de memoria flash
- 128B de memoria RAM
- Reloj de al menos 12MHz, ya que este parámetro es crítico para su funcionamiento. Soportando también las frecuencias: 12.8 MHz, 15MHz, 16MHz, 16.5MHz y 20MHz.

Además, no requiere de ningún puerto UART, timer o hardware especial para funcionar. Esto supone una gran ventaja ya que en caso de que el microcontrolador elegido tenga estas características, quedan completamente disponibles para su uso.

La libería está escrita en C junto con algunas partes en ensamblador y consta de diferentes ficheros donde se emula el comportamiento de un controlador USB hardware, así como diferentes ficheros de cabecera *.h* que permiten configurar algunos de los parámetros disponibles en este tipo de dispositivos, y activar o desactivar las funcionalidades que el framework implementa.

En este proyecto, hemos hecho uso de la librería tanto en el microcontrolador *ATTiny85* como en el *ATMega328P* con completo éxito, siendo este último mucho mejor opción debido a la frecuencia de reloj y el número de puertos E/S, así como del hardware adicional que trae, como timers o generadores de señales.

## Arquitectura Interna

La arquitectura interna de V-USB [@vusb-files] se basa en una serie de ficheros que trabajan en conjunto para implementar la funcionalidad que ofrece el framework. Posee un diseño bastante modular que permite adaptarse a múltiples microcontroladores y herramientas de compilación con apenas unos cambios en ficheros de cabecera `.h`.

En la parte de más bajo nivel de la librería podemos encontrar cuatro ficheros escritos en ensamblador que implementan rutinas para tareas específicas como la comprobación de errores en la comunicación o ajustes de temporización en base a la velocidad de reloj elegida. Estos ficheros son:

- `usbdrvasm.S`: Este es el principal fichero con código ensamblador. Contiene las instrucciones más generales para preprocesar las sentencias de código y la comprobación de errores mediante el mecanismo *CRC*. También se encarga de incluir la parte ensamblador dependiente de la velocidad de reloj elegida.
- `usbdrvasmX.inc`: Con el framework se incluyen diferentes ficheros de este tipo, donde la X del nombre representa la velocidad del reloj, y esencialmente ajustan las instrucciones al tiempo del reloj escogido. El fichero adecuado a la velocidad escogida es incluido por el fichero anterior, *usbdrvasm.S*.
- `asmcommon.inc`: Aquí podemos encontrar mayormente instrucciones compartidas por el resto de ficheros, ya que el código que contiene se extrae a este fichero porque su funcionamiento es independiente de la velocidad de reloj elegda. Su función principal es procesar los paquetes USB cuando son recibidos.
- `usbdrvasm.asm`: Este fichero es mayormente un fichero de compatibilidad. Se necesita cuando el entorno de compilación utiliza *IAR-C*, otro compilador ampliamente conocido que sirve de alterntiva a *GCC*.

Un poco más cercano al usuario y escrito en *C* podemos encontrar la *API* que nos permite incorporar V-USB a nuestro proyecto junto con algunas funciones de más alto nivel. En este nivel también nos encontramos con los ficheros de cabecera que permiten personalizar y ajustar el proyecto a nuestras necesidades hardware específicas. Los ficheros que conforman esta capa son:

- `usbdrv.h`: En este fichero podemos encontrar la interfaz de la *API* que nos permite integrar la comunicación USB en nuestro proyecto. Aquí se encuentran definidas todas las funciones que posteriormente implementaremos, algunas de ellas de manera obligatoria y otras que podemos habilitar a través de macros para activar ciertas funcionalidades que nos ofrece V-USB. Ya que la implementación de estas funciones es completamente personal y dependiente del usuario de la libería, en el capítulo donde se describe el firmware [REF] se pueden encontrar detalladas algunas de las funciones más importantes.
- `usbdrv.c`: Contiene la implementación de las funciones más generales y de más alto nivel del driver. Este fichero es el que se tiene que enlazar desde nuestro código.
- `usbconfig.h`: Otro fichero de configuración importante. Aquí podemos encontrar definidos los parámetros que V-USB nos permite configurar, entre los que podemos encontrar ajustes propios del protocolo USB y ajustes más cercanos a la parte del microcontrolador usado. Como su configuración también es muy dependiente del usuario, en el capítulo *Firmware* [REF] se puede encontrar una explicación detallada de las partes más importantes.
- `usbportability.h`: Aquí podemos encontrar mayormente sentencias que permiten ampliar la compatibilidad entre compiladores. Gracias a este fichero se puede elegir entre los entornos *IAR-C* y *GCC*.

Por último, encontramos la utlidad de debug que nos proporciona el framework. Consiste en un módulo que usando comunicación por puerto serie habilita una interfaz de debug a la que podemos conectar un PC y ver la salida que enviemos por dicho canal. Consta de dos ficheros, `oddebug.c` y `oddebug.h` cuya explicación en detalle se puede encontrar en la sección [REF].

## Dificultades encontradas

La elección de este framework para implementar el diposisito USB ha sido un gran acierto dadas todas las ventajas que ofrece,  pero a pesar de ello, también hemos encontrado algunas dificultades durante su uso.

La primera de ellas es la documentación. La mayor parte de información la podemos encontrar en los ficheros `.h` del proyecto en forma de comentarios y aunque son de gran valor, ya que describen con detalle las diferentes funciones, estructuras y macros, se necesita cierto conocimiento técnico sobre el protocolo USB y sobre microcontroladores para entender con claridad lo que describen. Hemos echado en falta una wiki más completa con mayor nivel de detalle que nos ayude a comprender algunos de los conceptos específicos del protocolo USB.

Otro de los inconvenientes que hemos encontrado es que hay múltiples proyectos que implementan diferentes periféricos, de maneras dispares y algunos con prácticas no del todo correctas como la desactivación del Watchdog, mecanismo de control esencial para evitar que se cuelgue el dispositivo. Dado que no hemos encontrado proyectos que soporten diferentes periféricos al mismo tiempo en el firmware, la implementación de este soporte multidispositivo ha sido algo complicada de llevar a cabo.


## Alternativas a V-USB

V-USB es una excelente opción para los escenarios que se tratan en los apartados anteriores, esencialmente nos permite emular un dispositivo USB en microcontroladores con pocos recursos utilizando simplemente código C.

Pero es posible que para otros proyectos necesitemos, por ejemplo, una mayor capacidad de procesamiento paralelo a la gestión de la comunicación USB, o mayor velocidad en la propia comunicación, ya que V-USB solo implementa dispositivos *USB 1.0* o lo que es lo mismo *Low Speed USB*.

Como alternativa para resolver los escenarios de ejemplo propuestos podríamos usar diferentes microcontroladores que incorporen una controladora hardware de USB. Gracias a esto, liberamos mucho a la CPU de la gestión de la comunicación, ya que todo este proceso se trata por un hardware a parte dedicado exclusivamente a ello.

Dentro de esta gama de microcontroladores y sin salirnos de la familia *ATMega*, encontramos el *ATMega32u4*. Este microcontrolador ofrece, entre otras funcionalidades, la comunicación USB por hardware con las siguientes características:

- Trabaja bajo el estandar USB 2.0 Full-speed/Low Speed y permite usar interrupciones
- Soporta transferencias de datos de hasta 12Mb/s y 1.5Mb/s
- 6 Endpoints, programables a través de registros, tanto de tipo IN como OUT y funcionando en todos los modos
- 832 bytes de memoria USB DPRAM completamente independiente para uso de los endpoints
- Mejor eficiencia cuando se trabaja en modo Low Speed

Saliendo de la marca *ATmel*, otra de las grandes alternativas debida a su potencia, y muy usada hoy en día, es el *STM32* de la marca ST [@ST-Electronic]. Este microcontrolador también nos ofrece la implementación USB por hardware con las siguientes características:

- Trabaja con los estándares USB 2.0/1.0
- Permite velocidades de transmisión de hasta 480 Mb/s y 12 Mb/s
- Tiene 6 endpoints completamente configurables
- Tiene un cristal dedicado que trabaja a una frecuencia de 48MHz
- Posee un DMA interno para USB

A pesar de que los microcontroladores con hardware USB también son una alternativa muy extendida en el campo de desarrollo de dispositivos USB, podemos enumerar ciertas ventajas en usar una implementación por software que nos hace decantarnos por esta opción:

- Los chips AVR son normalmente más fáciles de obtener que otros con soporte hardware para USB
- La mayoría de los chips que cuentan con este hardware vienen en empaquetados *SMD*, es decir, son de un tamaño muy pequeño y no permiten el prototipado de manera sencilla debido a la dificultad de conectarnos sin ser soldados en una placa.
- El entorno de desarrollo es extremadamente simple, basta con tener un compilador de tipo GCC y un editor de texto plano.
- Son significantemente más baratos. Por poner un ejemplo, en el distribuidor español *Mouser* [@mouser] podemos adquirir un *ATTiny85* por apenas 1,65€, mientras que el precio de un *ATMega32u4* alcanza los 5,27€.
