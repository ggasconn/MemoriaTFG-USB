<!-- Leave a blank line before the title -->

# Conclusions

The main objective of this project has been the construction of a low-cost hardware-software infrastructure, around 5€ per device, oriented to the prototyping of USB devices and the development of *drivers* for those devices. This infrastructure makes use of *AVR* microcontrollers from *Atmel* and is based on the *V-USB* project, which provides a generic firmware for the implementation of the USB protocol by software. It should be noted, however, that our initiative goes further by extending the capabilities of *V-USB* to facilitate the creation of USB firmware capable of managing multiple peripherals simultaneously.

Although there are alternatives such as the use of microcontrollers with hardware USB support, our infrastructure stands out not only for its low cost but also for its great versatility in enabling low-level software development. In particular, it provides the opportunity to experiment both with the development of USB drivers in GNU/Linux, as well as with the implementation of USB firmware for hardware management, using "*bare metal*" programming.

Another fundamental objective of this project has been to provide a set of USB *drivers* that exemplify the interaction with devices of diverse nature, such as RGB LED arrays, *OLED* displays, *7-segment* displays, buzzers, and temperature sensors. To this end, the proposed infrastructure is complemented with a collection of drivers that make use of a wide range of functions of the *API* provided by the *Linux* kernel for the development of USB drivers. These drivers are implemented as loadable modules of the *Linux* kernel, with the purpose of laying the groundwork for future practices in subjects dealing with Linux driver development, such as the undergraduate elective *Linux and Android Internal Architecture* (LIN) taught at UCM. It is worth mentioning that, although the development of USB software drivers in user space using *libusb* is outside the scope of this project, the kernel *drivers* can be used as a basis for the creation of drivers using this library.

Finally, the potential offered by the *Bee* board [@bee-repo] as a basic building block of the proposed infrastructure to facilitate USB device prototyping has been explored. The use of this board is motivated by its great flexibility for the creation of hardware practice environments, and by the fact that substantial investment has already been made to acquire a considerable number of them in the Faculties of Physical Sciences and Computer Science, for the realization of laboratories of different subjects.


## Natural extension of the infrastructure for the LIN elective

Currently, the LIN elective continues to use the USB device *Blinkstick Strip*. This device, described in detail in the \ref{sec:blinkstick} section of this report, has several limitations that prevent the full potential of USB from being explored.

The infrastructure created in our project will be a great opportunity to extend the practices of the course, allowing the use of new USB devices, and thus to study more deeply the characteristics of this technology. Among these new contents that can be taught, we can highlight the possibility of experimenting with additional modes of USB data transfers, and to perform through the practices a greater coverage of the kernel API for drivers, thus allowing the development of more complex USB drivers. In addition, this can be achieved at reduced cost due to the low price of the infrastructure components, and the reuse of the Bee board, already available for use in the labs. This board makes it possible to expand the created USB device with various parts. Thanks to the fact that the firmware of the developed infrastructure works with slight modifications in several boards with AVR microcontrollers, the possibility of creating a simple adapter that allows interconnecting in a compact way the board that integrates the microcontroller (as Nano \ref{sec:nano}), the Bee board and the circuit that integrates the USB port for connection to the *host* is being evaluated.  

Another barrier that would eliminate this infrastructure is the impediment that the *Blinkstick Strip* device (commercially purchased version) does not allow to modify the firmware in a simple way. Note that the microcontroller used by *Blinkstick Strip* is soldered on a custom-designed board, and its programming for any modification is relatively complex, as this board does not expose the pins to program the device directly.


## Use in other subjects

The infrastructure developed in this project is not limited to the extension of the practice material of the LIN subject. Its educational purpose can go much further, since this infrastructure is extensible (both at software and hardware level) to be used in other subjects where some of the knowledge related to the technologies used during the project are taught. 

A very good option could be to integrate the development of the *firmware* of the device as a practice of those subjects in which we work with embedded systems, low level development in C or microcontrollers in general. The same happens with the hardware, since the platform is widely extensible, allowing to transfer the possible extensions to practices where students can build new devices by adding peripherals that can work with the chosen microcontroller.


## Evaluation of the TFG

The project we have faced has been a great personal challenge for us due to its scope and complexity. We started with a basic development board --Digispark with ATTiny85 microcontroller--; time revealed important limitations in its use as a basic component of the infrastructure. This led us to choose another development alternative, and the search process was very enriching for us from a learning point of view. Similarly, the software part of the infrastructure has not been easy to implement either, since USB is an inherently complex protocol. This complexity has led us to have to understand very well all its features, in order to properly exploit the *firmware* provided by the V-USB framework. The research we have done, along with all the work that has been carried out to create the infrastructure has made us understand in a much clearer way the operation of USB, as well as its integration with the Linux kernel and the development of prototypes with microcontrollers.

Among the aspects that have demanded more work we could highlight the following:

- The learning to program a fully functional firmware in C, using the entire framework of tools that Atmel provides us for the compilation of the code and subsequent upload to an AVR microcontroller.

- The deep study of the *USB stack* in Linux and more specifically of the USB API that integrates, highlighting its synchronous and asynchronous part, along with multiple functions and structures that allow transfers of various types.

- The development of hardware using microcontrollers, and the different electronic components that are needed for this chip to work with the micro USB port. 

In short, we consider that it has been a very complete project since we have worked with both software and hardware, creating an infrastructure that we hope will add a lot of value to USB learning.


## Trabajo futuro

Gracias a la flexibilidad que ofrece la infraestructura desarrollada, el trabajo que se puede realizar en un futuro es realmente extenso. A continuación se enumeran algunas de las líneas de trabajo futuro:

1. **Ampliar periféricos soportados**. Ya que la infraestructura permite la rápida integración de nuevos periféricos, esta sería una muy buena opción para extender nuestro trabajo. Se podrían integrar dispositivos que utilicen protocolos o transferencias diferentes para comunicarse con el *host* y así ampliar el espectro de prácticas que los docentes pueden elaborar usando el dispositivo.

2. **Desarrollo de controladores más avanzados.** Al igual que con los periféricos, también se pueden seguir desarrollando *drivers* USB más avanzados. Nótese que los *drivers* proporcionados con la infraestructura propuesta están pensados para un público que se está iniciando en el mundo del desarrollo de *drivers* y su objetivo es ilustrar de manera sencilla el funcionamiento de la *API USB* del kernel Linux.

3. **Integrar nuevos estándares USB.** Otro campo donde se podría avanzar es creando nuevo hardware que permita soportar estándares USB más recientes como los *USB 3.X* o incluso USB 4.0. Esto permitiría desarrollar controladores que manejen transferencias masivas de datos ya que la velocidad de estos estándares maneja sin problemas este gran flujo de bytes.

4. **Experimentar con microcontroladores con soporte USB por hardware.** Para dar soporte a otros estándares de USB se podrían utilizar microcontroladores con soporte de USB por hardware. Esto garantizaría un mayor rendimiento y capacidad de procesamiento, ya que estos microcontroladores liberan a la *CPU* principal de la mayor parte de la gestión de la comunicación USB.

5. **Mejorar la configuración de perfiles.** A pesar de que la configuración de perfiles de hardware (selección de periféricos activos) ya es extremadamente sencilla, podrían pensarse nuevas formas de configurarlos, pero sin requerir conocimientos profundos de C y de la estructura del *firmware*. Por ejemplo, podría desarrollarse una aplicación con interfaz gráfica de usuario (GUI), que permita elegir cómodamente qué dispositivos se han de gestionar por el *firmware*, y se pueda compilar y cargar automáticamente el *firmware* con la configuración escogida en la plataforma USB.