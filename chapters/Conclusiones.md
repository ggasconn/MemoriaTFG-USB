<!-- Leave a blank line before the title -->

# Conclusiones

El objetivo principal de este proyecto ha radicado en la construcción de una infraestructura de *hardware-software* de bajo coste, orientada al prototipado de dispositivos USB y al desarrollo de controladores para dichos dispositivos. Esta infraestructura se basa en microcontroladores *AVR* de *Atmel* y aprovecha el proyecto *V-USB*, el cual proporciona un firmware genérico para la implementación del protocolo USB mediante software. Sin embargo, nuestra iniciativa va más allá al extender las capacidades de *V-USB* para facilitar la creación de firmware USB capaz de gestionar múltiples periféricos simultáneamente.

Aunque existen alternativas como el uso de microcontroladores con soporte de USB por hardware, nuestra infraestructura no solo se destaca por su bajo costo, sino también por su gran versatilidad en el desarrollo de software a nivel de bajo nivel. En particular, brinda la oportunidad de experimentar tanto con el desarrollo de controladores USB en sistemas *Linux*, como con la implementación de firmware USB para la gestión de hardware, empleando programación de "*bare metal*".

Además, otro objetivo fundamental de este proyecto ha sido proporcionar un conjunto de controladores USB que ejemplifiquen la interacción con dispositivos de diversas naturalezas, como conjuntos de L*EDs RGB*, pantallas *OLED*, displays de *7 segmentos*, zumbadores y sensores de temperatura. Para tal fin, la infraestructura propuesta se complementa con una colección de controladores que hacen uso de una amplia gama de funciones de la *API* provista por el kernel *Linux* para el desarrollo de controladores USB. Estos controladores se implementan como módulos cargables del kernel de *Linux*, con el propósito de sentar las bases para futuras prácticas en el contexto de la asignatura *LIN*. Cabe mencionar que, si bien el desarrollo de controladores USB en espacio de usuario utilizando *libusb* queda fuera del alcance de este proyecto, los controladores del kernel proporcionados pueden adaptarse fácilmente a esta biblioteca.

Por último, se ha explorado la flexibilidad que ofrece la placa *Bee* como base para la implementación de dispositivos USB. Es importante destacar que la *Facultad de Informática* ya ha adquirido un número considerable de estas placas para diversos cursos, lo cual permite aprovechar la reutilización de este hardware como componente fundamental de la infraestructura USB, incrementando así la rentabilidad de la inversión realizada.



## Extensión natural de la infraestructura para la asignatura Arquitectura interna de Linux y Android

Actualmente en la optativa *Arquitectura interna de Linux y Android* se continúa usando el dispositivo mencionado durante esta memoria,  *Blinkstick Strip*, el cual presenta diversas limitaciones que impiden explorar al completo el potencial de USB. La creación de esta infraestructura va a suponer un gran avance en la asignatura ya que se van a poder ampliar de manera sustancial las prácticas con nuevos conocimientos y diferentes escenarios sobre los cuáles los alumnos van a poder practicar. Entre estos nuevos conocimientos que podrán impartirse, podemos destacar la posibilidad de explorar nuevos tipos de transferencias de datos con todo lo que esto conlleva. Se van a poder explorar nuevas partes de la API del kernel que anteriormente no podían ser usadas por lo que los alumnos van a aprender más detalles sobre USB y van a ser capaces de desarrollar controladores USB de complejidad más avanzada, todo ello por un coste relativamente bajo debido al precio de los materiales y además la facultad ya cuenta con hardware perfectamente integrado en el desarrollo como es la placa Bee. 

La placa Bee permite ampliar con diversos perféricos el dispositivo USB creado y puesto que ya se poseen suficientes placas Bee, la integración de esta nueva modalidad de prácticas en la asignatura sería relativamente sencilla de llevar a cabo. Gracias a que el firmware de la placa funciona sin modificaciones en diversas placas con microcontroladores AVR, se está valorando la posibilidad de crear un simple adaptador que permita utilizar la placa mencionada en la sección [REF] con la ya existente placa Bee.

Otra barrera que eliminaría esta infraestructura es el impedimento que supone que el dispositivo en uso actualmente no sea modificable por los docentes para adaptar las prácticas de manera dinámica, al menos de forma sencilla, ya que el microcontrolador se encuentra soldado en la placa y su programación para realizar cualquier modificación es relativamente compleja.

## Uso en otras asignaturas

El alcance de este proyecto no se limita a la ampliación del material de prácticas de la asignatura LIN, su finalidad educativa puede ir mucho más allá ya que esta infraestructura permite extender el desarrollo otras asignaturas donde se imparten algunos de los conocimientos que han sido empleados durante el proyecto. Una muy buena opción podría ser integrar el desarrollo del firmware del dispositivo como práctica de aquellas asignaturas en las que se trabaje con sistemas empotrados, desarrollo a bajo nivel en C o microcontroladores en gener

De igual manera ocurre con el hardware, ya que la plataforma es ampliamente evolucionable permite integrar estos avances en prácticas donde los alumnos puedan construir nuevos dispositivos agregando periféricos que puedan funcionar con el microcontrolador elegido. 

<!--

### Firmware

La funcionalidad que se puede desarrollar utilizando V-USB puede ser enorme, con la posibilidad de utilizar distintos periféricos. Al exponerlos como un conjunto de dispositivos USB, es posible que cada alumno tenga un prototipo hardware distinto, por lo que el software que tiene que implementar en cada grupo es distinto (aprovechando la ventaja de V-USB para ejecutarse en microcontroladores que soporten la conexión simultánea de varios periféricos). El coste de cada microchip y los periféricos disponibles es relativamente bajo, como se explica en el siguiente apartado, es posible tener varias unidades de distintos microchips para aumentar los casos prácticos que se pueden desarrollar en la asignatura.



### Diseño de hardware específico

Al poder exponer todos los periféricos como un sólo dispositivo USB, se pueden crear multitud de posibilidades combinando periféricos. Una ventada del hardware propuesto en este proyecto es que es de bajo coste, en comparación con el dispositivo Blinkstick utilizado actualmente en LIN, por lo que la compra de este material para todos los alumnos, o incluso la reposición del mismo en caso de rotura deja de ser un problema con esta propuesta. Además, como hemos mencionado anteriormente, no es necesario que todos los alumnos dispongan de un mismo prototipo hardware, por lo que el desarrollo práctico en cada grupo es diferente, haciendo que cada grupo desarrolle una funcionalidad distinta.

-->

## Valoración del TFG

El proyecto que hemos afrontado ha sido un gran reto personal para nosotros debido a la amplitud y complejidad del mismo. Partíamos de un dispositivo de ejemplo el cuál estaba bastante limitado en características lo que nos llevó a tener que elegir otra alternativa gracias a la cual aprendimos mucho durante todo el proceso de búsqueda. De igual manera, la parte software tampoco ha sido sencilla de implementar, ya que USB es un protocolo que asume mucha complejidad para permitir funcionar con dispositivos de índoles muy diversas, lo que nos ha llevado a tener que comprender muy bien todas sus partes para poder hacer un correcto uso del framework V-USB. Pero no todo han sido dificultades, la investigación que hemos realizado junto todo el trabajo que se ha llevado a cabo para crear la infraestructura nos ha hecho comprender de una manera mucho más clara el funcionamiento de USB, así como su integración con el kernel Linux y el desarrollo de prototipos con microcontroladores.

Dentro de los aspectos que más hemos trabajo podríamos destacar:

- El aprendizaje para programar un firmware completamente funcional en C, utilizando todo el marco de herramientas que nos proporciona AVR para la compilación del código y posterior subida al microcontrolador.
- El estudio profundo del stack USB en Linux y más concretamente de la API USB que integra, destacando su parte síncrona y asíncrona, junto con múltiples funciones y estucturas que permiten realizar transferencias de diversos tipos.
- El desarrollo de hardware utilizando microcontroladores y los diferentes componentes electrónicos que se necesitan para que este chip funcione con el puerto micro usb. Así como conceptos básicos de micro electrónica.

En definitiva, ha sido un proyecto bastante completo ya que se ha trabajado tanto con software como con hardware, creando una infraestructura que esperamos aporte mucho valor al aprendizaje de USB.



## Trabajo futuro

Gracias a la flexibilidad que ofrece la plataforma que hemos construido, el trabajo que se puede realizar en un futuro es realmente extenso. Algunos de los ejemplos con los que se podría empezar son:

1. Ampliar periféricos soportados. Ya que la infraestructura permite la rápida integración de nuevos periféricos esta sería una muy buena opción de trabajo para comenzar. Se podrían integrar dispositivos que utilicen protocolos o transferencias diferentes para comunicarse con el host y así ampliar el espectro de prácticas que los docentes pueden generar usando el dispositivo.
2. Desarrollo de controladores nuevos y más avanzados. Al igual que con los periféricos, también se pueden seguir desarrollando controladores más avanzados ya que los existentes están pensados para gente que se está iniciando en el mundo del desarrollo USB y su objetivo es ilustrar de manera sencilla el funcionamiento de la *API USB* del kernel Linux.
3. Integrar nuevos estándares USB. Otro campo donde se podría avanzar es creando nuevo hardware que permita soportar estándares USB más recientes como los *USB 3.X* o incluso USB 4.0. *Esto* permitiría desarrollar controladores que manejen transferencias masivas de datos ya que la velocidad de estos estándares maneja sin problemas este gran flujo de bytes.
4. Utilizar un microcontrolador con soporte USB por hardware. Para dar soporte a otros estándares de USB se podrían utilizar microcontroladores que traigan de manera nativa el protocolo USB implementado por hardware. Esto nos aseguraría un mejor rendimiento y mayor capacidad de procesamiento debido a que se libera a la *CPU* principal de toda la gestión de la comunicación USB.
5. Mejorar la configuración de perfiles. A pesar de que la configuración de perfiles ya es extremadamente sencilla, podrían pensarse nuevas formas de configurarlos que no requieran conocer el lenguaje *C*. Por ejemplo, podría habilitarse una interfaz de escritorio que se comunique con el dispositivo y elija una configuración concreta.