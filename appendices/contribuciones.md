<!-- Leave a blank line before the title -->

# Contribuciones

## Aportación de Guillermo Gascón Celdrán {.unnumbered}

Durante mi tercer año de carrera cursé la optativa de *Arquitectura Interna de Linux y Android* y cuando se acercaba el final de la asignatura Juan Carlos nos presentó diferentes propuestas de Trabajos de Fin de Grado relacionados con los conceptos aprendidos, entre las cuáles se encontraba esta. Desde aquél entonces le confirmé mi interés por realizar este proyecto ya que me motivaba mucho el desarrollar un dispositivo que posteriormente se pudiera usar para que otros alumnos, como yo lo fuí, aprendieran lo máximo posible sobre USB.

Cuando recibí la confirmación por parte de Juan Carlos de que podía realizar dicho TFG comencé a informarme sobre el protocolo USB, siguiendo todo lo que habíamos aprendido en clase y profundizando más con lecturas de libros y documentos. Este periodo de lectura inicial que hice me ha servido para entender los nuevos conceptos a los que nos hemos ido enfrentando durante el desarrollo de este trabajo.

Posteriormente, una vez finalizado el curso *2021-2022*, Juan Carlos nos ofreció prestarnos algo de material antes de verano para comenzar a investigar sobre los periféricos que ibamos a utilizar, así como con el entorno de desarrollo y el *framework V-USB*. Sin dudarlo acepté y durante los meses de verano estuve aprendiendo a usar tanto el anillo de *LED* circular, como un *LED RGB* individual que hace uso de un protocolo diferente para funcionar, además aprendí en profundidad como funciona el entorno de programación *AVR* y conseguí utilizar las diferentes herramientas que nos proporciona para compilar y subir nuevo código al microcontrolador con éxito.

Ya más entrados en el curso *2022-2023*, comencé a investigar sobre cómo estaba construido el dispositivo que tomamos de referencia, el *Blinkstick Strip*, consiguiendo comprender su firmware al completo y familiarizándome también con su construcción hardware. Esto me permitió crear una primera versión de prueba del firmware con la que pude hacer que el ordenador reconociera el microcontrolador bajo el nombre de *PwnedDevice* y con los parámetros definidos por mí.

Después de esto, comencé a investigar sobre cómo podíamos incluir dentro del firmware de prueba que ya teníamos el código necesario para utilizar el anillo de *LED*. Finalmente, tras realizar algunos ajustes de configuración y modificar alguna función, conseguí incluir con éxito una librería que nos permitía controlar el anillo por completo. Posteriormente, mejoré la eficiencia cuando se usa este periférico guardando ciertos datos sobre él que permitían ahorrar tiempo y cálculos.

Una vez implementado el anillo, me puse con la función de lectura. Esta función es la que se ejecuta cuando el ordenador pide al dispositivo un cierto número de datos y tuve que aprender cómo funciona *V-USB* por dentro y cómo gestiona este tipo de llamadas para finalmente implementar con éxito la función y hacer que cuando el dispositivo reciba una lectura devuelva la cadena *Hello World! I'm pwnedDevice ;)*.

Después de la función de lectura empecé a implementar la integración en el firmware de un sensor de temperatura, esta investigación me llevo mucho tiempo y finalmente me fue imposible integrar este periférico dentro de *V-USB*. Actualmente desconocemos el problema que ocasiona mezclar el sensor de temperatura con el *framework V-USB*, ya que sin este último, el sensor funciona como se espera.

Siguiendo con la implementación de periféricos, decidí integrar una pantalla *OLED* pequeña. Esta implementación si tuvo éxito y realicé las modificaciones necesarias en el firmware para que esta pequeña pantalla funcionara sin problemas.

A continuación, estuve aprendiendo a usar los timers internos del microcontrolador que terminamos usando, el *ATMega328p*. Esto me llevó un tiempo puesto que era mi primera vez trabajando con hardware de este tipo, pero finalmente conseguí generar diferentes señales *PWM* con las que pude controlar el buzzer y el diodo *LED*, haciendo sonar incluso una canción de cumpleaños feliz.

Por último dentro del desarrollo del firmware, integré el endpoint de tipo *INTERRUPT IN* e implementé con éxito la devolución de un buffer de 8 bytes relleno de información cada vez que se recibe una transferencia de este tipo.

Durante todo el desarrollo del firmware, he creado también diferentes utilidades que hacen uso de la librería *libusb* en *C* y en *Python*, que me permitieron probar los nuevos periféricos de una manera rápida debido a la simplicidad con la que se creaban los mensajes para interactuar con el microcontrolador.

A su vez, también estuve desarrollando los drivers que vienen dentro del paquete de drivers de ejemplo que ofrece este proyecto. Con ayuda de Juan Carlos, hemos conseguido exprimir las diferentes partes de la *API* que ofrece *USB* Core quedando un conjunto de ejemplo ideal para alumnos que se introducen en este campo.

Como último retoque, una vez tuvimos el prototipo hardware funcionando, cree una pequeña placa prototipo con los componentes soldados quedando un dispositivo *USB* más compacto que cuando se encontraba todo conectado a la placa de desarrollo.

Y estas han sido mis aportaciones al proyecto, junto con mucho tiempo de investigacion que ha acompañado a todas las etapas del desarrollo tanto software como hardware.


## Aportación de Javier Rodríguez-Avello Tapias {.unnumbered}
