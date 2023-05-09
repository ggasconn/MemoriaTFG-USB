<!-- Leave a blank line before the title -->

# Introducción

En los siguientes apartados se explican los motivos de la realización del proyecto, los objetivos y la planificación de las tareas para llevarlo a cabo.




## Motivación

El protocolo USB es ampliamente usado en todo el mundo para la comunicación con casi cualquier periférico. De forma sencilla, cualquier usuario con un ordenador puede conectar un dispositivo, da igual la funcionalidad que tenga, al puerto USB de su ordenador y éste reconocerlo para que pueda comunicarse con la CPU y realizar la función para la que ha sido desarrollado.

Para que todo esto se pueda dar, el protocolo USB tiene una complejidad enorme en cuanto a paquetes de comunicación entre la CPU y el dispositivo o la controladora USB, a parte de necesitar un driver instalado en el sistema operativo para que pueda interpretarlo. A pesar de lo sencillo que pueda parecer el hardware (2 cables de datos), hay muchos elementos en la comunicación USB que tienen lugar para que el dispositivo pueda ser reconocido.

Para poder estudiar todo el protocolo USB y la interacción entre el dispositivo y la CPU, en este proyecto hemos desarrollado un firmware en C que se comunica con diferentes sensores y dispositivos, utilizando un microcontrolador AVR. Así como un conjunto de drivers de ejemplo para mostrar el funcionamiento y la comunicación entre el ordenador y el dispositivo.




## Objetivos

El objetivo principal de este proyecto es construir un dispositivo USB de bajo coste, que pueda ser utilizado para familiarizarse con el complejo protocolo que maneja este estandar de conexión, entre otros aspectos en los que podemos destacar los paquetes de información utilizados, *URBs*, y los pasos en la comunicación entre el dispositivo USB y el host.  Así como con el desarrollo de drivers para dispositivos USB, utilizando la API síncrona y asícrona del kernel Linux.

Además, también pretende familizarse con los distintos dispositivos y sensores, como pantallas o lectores de datos, que se usan conectados al dispositivo USB que en este proyecto se desarrolla.

Para la realización del proyecto, se han contemplado distintas alternativas, en lugar de usar la librería V-USB. Entre ellas, la posibilidad de utilizar dispositivos hardware que implementen de fábrica la gestión del protocolo USB, pero las ventajas de esta librería son superiores, así como los conocimientos que podamos adquirir trabajando en C con los elementos de bajo nivel de este protocolo, que de otra forma no podríamos haber adquirido.



## Plan de trabajo

Para el desarrollo de este proyecto, se han mantenido distintas reuniones con los directores del proyecto y entre los desarrolladores. En la parte inicial del proyecto, se ha estudiado el funcionamiento de la librería V-USB con distintos proyectos que utilizan otros microchips, para ver qué uso se hacen de los distintos elementos del firmware y las distintas funciones empleadas en el código. Se ha utilizado el software Wireshark [@sw-wireshark] para estudiar la comunicación USB, es decir, los endpoints usados y los paquetes URBs en la comunicación. También se ha tenido en cuenta la configuración del software y la posibilidad de utilizar, por ejemplo, interrupciones.

Para el desarrollo de los periféricos que se van a usar conjuntamente en la placa, Guillermo se ha encargado de estudiar y desarrollar el código correspondiente a los periféricos de NeoPixel, y del buzzer y display 7segmentos; Javier al correspondiente con la pantalla LCD 2x16, que posteriormente se ha utilizado una pantalla LCD OLED. Se han ido manteniendo conversaciones constantes sobre los avances de implementación de todas las partes, así como reuniones en caso de bloqueos.

Para la integración del código, se ha utilizado GitHub.




## Organización de la memoria

Para la realización de esta memoria, se ha dividido la misma en varios capítulos, que se explican a continuación:

- En el primero, se hace una introducción del proyecto, con objetivos y plan de trabajo.

- En el segundo, se explica todo el funcionamiento a bajo nivel del protocolo USB, así como sus distintos componentes como los descriptores, Endpoints, o los paquetes URBs en los drivers.
- En el tercero, se explica a modo de introducción el proyecto V-USB de forma general, con algún ejemplo de código, así como la explicación de las posibles alternativas al uso de esta librería.
- En el cuarto, se explican en primer lugar los microcontroladores utilizados en el proyecto, así como la placa que finalmente se ha decidido presentar en el proyecto, el ATmega328p. Posteriormente, los elementos hardware con soporte en el firmware, como el anillo circular de leds o la pantalla LCD OLED. 
- En el quinto, se explica con detalle todos los elementos y funcionalidades implementadas en el firmware.
- En el sexto, se explican los drivers de ejemplo que hacen uso de los periféricos con soporte en el firmware.
- En el séptimo, se hace una reflexión sobre el posible uso de este proyecto para otros tipos de trabajos y pruebas, así como su integración en la asignatura de Arquitectura Interna de Linux y Android, impartida en esta facultad.

En la parte de apéndices, contamos con dos guías, en la primera se describen las instrucciones para poder analizar los paquetes URBs de USB con el software Wireshark, en la segunda se explica el proceso para poder *flashear* el firmware en la placa, con indicaciones sobre los pines concretos que hay que conectar.
