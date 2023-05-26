<!-- Leave a blank line before the title -->

# Introducción

En este capítulo introductorio se presenta la motivación de este proyecto, y se describen los objetivos y la planificación de las tareas para llevarlo a cabo.




## Motivación

El protocolo USB se usa ampliamente en todo el mundo para la interacción con periféricos de muy diversa naturaleza. Cualquier usuario puede conectar un dispositivo a su ordenador. Al hacerlo, el *driver* correspondiente instalado en el sistema reconoce el dispositivo y permite realizar las funciones para las que este ha sido desarrollado. Para lograr esta simplicidad de uso, el desarrollador del *driver* y el fabricante del dispositivo USB tienen que lidiar con la complejidad del protocolo USB. 

En las titulaciones actuales de grado en Informática, el tiempo limitado que puede dedicarse a describir aspectos de bajo nivel de entrada-salida unido a la complejidad de la tecnología y especificación USB dificulta a los estudiantes la adquisición de conocimientos sobre diseño de dispositivos USB y desarrollo de *drivers* para estos. Para desarrollar contenidos prácticos sobre estos aspectos se ha disponer además de hardware USB suficientemente sencillo, versátil y de bajo coste, lo cual puede constituir un reto significativo.

En la Facultad de Informática de la Universidad Complutense de Madrid una de las pocas asignaturas que aborda el desarrollo de *drivers* USB es la asignatura *Arquitectura Interna de Linux y Android*, optativa común ofertada a estudiantes de distintas titulaciones de grado. En esta memoria nos referiremos a esta asignatura como "LIN", ya que este es el nombre corto empleado administrativamente en la Facultad de Informática para referirse a ella. En las prácticas de laboratorio de LIN se hace uso del dispostitivo Blinkstick Strip [@blinkstick-strip], un dispositivo USB sencillo que cuenta con 8 LEDs de colores cuyo estado puede alterarse individualmente. La simplicidad de este dispositivo permite elaborar prácticas para familiarizarse con el desarrollo de *drivers* USB, especialmente aquellos implementados a nivel del kernel del sistema operativo. 

Blinkstick Strip, no obstante, tiene varias limitaciones. En primer lugar, este dispositivo es tan sencillo que impide explorar el potencial de USB en profundidad. Al fin y al cabo, solo permite alterar el estado de cada uno de los LEDs mediante el envío de mensajes de control sencillos al dispositivo. En consecuencia, las prácticas correspondientes no posibilitan la experimentación con otras modalidades de transferencia de datos recogidas por la especificación USB, y, por tanto, emplean un subconjunto muy reducido de las llamadas de API para desarrollo de *drivers* USB. En segundo lugar, su escasa versatilidad no justifica su precio (alrededor de 20€ por unidad) para posibilitar una adopción a gran escala del dispositivo en distintas asignaturas y titulaciones de grado en la facultad. Por último, aunque el firmware del Blinkstick Strip está disponible de forma gratuita, su fabricante no proporciona documentación sobre cómo realizar modificaciones del mismo empleando la platforma hardware comercial. Esto constituye una barrera para la utilización del dispositivo para prácticas de diseño de *firmware*, donde es preciso realizar programación *bare metal*. 

Con independencia del protocolo o tecnología de entrada-salida empleada, disponer de una plataforma hardware suficientemente flexible es un aspecto clave en prácticas de laboratorio centradas en la interacción con periféricos. En particular, la capacidad de añadir nuevos dispositivos de entrada-salida al sistema sin requerir incorporar mucho cableado adicional permite desarrollar un gran número de prácticas de laboratorio para así poder satisfacer las necesidades de distintas asignaturas. Asimismo, esto posibilita la introducción de pequeñas variaciones en los contenidos prácticos de una misma asignatura a lo largo del tiempo. Estos factores permiten extender el ciclo de vida de una plataforma hardware orientada a docencia, y con ello reducir sustancialmente el coste de equipamiento de laboratorio. 

La placa Bee [@bee-repo;@bee-board] desarrollada por el Prof. Christian Tenllado es un claro ejemplo de sistema diseñado para la construcción de plataformas hardware flexibles. Se trata de una placa de expansión para sistemas basados en el *pinout* del mini PC Raspberry Pi (v1-v4) que está provista de un conjunto de dispositivos periféricos sencillos, y permite la ampliación de estos mediante circuitos de polarización genéricos. Esta placa, utilizada en sus distintas versiones en asignaturas de grado de las Facultades de Informática y de Ciencias Físicas en la UCM, a pesar de no proporcionar soporte para USB, ha supuesto una gran inspiración para la realización de este TFG.  



## Objetivos

El objetivo principal de este proyecto ha sido construir una infraestructura hardware software de bajo coste que permita el prototipado de dispositivos USB y el desarrollo de drivers para estos dispositivos.  Nuestra infraestructura está basada en microcontroladores AVR de Atmel [@avr-atmel], y en el proyecto V-USB [@v-usb], que proporciona un *firmware* genérico para implementar soporte de USB por software. Nuestro proyecto extiende V-USB para facilitar la creación de *firmware* USB que  gestione múltiples periféricos simultáneamente. Aunque existen otras alternativas al uso del  framework de  V-USB (p.ej. el uso de microcontroladores con soporte de USB por hardware), la infraestructura propuesta no solo tiene bajo coste, sino que también ofrece gran versatilidad para el desarrollo de software de bajo nivel. En particular, permite experimentar tanto con el desarrollo de *drivers* USB en sistemas provistos de sistema operativo, así como como la implementación de *firmware USB* para la gestión de hardware, empleando programación *bare metal*.  En este TFG se ha realizado desarrollo en ambos ámbitos.  

Otro objetivo del proyecto es proporcionar un conjunto de *drivers* USB que ilustren la interacción con dispositivos de distinta naturaleza, como conjuntos de LEDs RGB, pantallas LCD, displays 7 segmentos, zumbadores o sensores de temperatura. Para ello la infraestructura propuesta se acompaña de una colección de drivers que emplean una amplia colección de funciones de la API proporcionada por el kernel Linux para el desarrollo de drivers USB. Estos *drivers* se implementan como módulos cargables del kernel Linux, para así servir de base para futuras prácticas de la asignatura LIN. Aunque el desarrollo de *drivers* USB en espacio de usuario con *libusb* [@libusb] queda fuera del ámbito de este TFG, los drivers del kernel proporcionados sirven de base para la creación de controladores en espacio de usuario desarrollados con esta biblioteca. 

Finalmente, dada la flexibilidad proporcionada por la placa Bee [@bee-repo;@bee-board], en este proyecto se han estudiado formas de emplear dicha placa como base para la implementación de dispositivos USB. Nótese que la Facultad de Informática ya ha adquirido un buen número de estas placas para distintas asignaturas, por lo que la reutilización de este hardware como bloque básico de la infraestructura USB permite amortizar aún más la inversión realizada. 



## Plan de trabajo

En este proyecto se han desarrollado dos prototipos hardware de la infraestructura. El primer prototipo está basado en el microcontrolador ATTiny85 --mismo chip empleado por el dispositivo Blinkstick Strip. El segundo prototipo supera las limitaciones del anterior (descritas en detalle en el capítulo \ref{cap:hw}), y emplea el microcontrolador ATMega328p. Durante el proyecto ha sido preciso familiarizarse con la programación sobre los dos citados microcontroladores --integrados en las placas Digispark [@digisparkboard-image] y Nano [@atmegaboard-image], respectivamente--, así como con una amplia colección de dispositivos de entrada-salida empleados en los prototipos. Entre estos dispositivos se incluye un anillo circular de LEDs RGB, un zumbador (o *buzzer* ), una pantalla OLED y un display 7 segmentos (integrado en la placa Bee [@bee-repo]).

Teniendo esto en cuenta, se llevó a cabo un plan de trabajo, que consta de las siguientes tareas (ver Fig \ref{fig:gantt}):



![Diagrama de Gantt del proyecto](img/gantt.png){width=90% #fig:gantt}

<!-- 

https://www.uv.es/wikibase/doc/cas/pandoc_manual_2.7.3.wiki?82

-->




T1

:	Estudio de la placa Digispark y análisis del control de los diferentes periféricos bajo entorno Arduino

T2

: Estudio del dispositivo *Blinkstick Strip* y de su firmware de código abierto

T3

: Análisis en profundidad del *firmware* del proyecto V-USB

T4

: Desarrollo de primera versión del firmware V-USB sobre la placa Digispark (sin soporte completo de périféricos)

T5

: Estudio de las funciones de E/S utilzando los registros del microcontrolador ATTiny85

T6

: Ampliación de la funcionalidad  del firmware V-USB desarrollado en T4 con soporte de control de dispositivos básicos 

T7

: Desarrollo del control a nivel de registros (sin usar entorno Arduino) del Diodo LED de la placa Digispark 

T8

: Desarrollo del control a nivel de registros del anillo circular de LEDs

T9

: Desarrollo del control a nivel de registros del *buzzer*

T10

: Desarrollo del control a nivel de registros del display 7 segmentos

T11

: Estudio de alternativas de diseño de circuitería extra asociada al conector micro USB requerido por el segundo prototipo

T12

: Diseño del prototipo hardware basado en ATMega328p

T13

: Integración de placa Nano (con ATMega328p) con circuito del conector micro USB   

T14

: Modificación del firmware para funcionar con chip ATMega328p

T15

: Adaptación de herramienta de depuración a nuestro prototipo

T16

: Desarrollo del control a nivel de registros de la pantalla OLED 

T17

: Estudio de los temporizadores incluidos en el ATMega328p

T18

: Estudio de señales PWM y su generación a través de temporizadores

T19

: Refactorización del *firmware* de V-USB para la utilización de perfiles

T20

: Implementación de perfiles hardware a través de macros

T21

: Migración de la implementación del controlador de LED a PWM

T22

: Migración de la implementación del controlador del buzzer a PWM

T23

: Montaje del prototipo final (basado en ATMega328p) en placa perforada

T24

: Redacción de la memoria del TFG


Durante el desarrollo del proyecto se han mantenido distintas reuniones con los directores del TFG. La frecuencia de las mismas se ha ido incrementando de forma gradual conforme ha ido avanzado el desarrollo. Estas reuniones han servido para resolver problemas de diseño, plantear las diferentes fases y solucionar múltiples dudas y limitaciones que han ido surgiendo durante todo el proyecto.


Para la gestión tanto del código como de las fuentes de la memoria del proyecto se han utilizado repositorios compartidos en la plataforma GitHub. Los enlaces a ambos repositorios son:
* **Código fuente**. https://github.com/ggasconn/TFG_USB
* **Memoria**. https://github.com/ggasconn/MemoriaTFG-USB/



## Organización de la memoria

El resto de esta memoria se ha dividido en los siguientes capítulos:

- El **capítulo 2** describe las características de bajo nivel del protocolo USB, así como sus abstracciones principales, como es el caso de los descriptores, *endpoints*, o los URBs.

- El **capítulo 3** proporciona una introducción general al proyecto V-USB y analiza la estructura del código fuente del *firmware* proporcionado. Este capítulo también discute las posibles alternativas al uso de V-USB para la construcción de infraestructuras hardware software como la propuesta de este proyecto.

- El **capítulo 4** presenta las características de los distintos microcontroladores utilizados en el proyecto, los detalles técnicos de las placas empleadas en diversas fases del proyecto, y los aspectos hardware de los periféricos soportados por nuestro prototipo.

- El **capítulo 5** analiza en detalle todos los elementos y funcionalidades implementadas en el *firmware* del prototipo, que constituye una extensión del proyecto V-USB.

- En **capítulo 6** describe en detalle los drivers USB de ejemplo que gestionan los dispositivos periféricos con soporte en el firmware.

- El **capítulo 7** recoge las principales conclusiones del TFG y enumera las líneas de trabajo futuro


Al final de esta memoria pueden encontrarse también los siguientes cinco apéndices:

*  El **apéndice A** enumera las contribuciones aportadas por cada integrante de este Trabajo Fin de Grado al proyecto. 
* En el **apéndice B**  se describen las instrucciones para poder analizar los paquetes URBs de USB con el software Wireshark. 
* En el **apéndice C** se proporciona una guía que detalla el proceso para poder *flashear* el firmware en la placa del prototipo. 
* Finalmente los **apéndices D y E** recogen la traducción al inglés de este capítulo introductorio así como de las conclusiones.

