<!-- Leave a blank line before the title -->

# Conclusiones

Después de haber realizado este proyecto, estamos más familiarizados sobre el funcionamiento a bajo nivel del protocolo USB, y la gran multitud de firmwares y funcionalidades que se pueden especificiar siguiendo este protocolo. Además, hemos estudiado un ejemplo concreto de firmware como es el V-USB [@objective-development], con la correspondiente complejidad que lleva el desarrollo de un firmware de estas características, haciendo uso de los distintos endpoints o los report-IDs para las distintas funcionalidades implementadas. 

Con el firmware de este proyecto, se espera que el desarrollo de software para realizar pruebas con periféricos relacionados sea mucho más fácil, y que la gente pueda contribuir con sus propios desarrollos a través de la plataforma GitHub, y que se puedan desarrollar drivers para hacer funcionar estos dispositivos. Además, se pretende extender los posibles casos prácticos que existen actualmente en la asignatura Arquitectura Interna de Linux y Android, dotando de un mayor abanico de posibilidades de programación con el uso de este firmware y tomando como ejemplo los periféricos utilizados.



## Extensión natural de la infraestructura para la asignatura Arquitectura interna de Linux y Android

<!--

Actualmente se cuenta con la placa Bee 2.0 en el que se integran, entre otros periféricos, un buzzer, display 7 segmentos, tres pulsadores, un LED RGB, etc, para la asignatura. Las posibilidades que existen de programación utilizando esta placa son infinitas, pero actualmente se encuentra la limitación del uso únicamente de drivers que utilizan endpoints de tipo Control, por lo que restringe las posibilidades de casos prácticos para proponer a los alumnos, ya que sólo se puede utilizar hardware específico. Utilizando el firmware de este proyecto se pueden extender la funcionalidad que se quiera implementar (añadiendo, por ejemplo, Report-IDs). Si se combina con la placa Bee 2.0, hay un abanico enorme de posibilidades para combinar periféricos, además de poder desarrollar drivers que utilicen endpoints de tipo Interrupt (como en uno de los drivers de ejemplo que se explican en este proyecto). Por lo que puede enriquecer enormemente los posibles casos prácticos a desarrollar en esta asignatura, y además, se puede aprovechar esta placa en su totalidad, incluso sin necesidad de utilizar una Raspberry (En este proyecto se han explicado dos posibles placas con dos microcontroladores que pueden ejecutar el firmware del proyecto). 

La gran ventaja del uso de esta placa con V-USB es que todo el hardware puede exponerse como un conjunto de dispositivos USB, para poder desarrollar funcionalidad a estos dispositivos es necesario estudiar los aspectos de bajo nivel del protocolo USB, uno de los puntos de la asignatura. 

-->

## Uso en otras asignaturas

### Firmware

<!--

La funcionalidad que se puede desarrollar utilizando V-USB puede ser enorme, con la posibilidad de utilizar distintos periféricos. Al exponerlos como un conjunto de dispositivos USB, es posible que cada alumno tenga un prototipo hardware distinto, por lo que el software que tiene que implementar en cada grupo es distinto (aprovechando la ventaja de V-USB para ejecutarse en microcontroladores que soporten la conexión simultánea de varios periféricos). El coste de cada microchip y los periféricos disponibles es relativamente bajo, como se explica en el siguiente apartado, es posible tener varias unidades de distintos microchips para aumentar los casos prácticos que se pueden desarrollar en la asignatura.

-->



### Diseño de hardware específico

<!--

Al poder exponer todos los periféricos como un sólo dispositivo USB, se pueden crear multitud de posibilidades combinando periféricos. Una ventada del hardware propuesto en este proyecto es que es de bajo coste, en comparación con el dispositivo Blinkstick utilizado actualmente en LIN, por lo que la compra de este material para todos los alumnos, o incluso la reposición del mismo en caso de rotura deja de ser un problema con esta propuesta. Además, como hemos mencionado anteriormente, no es necesario que todos los alumnos dispongan de un mismo prototipo hardware, por lo que el desarrollo práctico en cada grupo es diferente, haciendo que cada grupo desarrolle una funcionalidad distinta.

-->

## Valoración del TFG

<!--

Para poder llevar a cabo el desarrollo de este proyecto, hemos tenido que documentarnos sobre distintos dispositivos hardware e informarnos sobre ellos mediante sus correspondientes *datasheets*, lo cual nos ha hecho coger bastante experiencia para poder pensar en un prototipo que incluyera todos los dispositivos que queremos añadir, además de poder modificar un firmware concreto que va a contener el microchip del dispositivo en cuestión.

Además, hemos tenido que estudiar el protocolo USB en profundidad, con todos los elementos a bajo nivel que utiliza, como los paquetes URBs o los extremos de comunicación o *endpoints*, de tipo Control (estudiados en la asignatura LIN) o de tipo Interrupt (estudiado a fondo en este proyecto). El tener estos conocimientos en un protocolo ampliamente usado en todo el mundo es muy importante de cara al futuro, ya que es un sector muy demandado.

-->



## Trabajo futuro

Gracias a la flexibilidad que ofrece la plataforma que hemos construido, el trabajo que se puede realizar en un futuro sobre esta es realmente grande. Algunos de los ejemplos por donde se podría empezar a trabajar son:

- Ampliar periféricos soportados. Ya que la infraestructura permite la rápida integración de nuevos periféricos esta sería una muy buena ruta de trabajo. Se podrían integrar dispositivos que utilicen protocolos o transferencias diferentes para comunicarse con el host y así ampliar el espectro de prácticas que los docentes pueden generar usando el dispositivo.
- Desarrollo de controladores nuevos y más avanzados. Al igual que con los periféricos, también se pueden seguir desarrollando controladores más avanzados ya que los existentes están pensados para gente que se está iniciando en el mundo del desarrollo USB y su objetivo es ilustrar de manera sencilla el funcionamiento de la API USB del kernel Linux.
- Integrar nuevos estándares USB. Otro campo donde se podría avanzar es creando nuevo hardware que permita soportar estándares de USB más recientes como los USB 3.X o incluso USB 4.0. Esto permitiría desarrollar controladores que manejen transferencias masivas de datos ya que la velocidad de estos estándares maneja sin problemas este flujo masivo de bytes.
- Utilizar un microcontrolador con soporte USB por hardware. Para dar soporte a otros estándares de USB se podrían utilizar microcontroladores que traigan de manera nativa el protocolo USB implementado por hardware. Esto nos aseguraría un mejor rendimiento y mayor capacidad de procesamiento debido a que se libera a la CPU principal de toda la gestión del protocolo USB.
- Mejorar la configuración de perfiles. A pesar de que la configuración de perfiles ya es extremadamente sencilla, podrían pensarse nuevas formas de configurarlos que no requieran saber programar en C. Por ejemplo, podría habilitarse una interfaz de escritorio que se comunique con el dispositivo y elija una configuración concreta.