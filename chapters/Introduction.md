<!-- Leave a blank line before the title -->

# Introducción

En los siguientes apartados se explican los motivos de la realización del proyecto, los objetivos y la planificación de las tareas para llevarlo a cabo.




## Motivación

El protocolo USB es ampliamente usado en todo el mundo para la comunicación con casi cualquier periférico. De forma sencilla, cualquier usuario con un ordenador puede conectar un dispositivo, da igual la funcionalidad que tenga, al puerto USB de su ordenador y éste reconocerlo para que pueda comunicarse con la CPU y realizar la función para la que ha sido desarrollado.

Para que todo esto se pueda dar, el protocolo USB tiene una complejidad enorme en cuanto a paquetes de comunicación entre la CPU y el dispositivo o la controladora USB, a parte de necesitar un driver instalado en el sistema operativo para que pueda interpretarlo. A pesar de lo sencillo que pueda parecer el hardware (2 cables de datos), hay muchos elementos en la comunicación USB que tienen lugar para que el dispositivo pueda ser reconocido.

Para poder estudiar todo el protocolo USB y la interacción entre el dispositivo y la CPU, en este proyecto hemos desarrollado un firmware en C que se comunica con diferentes sensores y dispositivos, utilizando un microcontrolador AVR. Así como un conjunto de drivers de ejemplo para mostrar el funcionamiento y la comunicación entre el ordenador y el dispositivo.




## Objetivos

El objetivo principal de este proyecto es construir un dispositivo USB de bajo coste, que pueda ser utilizado para familiarizarse con el complejo protocolo que maneja este estandar de conexión, entre otros aspectos podemos destacar los paquetes de información utilizados, *URBs*, y los pasos en la comunicación entre el dispositivo USB y el host.  Así como con el desarrollo de drivers para dispositivos USB, utilizando la API síncrona y asícrona del kernel Linux.

Además, también pretende familizarse con los distintos dispositivos y sensores, como pantallas o lectores de temperatura, que se usan conectados al dispositivo USB que en este proyecto se desarrolla.

! EXPLICAR LAS DISTINTAS ALTERNATIVAS CONTEMPLADAS, EN LUGAR DE USAR V-USB



## Plan de trabajo

Para el desarrollo de este proyecto, se han mantenido distintas reuniones con los directores del proyecto y entre los desarrolladores. En la parte inicial del proyecto, se ha estudiado el funcionamiento de la librería V-USB con distintos proyectos que utilizan otros microchips, para ver qué uso se hacen de los report-IDs y las distintas funciones empleadas en el código. Se ha utilizado el software Wireshark [@sw-wireshark] para estudiar los endpoints usados y los paquetes URBs en la comunicación, y así estudiar la configuración del software y la posibilidad de utilizar, por ejemplo, interrupciones.

Para el desarrollo de los periféricos que se van a usar conjuntamente en la placa, Guillermo se ha encargado de estudiar y desarrollar el código correspondiente al sensor de temperatura y Javier al correspondiente con la pantalla LCD 2x16, que posteriormente se ha utilizado una pantalla LCD OLED.

Para la integración del código, se ha utilizado GitHub.




## Organización de la memoria

Para la realización de esta memoria, se ha dividido la misma en varios capítulos. En el primero, se explica a modo de introducción el hardware utilizado y algunos aspectos de bajo nivel sobre la librería utilizada. En el siguiente capítulo, se explican los elementos hardware utilizados, como el anillo circular de leds o la pantalla LCD OLED. Después, se explica con detalle el microcontrolador utilizado finalmente para el prototipo final, ya que éste no tiene las limitaciones que contábamos con el ATTiny85, al poder tener más líneas de comunicación con él.

En la parte de apéndices, contamos con las aportaciones de cada integrante del proyecto, así como distintas instrucciones para poder analizar los paquetes URBs de USB con el software Wireshark, o también para poder *flashear* nuestro propio firmware, con indicaciones sobre los pines concretos a utilizar en la placa.
