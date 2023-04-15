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

## Plan de trabajo

Para el desarrollo de este proyecto, se han mantenido distintas reuniones con los directores del proyecto y entre los desarrolladores. En la parte inicial del proyecto, se ha estudiado el funcionamiento de la librería V-USB con distintos proyectos que utilizan otros microchips, para ver qué uso se hacen de los report-IDs y las distintas funciones empleadas en el código. Se ha utilizado el software Wireshark [@sw-wireshark] para estudiar los endpoints usados y los paquetes URBs en la comunicación, y así estudiar la configuración del software y la posibilidad de utilizar, por ejemplo, interrupciones.

Para el desarrollo de los periféricos que se van a usar conjuntamente en la placa, Guillermo se ha encargado de estudiar y desarrollar el código correspondiente al sensor de temperatura y Javier al correspondiente con la pantalla LCD 2x16, que posteriormente se ha utilizado una pantalla LCD OLED.

Para la integración del código, se ha utilizado GitHub.




## Organización de la memoria

Para la realización de esta memoria, se ha dividido la misma en varios capítulos. En el primero, se explica a modo de introducción el hardware utilizado y algunos aspectos de bajo nivel sobre la librería utilizada. En el siguiente capítulo, se explican los elementos hardware utilizados, como el anillo circular de leds o la pantalla LCD OLED. Después, se explica con detalle el microcontrolador utilizado finalmente para el prototipo final, ya que éste no tiene las limitaciones que contábamos con el ATTiny85, al poder tener más líneas de comunicación con él.

En la parte de apéndices, contamos con las aportaciones de cada integrante del proyecto, así como distintas instrucciones para poder analizar los paquetes URBs de USB con el software Wireshark, o también para poder *flashear* nuestro propio firmware, con indicaciones sobre los pines concretos a utilizar en la placa.


# Introduction

The following sections explain the reasons for carrying out the project, the objectives and the planning of the tasks to carry it out.


## Motivation

The USB protocol is widely used throughout the world for communication with almost any peripheral. In a simple way, any user with a computer can connect a device, regardless of its functionality, to the USB port of his computer and it will recognize it so that it can communicate with the CPU and perform the function for which it has been developed.

For all this to happen, the USB protocol has an enormous complexity in terms of communication packets between the CPU and the device or the USB controller, in addition to needing a driver installed in the operating system so that it can interpret it. As simple as the hardware may seem (2 data cables), there are many elements in the USB communication that take place so that the device can be recognized.

In order to study the whole USB protocol and the interaction between the device and the CPU, in this project we have developed a firmware in C that communicates with different sensors and devices, using an AVR microcontroller. As well as a set of example drivers to show the operation and communication between the computer and the device.


## Objectives

The main objective of this project is to build a low-cost USB device, which can be used to become familiar with the complex protocol that handles this connection standard, among other aspects we can highlight the information packets used, *URBs*, and the steps in the communication between the USB device and the host.  As well as the development of drivers for USB devices, using the synchronous and asynchronous API of the Linux kernel.

In addition, it also aims to become familiar with the different devices and sensors, such as displays or temperature readers, which are used connected to the USB device developed in this project.


## Work plan

For the development of this project, several meetings have been held with the project managers and among the developers. In the initial part of the project, the operation of the V-USB library has been studied with different projects using other microchips, to see what use is made of the report-IDs and the different functions used in the code. The Wireshark software [@sw-wireshark] has been used to study the endpoints used and the URBs packets in the communication, to study the software configuration and the possibility of using, for example, interrupts.

For the development of the peripherals to be used together on the Digispark board, Guillermo has been in charge of studying and developing the code corresponding to the temperature sensor and Javier to the one corresponding to the 2x16 LCD display, which later an OLED LCD display has been used.

For the integration of the code, GitHub has been used.


## Organization of the report

For the realization of this report, it has been divided into several chapters. In the first chapter, the hardware used and some low-level aspects of the library used are explained as an introduction. In the next chapter, the hardware elements used, such as the circular LED ring or the OLED LCD screen, are explained. Then, the microcontroller finally used for the final prototype is explained in detail, since it does not have the limitations that we had with the ATTiny85, as it can have more lines of communication with it.

In the appendices, we have the contributions of each member of the project, as well as different instructions to analyze the USB URBs packets with the Wireshark software, or also to *flash* our own firmware, with indications on the specific pins to use on the board.