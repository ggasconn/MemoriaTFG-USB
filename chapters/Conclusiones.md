<!-- Leave a blank line before the title -->

# Conclusiones

El objetivo principal de este proyecto ha radicado en la construcción de una infraestructura hardware-software de bajo coste, entorno a unos 5€ por dispositivo, orientada al prototipado de dispositivos USB y al desarrollo de *drivers* para dichos dispositivos. Esta infraestructura hace uso de microcontroladores *AVR* de *Atmel* y está basada en el proyecto *V-USB*, el cual proporciona un firmware genérico para la implementación del protocolo USB por software. Cabe destacar, no obstante, que nuestra iniciativa va más allá al extender las capacidades de *V-USB* para facilitar la creación de firmware USB capaz de gestionar múltiples periféricos simultáneamente.

Aunque existen alternativas como el uso de microcontroladores con soporte de USB por hardware, nuestra infraestructura no solo destaca por su bajo coste, sino también por su gran versatilidad para posibilitar el desarrollo de software a bajo nivel. En particular, brinda la oportunidad de experimentar tanto con el desarrollo de drivers USB en GNU/Linux, así como con la implementación de firmware USB para la gestión de hardware, empleando programación "*bare metal*".

Otro objetivo fundamental de este proyecto ha sido proporcionar un conjunto de *drivers* USB que ejemplifiquen la interacción con dispositivos de diversa naturaleza, como conjuntos de LEDs RGB, pantallas *OLED*, displays *7 segmentos*, zumbadores y sensores de temperatura. Para tal fin, la infraestructura propuesta se complementa con una colección de controladores que hacen uso de una amplia gama de funciones de la *API* provista por el kernel *Linux* para el desarrollo de controladores USB. Estos controladores se implementan como módulos cargables del kernel de *Linux*, con el propósito de sentar las bases para futuras prácticas en asignaturas que aborden el desarrollo de drivers en Linux, como la asignatura optativa de grado *Arquitectura Interna de Linux y Android* (LIN) impartida en la UCM. Cabe mencionar que, si bien el desarrollo de controladores software USB en espacio de usuario utilizando *libusb* queda fuera del alcance de este proyecto, los *drivers* del kernel pueden emplearse como base para la creación de controladores empleando esta biblioteca.

Por último, se ha explorado el potencial que ofrece la placa *Bee* [@bee-repo] como bloque básico de la infraestructura propuesta para facilitar el prototipado de dispositivos USB. El uso de esta placa está motivado por su gran flexibilidad para creación de entornos de prácticas de hardware, y por el hecho de que ya se ha realizado una inversión sustancial para adquirir un número considerable de ellas en las Facultades de Ciencias Físicas y de Informática, para la realización de laboratorios de distintas asignaturas.



## Extensión natural de la infraestructura para la asignatura LIN

Actualmente en la asignatura optativa LIN  se continúa usando el dispositivo USB *Blinkstick Strip*. Este dispositivo, descrito en detalle en la sección \ref{sec:blinkstick} de esta memoria, presenta diversas limitaciones que impiden explorar al completo el potencial de USB.

La infraestructura creada en nuestro proyecto va a suponer una gran oportunidad para extender las prácticas de la asignatura, permitiendo emplear nuevos dispositivos USB, y con ello estudiar más profundamente las características de esta tecnología. Entre estos nuevos contenidos que podrán impartirse, podemos destacar la posibilidad de experimentar con modalidades adicionales de transferencias de datos en USB, y realizar mediante las prácticas una mayor cobertura de la API del kernel para drivers, permitiendo con ello el desarrollo de controladores USB más complejos. Además, esto puede lograrse con un coste reducido debido al bajo precio de los componentes de la infraestructura, y a la reutilización de la placa Bee, ya disponible para su uso en los laboratorios. Esta placa permite ampliar con diversos perféricos el dispositivo USB creado. Gracias a que el firmware de la infraestructura desarrollada funciona con leves modificaciones en diversas placas con microcontroladores AVR, se está valorando la posibilidad de crear un simple adaptador que permita interconectar de forma compacta la placa que integra el microcontrolador  (como Nano \ref{sec:nano}) , la placa Bee y el circuito que integra el puerto USB para conexión al *host*.  

Otra barrera que eliminaría esta infraestructura es el impedimento que supone que el dispositivo *Blinkstick Strip* (versión adquirida comercialmente) no permite modificar el firmware de forma sencilla. Nótese que el microcontrolador usado por *Blinkstick Strip* se encuentra soldado en una placa diseñada a medida, y su programación para realizar cualquier modificación es relativamente compleja, ya que esta placa no expone los pines para programar el dispositivo de forma directa. 



## Uso en otras asignaturas

La infraestructura desarrollada en este proyecto no se limita a la ampliación del material de prácticas de la asignatura LIN. Su finalidad educativa puede ir mucho más allá, ya que esta infraestructura es extensible (tanto a nivel de software como de hardware) para ser utilizada en otras asignaturas donde se imparten algunos de los conocimientos afines a las tecnologías empleadas durante el proyecto. 

Una muy buena opción podría ser integrar el desarrollo del *firmware* del dispositivo como práctica de aquellas asignaturas en las que se trabaje con sistemas empotrados, desarrollo a bajo nivel en C o microcontroladores en general. De igual manera ocurre con el hardware, ya que la plataforma es ampliamente extensible, permitiendo  trasladar las posibles extensiones a prácticas donde los alumnos puedan construir nuevos dispositivos agregando periféricos que puedan funcionar con el microcontrolador elegido. 



## Valoración del TFG

El proyecto que hemos afrontado ha sido un gran reto personal para nosotros debido a la amplitud y complejidad del mismo. Partíamos de una placa básica de desarrollo --Digispark con microcontrolador ATTiny85--; el tiempo reveló importantes limitaciones en su uso como componente básico de la infraestructura. Esto nos llevó a tener que elegir otra alternativa de desarrollo, y su proceso búsqueda fue muy enriquecedor para nosotros desde el punto de vista del aprendizaje. De igual manera, la parte software de la infraestructura tampoco ha sido sencilla de implementar, ya que USB es un protocolo inherentemente complejo. Esta complejidad nos ha llevado a tener que comprender muy bien todas sus características, para poder explotar correctamente el *firmware* proporcionado por el framework V-USB. La investigación que hemos realizado, junto todo el trabajo que se ha llevado a cabo para crear la infraestructura nos ha hecho comprender de una manera mucho más clara el funcionamiento de USB, así como su integración con el kernel Linux y el desarrollo de prototipos con microcontroladores.

Dentro de los aspectos que más trabajo han demandado podríamos destacar lo siguiente:

- El aprendizaje para programar un firmware completamente funcional en C, utilizando todo el marco de herramientas que nos proporciona Atmel para la compilación del código y posterior subida a un microcontrolador AVR.
- El estudio profundo del *stack USB* en Linux y más concretamente de la API USB que integra, destacando su parte síncrona y asíncrona, junto con múltiples funciones y estucturas que permiten realizar transferencias de diversos tipos.
- El desarrollo de hardware utilizando microcontroladores, y los diferentes componentes electrónicos que se necesitan para que este chip funcione con el puerto micro USB. 

En definitiva, consideramos que ha sido un proyecto bastante completo ya que se ha trabajado tanto con software como con hardware, creando una infraestructura que esperamos aporte mucho valor al aprendizaje de USB.



## Trabajo futuro

Gracias a la flexibilidad que ofrece la infraestructura desarrollada, el trabajo que se puede realizar en un futuro es realmente extenso. A continuación se enumeran algunas de las líneas de trabajo futuro:

1. **Ampliar periféricos soportados**. Ya que la infraestructura permite la rápida integración de nuevos periféricos, esta sería una muy buena opción para extender nuestro trabajo. Se podrían integrar dispositivos que utilicen protocolos o transferencias diferentes para comunicarse con el *host* y así ampliar el espectro de prácticas que los docentes pueden elaborar usando el dispositivo.
2. **Desarrollo de controladores más avanzados.** Al igual que con los periféricos, también se pueden seguir desarrollando *drivers* USB más avanzados. Nótese que los *drivers* proporcionados con la infraestructura propuesta están pensados para un público que se está iniciando en el mundo del desarrollo de *drivers* y su objetivo es ilustrar de manera sencilla el funcionamiento de la *API USB* del kernel Linux.
3. **Integrar nuevos estándares USB.** Otro campo donde se podría avanzar es creando nuevo hardware que permita soportar estándares USB más recientes como los *USB 3.X* o incluso USB 4.0. Esto permitiría desarrollar controladores que manejen transferencias masivas de datos ya que la velocidad de estos estándares maneja sin problemas este gran flujo de bytes.
4. **Experimentar con microcontroladores con soporte USB por hardware.** Para dar soporte a otros estándares de USB se podrían utilizar microcontroladores con soporte de USB por hardware. Esto garantizaría un mayor rendimiento y capacidad de procesamiento, ya que estos microcontroladores liberan a la *CPU* principal de la mayor parte de la gestión de la comunicación USB.
5. **Mejorar la configuración de perfiles.** A pesar de que la configuración de perfiles de hardware (selección de periféricos activos) ya es extremadamente sencilla, podrían pensarse nuevas formas de configurarlos, pero sin requerir conocimientos profundos de C y de la estructura del *firmware*. Por ejemplo, podría desarrollarse una aplicación con interfaz gráfica de usuario (GUI), que permita elegir cómodamente qué dispositivos se han de gestionar por el *firmware*, y se pueda compilar y cargar automáticamente el *firmware* con la configuración escogida en la plataforma USB.