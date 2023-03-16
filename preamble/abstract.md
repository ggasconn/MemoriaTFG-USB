<!-- Leave a blank line before the title -->

# Resumen {.unnumbered}

El protocolo de comunicación USB, ampliamente usado en numerosos dispositivos en todo el mundo, resulta complejo a la hora de realizar modificaciones para determinados periféricos, sobre todo a la hora de realizar cambios en la programación del dispositivo en cuestión. En este proyecto se trabaja en la realización de un firmware para un chip determinado (ATTiny85) que sea capaz de realizar distintas funciones sobre periféricos, que trabajen con un driver cargado en Linux, todo ello programado en lenguaje C.

La complejidad de la tecnología/especificación USB unido al tiempo limitado que puede dedicarse a describir aspectos de bajo nivel de entrada-salida en las titulaciones de Informática, dificulta la adquisición de conocimientos sobre desarrollo de drivers USB, muy valorados en sectores estratégicos. Asimismo, para iniciarse en el desarrollo de este tipo de drivers es preciso disponer de hardware suficientemente sencillo y de bajo coste. Un ejemplo de dispositivo para iniciación es el dispostitivo Blinkstick Strip, que cuenta con 8 LEDs de colores cuyo estado puede alterarse individualmente.

En este proyecto se diseña y programa un dispositivo USB de bajo coste que pueda usarse masivamente como banco de pruebas para el desarrollo de drivers USB. Dicho dispositivo estará formado por varios componentes (LEDs, pantallas LCD, sensores, etc.), y permitirá al desarrollador familiarizarse con distintos aspectos y modos de interacción con el hardware USB, para poder afrontar la complejidad de USB de forma gradual. Además del diseño del hardware, y la construcción de un prototipo del mismo empleando un microcontrolador, como el ATTiny85, será necesario el desarrollo del firmware del dispositivo así como de un conjunto de drivers de ejemplo (preferentemente en Linux), para exponer al usuario los distintos componentes del dispositivo USB.

**palabras clave**: Firmware, USB, endpoint, kernel Linux, report-ID, AVR, V-USB.



# Abstract {.unnumbered}

The USB communication protocol, widely used in numerous devices around the world, is complex when making changes to certain peripherals, especially when making changes to the programming of the device in question. In this project we work on the realization of a firmware for a certain chip (ATTiny85) that is capable of performing different functions on peripherals, that work with a driver loaded in Linux, all programmed in C language.

The complexity of the USB technology/specification together with the limited time that can be devoted to describing low-level input-output aspects in Computer Science degrees, makes it difficult to acquire knowledge on USB driver development, highly valued in strategic sectors. Likewise, to start developing this type of drivers it is necessary to have sufficiently simple and low-cost hardware. An example of a starter device is the Blinkstick Strip device, which has 8 colored LEDs whose state can be individually altered.

In this project, a low-cost USB device is designed and programmed that can be used massively as a test bench for the development of USB drivers. Said device will be made up of several components (LEDs, LCD screens, sensors, etc.), and will allow the developer to become familiar with different aspects and modes of interaction with USB hardware, in order to gradually face the complexity of USB. In addition to the hardware design, and the construction of a prototype of it using a microcontroller, such as the ATTiny85, it will be necessary to develop the firmware of the device as well as a set of example drivers (preferably in Linux), to expose the user to the various components of the USB device.
