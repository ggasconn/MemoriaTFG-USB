<!-- Leave a blank line before the title -->

# Introducción

En este capítulo introductorio se presenta la motivación de este proyecto, y se describen los objetivos y la planificación de las tareas para llevarlo a cabo.




## Motivación

El protocolo USB es ampliamente usado en todo el mundo para la interacción con periféricos de muy diversa naturaleza. Cualquier usuario puede conectar un dispositivo a su ordenador. Al hacerlo, el *driver* correspondiente instalado en el sistema reconoce el dispositivo y permite realizar las funciones para las que este ha sido desarrollado. Para lograr esta simplicidad de uso, el desarrollador del *driver* y el fabricante del dispositivo USB tienen que lidiar con la complejidad del protocolo USB. 

En las titulaciones actuales de Informática, el tiempo limitado que puede dedicarse a describir aspectos de bajo nivel de entrada-salida unido a la complejidad de la tecnología y especificación USB dificulta a los estudiantes la adquisición de conocimientos sobre diseño de dispositivos USB y desarrollo de *drivers* para estos. Para desarrollar contenidos prácticos sobre estos aspectos se ha disponer además de hardware USB suficientemente sencillo, versátil y de bajo coste, lo cual puede constituir un reto significativo.

En la Facultad de Informática de la Universidad Complutense de Madrid una de las pocas asignaturas que aborda el desarrollo de drivers USB es la asignatura "Arquitectura Interna de Linux y Android", optativa común ofertada a estudiantes de distintas titulaciones de grado. En esta memoria nos referiremos a esta asignatura como "LIN", ya que este es el nombre corto empleado administrativamente en la Facultad de Informática para referirse a ella. En las prácticas de laboratorio de LIN se hace uso del dispostitivo Blinkstick Strip [@blinkstick-strip], un dispositivo USB sencillo que cuenta con 8 LEDs de colores cuyo estado puede alterarse individualmente. La simplicidad de este dispositivo permite elaborar prácticas para familiarizarse con el desarrollo de *drivers* USB, especialmente aquellos implementados a nivel del kernel del sistema operativo. 

Blinkstick Strip, no obstante, tiene varias limitaciones. En primer lugar, este dispositivo es tan sencillo que impide explorar el potencial de USB en profundidad. Al fin y al cabo, solo permite alterar el estado de cada uno de los LEDs mediante el envío de mensajes de control sencillos al dispositivo. En consecuencia, las prácticas correspondientes no posibilitan la experimentación con otras modalidades de transferencia de datos recogidas por la especificación USB, y emplean un subconjunto muy reducido de las llamadas de API para desarrollo de *drivers* USB. En segundo lugar, su escasa versatilidad no justifica su precio (alrededor de 20€ por unidad) para posibilitar una adopción a gran escala del dispositivo en distintas asignaturas y titulaciones de grado en la facultad. Por último, aunque el firmware del Blinkstick Strip está disponible de forma gratuita, su fabricante no proporciona documentación sobre cómo realizar modificaciones del mismo empleando la platforma hardware comercial. Esto constituye una barrera para la utilización del dispositivo para prácticas de diseño de *firmware*, donde es preciso realizar programación *bare metal*. 

Con independencia del protocolo o tecnología de entrada-salida empleada, disponer de una plataforma hardware suficientemente flexible es un aspecto clave en prácticas de laboratorio centradas en la interacción con periféricos. En particular, la capacidad de añadir nuevos dispositivos de entrada-salida al sistema sin requerir incorporar mucho cableado adicional permite desarrollar un gran número de prácticas de laboratorio para así poder satisfacer las necesidades de distintas asignaturas. Asimismo, esto posibilita la introducción de pequeñas variaciones en los contenidos prácticos de una misma asignatura a lo largo del tiempo. Estos factores permiten extender el ciclo de vida de una plataforma hardware orientada a docencia, y con ello reducir sustancialmente el coste de equipamiento de laboratorio. 

La placa Bee [@bee-repo;@bee-board] desarrollada por el Prof. Christian Tenllado es un claro ejemplo de sistema diseñado para la construcción de plataformas hardware flexibles. Se trata de una placa de expansión para sistemas basados en el *pinout* del mini PC Raspberry Pi (v1-v4) que está provista de un conjunto de dispositivos periféricos sencillos, y permite la ampliación de estos mediante circuitos de polarización genéricos. Esta placa, utilizada en sus distintas versiones en asignaturas de grado de las Facultades de Informática y de Ciencias Físicas en la UCM, a pesar de no proporcionar soporte para USB, ha supuesto una gran inspiración para la realización de este TFG.  

 

<!--

muy valorados en sectores estratégicos. para iniciarse en el desarrollo de este tipo de drivers es preciso disponer de hardware suficientemente sencillo y de bajo coste.  

Para que todo esto se pueda dar, el protocolo USB tiene una complejidad enorme en cuanto a paquetes de comunicación entre la CPU y el dispositivo o la controladora USB, a parte de necesitar un driver instalado en el sistema operativo para que pueda interpretarlo. A pesar de lo sencillo que pueda parecer el hardware (2 cables de datos), hay muchos elementos en la comunicación USB que tienen lugar para que el dispositivo pueda ser reconocido.

Para poder estudiar todo el protocolo USB y la interacción entre el dispositivo y la CPU, en este proyecto hemos desarrollado un firmware en C que se comunica con diferentes sensores y dispositivos, utilizando un microcontrolador AVR. Así como un conjunto de drivers de ejemplo para mostrar el funcionamiento y la comunicación entre el ordenador y el dispositivo.



-->




## Objetivos

El objetivo principal de este proyecto ha sido construir una infraestructura hardware software de bajo coste que permita el prototipado de dispositivos USB y el desarrollo de drivers para estos dispositivos.  Nuestra infraestructura está basada en microcontroladores AVR de Atmel [@avr-atmel], y en el proyecto V-USB [@v-usb], que proporciona un *firmware* genérico para implementar soporte de USB por software. Nuestro proyecto extiende V-USB para facilitar la creación de *firmware* USB que  gestione múltiples periféricos simultáneamente. Aunque existen otras alternativas al uso del  framework de  V-USB (p.ej. el uso de microcontroladores con soporte de USB por hardware), la infraestructura propuesta no solo tiene bajo coste, sino que también ofrece gran versatilidad para el desarrollo de software de bajo nivel. En particular, permite experimentar tanto con el desarrollo de drivers USB en sistemas provistos de sistema operativo, así como como la implementación de *firmware USB* para la gestión de hardware, empleando programación *bare metal*.  En este TFG se ha realizado desarrollo en ambos ámbitos.  

Otro objetivo del proyecto es proporcionar un conjunto de drivers USB que ilustren la interacción con dispositivos de distinta naturaleza, como conjuntos de LEDs RGB, pantallas LCD, displays 7 segmentos, zumbadores o sensores de tempera. Para ello la infraestructura propuesta se acompaña de una colección de drivers que emplean una amplia colección de funciones de la API proporcionada por el kernel Linux para el desarrollo de drivers USB. Estos drivers se implementan como módulos cargables del kernel Linux, para así servir de base para futuras prácticas de la asignatura LIN. Aunque el desarrollo de drivers USB en espacio de usuario con *libusb* [@libusb] queda fuera del ámbito de este TFG, los drivers del kernel proporcionados son fácilmente adaptables a esta biblioteca. 

Finalmente, dada la flexibilidad proporcionada por la placa Bee [@bee-repo;@bee-board], en este proyecto se han estudiado formas de emplear dicha placa como base para la implementación de dispositivos USB. Nótese que la Facultad de Informática ya ha adquirido un buen número de estas placas para distintas asignaturas, por lo que la reutilización de este hardware como bloque básico de la infraestructura USB permite amortizar aún más la inversión realizada. 



<!-- 

Para la realización del proyecto, se han contemplado distintas alternativas, en lugar de usar la librería V-USB. Entre ellas, la posibilidad de utilizar dispositivos hardware que implementen de fábrica la gestión del protocolo USB, pero las ventajas de esta librería son superiores, así como los conocimientos que podamos adquirir trabajando en C con los elementos de bajo nivel de este protocolo, que de otra forma no podríamos haber adquirido.¡



-->



## Plan de trabajo {#cap:hw}

En este proyecto se han desarrollado dos prototipos hardware de la infraestructura. El primer prototipo está basado en el microcontrolador ATTiny85 --mismo chip empleado por el dispositivo Blinkstick Strip. El segundo prototipo hace uso del micro controlador ATMega328p, que supera las limitaciones del anterior microcontrolador, descritas en detalle en el capítulo \ref{#cap:hw}. Teniendo esto en cuenta, se llevó a cabo un plan de trabajo, que consta de las siguientes tareas (ver Fig \ref{fig:gantt}) :

T1) 

: Aquí va la descripción



T2)

: Aquí va la descripción



<!-- 

https://www.uv.es/wikibase/doc/cas/pandoc_manual_2.7.3.wiki?82

-->







T1

:	Estudio de los diferentes periféricos y placa Digispark bajo entorno Arduino

T

: Estudio del dispositivo *Blinkstick Strip* y su firmware de código abierto

T

: Estudio de la librería V-USB

T

: Creación de un firmware de prueba para testear V-USB en la placa Digispark

T

: Estudio de las funciones de E/S utilzando los registros del ATTiny85

T

: Desarrollo y puesta en marcha del diodo led a nivel de registros

T

: Desarrollo y puesta en marcha del anillo de led a nivel de registros

T

: Desarrollo y puesta en marcha del buzzer a nivel de registros

T

: Desarrollo y puesta en marcha del display 7 segmentos a nivel de registros

T

: Estudio del circuito del conector micro USB

T

: Adaptación del circuito USB a Arduino Nano usando ATTMega328P

T

: Modificación del firmware para funcionar con chip ATTMega328P

T

: Adaptación de herramienta de debug a nuestro prototipo

T

: Desarrollo y puesta en marcha de la pantalla OLED a nivel de registros

T

: Estudio de los temporizadores incluidos en el ATMega328P

T

: Estudio de señales PWM y su generación a través de temporizadores

T

: Diseño del firmware para la utilización de perfiles

T

: Implementación de perfiles hardware a través de macros

T

: Migración de la implementación del led a PWM

T

: Migración de la implementación del buzzer a PWM

T

: Montaje del prototipo final en placa perforada





Para el desarrollo de este proyecto, se han mantenido distintas reuniones con los directores del proyecto y entre los desarrolladores, que han ido creciendo de manera exponencial conforme ha ido avanzado el desarrollo. En la parte inicial del proyecto, se ha estudiado el funcionamiento de la librería V-USB con distintos proyectos que utilizan otros microchips, para ver qué uso se hacen de los distintos elementos del firmware y las distintas funciones empleadas en el código. Se ha utilizado el software Wireshark [@sw-wireshark] para estudiar la comunicación USB, es decir, los endpoints usados y los paquetes URBs en la comunicación. También se ha tenido en cuenta la configuración del software y la posibilidad de utilizar, por ejemplo, interrupciones.



Para la integración del código, se ha utilizado GitHub.




## Organización de la memoria

El resto de esta memoria se ha dividido en los siguientes capítulos:

- El **capítulo 2** describe las características de bajo nivel del protocolo USB, así como sus abstracciones principales, como es el caso de los descriptores, *endpoints*, o los URBs.

- El **capítulo 3** proporciona una introducción general al proyecto V-USB y analiza la estructura del código fuente del *firmware* proporcionado. Este capítulo también discute las posibles alternativas al uso de V-USB para la construcción de infraestructuras hardware software como la propuesta de este proyecto.

- El **capítulo 4** presenta las características de los distintos microcontroladores utilizados en el proyecto, los detalles técnicos de las placas empleadas en diversas fases del proyecto, y los aspectos hardware de los periféricos soportados por nuestro prototipo.

- El **capítulo 5** analiza en detalle todos los elementos y funcionalidades implementadas en el *firmware* del prototipo, que constituye una extensión del proyecto V-USB.

- En **capítulo 6** describe en detalle los drivers USB de ejemplo que gestionan los dispositivos periféricos con soporte en el firmware.

- El **capítulo 7** recoge las principales conclusiones del TFG y enumera las líneas de trabajo futuro

  

  <!--

   se hace una reflexión sobre el posible uso de este proyecto para otros tipos de trabajos y pruebas, así como su integración en la asignatura de Arquitectura Interna de Linux y Android, impartida en esta facultad.

  -->

Finalmente, esta memoria consta de tres apéndices. En el primero se describen las instrucciones para poder analizar los paquetes URBs de USB con el software Wireshark. En el segundo apéndice se proporciona una guía que detalla el proceso para poder *flashear* el firmware en la placa del prototipo. **El tercer y último apéndice enumera las contribuciones aportadas por cada integrante de este Trabajo Fin de Grado al proyecto.**

