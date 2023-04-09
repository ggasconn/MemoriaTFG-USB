<!-- Leave a blank line before the title -->

# Introducción

En los siguientes apartados se explican los motivos de la realización del proyecto, los objetivos y la planificación de las tareas para llevarlo a cabo.



## Motivación

El protocolo USB es ampliamente usado en todo el mundo para la comunicación con casi cualquier periférico. De forma sencilla, cualquier usuario con un ordenador puede conectar un dispositivo, da igual la funcionalidad que tenga, al puerto USB de su ordenador y éste reconocerlo para que pueda comunicarse con la CPU y realizar la función para la que ha sido desarrollado.

Para que todo esto se pueda dar, el protocolo USB tiene una complejidad enorme en cuanto a paquetes de comunicación entre la CPU y el dispositivo o la controladora USB, a parte de necesitar un driver instalado en el sistema operativo para que pueda interpretarlo. A pesar de lo sencillo que pueda parecer el hardware (2 cables de datos), hay muchos elementos en la comunicación USB que tienen lugar para que el dispositivo pueda ser reconocido.

Para poder estudiar todo el protocolo USB y la interacción entre el dispositivo y la CPU, en este proyecto hemos desarrollado un firmware en C que realiza una serie de funciones sobre una pantalla LCD y un sensor de temperatura, utilizando un microchip ATTiny85 sobre una placa Digispark, que contiene el propio controlador USB.



## Objetivos

El objetivo principal del proyecto es el uso del firmware como banco masivo de pruebas para otros proyectos que puedan utilizar la librería V-USB. El diseño de un dispositivo con distintos periféricos (son los que se explican en los siguientes capítulos) es un ejemplo de utilidad de dicha librería. 

Otro objetivo fundamental del proyecto es el poder familiarizarse con el protocolo USB, los distintos paquetes utilizados para la comunicación y cómo estos interactúan con el microcontralor. 



## Plan de trabajo

Para el desarrollo de este proyecto, se han mantenido distintas reuniones con los directores del proyecto y entre los desarrolladores. En la parte inicial del proyecto, se ha estudiado el funcionamiento de la librería V-USB con distintos proyectos que utilizan otros microchips, para ver qué uso se hacen de los report-IDs y las distintas funciones empleadas en el código. Se ha utilizado el software Wireshark [@sw-wireshark] para estudiar los endpoints usados y los paquetes URBs en la comunicación, y así estudiar la configuración del software y la posibilidad de utilizar, por ejemplo, interrupciones.

Para el desarrollo de los periféricos que se van a usar conjuntamente en la placa Digispark, Guillermo se ha encargado de estudiar y desarrollar el código correspondiente al sensor de temperatura y Javier al correspondiente con la pantalla LCD 2x16, que posteriormente se ha utilizado una pantalla LCD OLED.

Para la integración del código, se ha utilizado GitHub.




## Organización de la memoria

Para la realización de esta memoria, se ha dividido la misma en varios capítulos. En el primero, se explica a modo de introducción el hardware utilizado y algunos aspectos de bajo nivel sobre la librería utilizada. En el siguiente capítulo, se explican los elementos hardware utilizados, como el anillo circular de leds o la pantalla LCD OLED. Después, se explica con detalle el microcontrolador utilizado finalmente para el prototipo final, ya que éste no tiene las limitaciones que contábamos con el ATTiny85, al poder tener más líneas de comunicación con él.

En la parte de apéndices, contamos con las aportaciones de cada integrante del proyecto, así como distintas instrucciones para poder analizar los paquetes URBs de USB con el software Wireshark, o también para poder *flashear* nuestro propio firmware, con indicaciones sobre los pines concretos a utilizar en la placa.