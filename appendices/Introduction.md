<!-- Leave a blank line before the title -->



# Introduction

This introductory chapter presents the motivation for this project, and describes the objectives and the planning of the tasks to carry it out.


## Motivation

The USB protocol is widely used throughout the world for interaction with peripherals of a very diverse nature. Any user can connect a device to his computer. When doing so, the corresponding *driver* installed on the system recognizes the device and allows it to perform the functions for which it has been developed. To achieve this simplicity of use, the developer of the *driver* and the manufacturer of the USB device have to deal with the complexity of the USB protocol. 

In current Computer Science degrees, the limited time that can be spent on describing low-level input-output aspects coupled with the complexity of the USB technology and specification makes it difficult for students to acquire knowledge about USB device design and driver development. In order to develop practical content on these aspects, it is also necessary to have sufficiently simple, versatile and low-cost USB hardware, which can be a significant challenge.

In the Faculty of Computer Science of the Complutense University of Madrid, one of the few subjects that deals with the development of USB *drivers* is the subject *Internal Architecture of Linux and Android*, a common elective offered to students of different degree courses. In this report we will refer to this subject as "LIN", since this is the short name used administratively in the Faculty of Computer Science to refer to it. In the LIN lab practices we make use of the Blinkstick Strip device [@blinkstick-strip], a simple USB device with 8 colored LEDs whose state can be altered individually. The simplicity of this device allows to elaborate practices to become familiar with the development of USB *drivers*, especially those implemented at the operating system kernel level.

Blinkstick Strip, however, has several limitations. First, this device is so simple that it prevents you from exploring the potential of USB in depth. After all, it only allows altering the state of individual LEDs by sending simple control messages to the device. Consequently, the corresponding practices do not allow experimentation with other modes of data transfer covered by the USB specification, and thus use a very small subset of the API calls for USB *driver* development. Secondly, their limited versatility does not justify their price (around 20€ per unit) to enable a large-scale adoption of the device in different subjects and undergraduate degrees at the faculty. Finally, although the Blinkstick Strip firmware is freely available, its manufacturer does not provide documentation on how to modify it using the commercial hardware platform. This is a barrier to using the device for *firmware* design practices, where *bare metal* programming is required. 

Regardless of the input-output protocol or technology used, having a sufficiently flexible hardware platform is a key issue in laboratory practices focused on interaction with peripherals. In particular, the ability to add new input-output devices to the system without requiring a lot of additional cabling allows a large number of lab practices to be developed to meet the needs of different subjects. This also makes it possible to introduce small variations in the practical content of the same subject over time. These factors make it possible to extend the life cycle of a teaching-oriented hardware platform, thereby substantially reducing the cost of laboratory equipment. 

The Bee board [@bee-repo;@bee-board] developed by Prof. Christian Tenllado is a clear example of a system designed for the construction of flexible hardware platforms. It is an expansion board for systems based on the *pinout* of the Raspberry Pi mini PC (v1-v4) that is provided with a set of simple peripheral devices, and allows the expansion of these by means of generic bias circuits. This board, used in its different versions in undergraduate courses at the Faculties of Computer Science and Physical Sciences at UCM, despite not providing support for USB, has been a great inspiration for the realization of this TFG.


## Objectives

The main objective of this project has been to build a low-cost hardware-software infrastructure that allows prototyping USB devices and developing drivers for these devices.  Our infrastructure is based on AVR microcontrollers from Atmel [@avr-atmel], and on the V-USB project [@v-usb], which provides a generic *firmware* to implement software USB support. Our project extends V-USB to facilitate the creation of USB *firmware* that manages multiple peripherals simultaneously. Although there are other alternatives to using the V-USB framework (e.g., the use of microcontrollers with hardware USB support), the proposed infrastructure is not only low-cost, but also offers great versatility for low-level software development. In particular, it allows experimenting with both the development of USB *drivers* in systems provided with operating system, as well as the implementation of *USB firmware* for hardware management, using *bare metal* programming.  In this TFG, development has been carried out in both areas.  

Another objective of the project is to provide a set of USB *drivers* that illustrate the interaction with devices of different nature, such as RGB LED arrays, LCD displays, 7-segment displays, buzzers or temperature sensors. For this purpose the proposed infrastructure is accompanied by a collection of drivers that employ a wide collection of functions of the API provided by the Linux kernel for the development of USB drivers. These *drivers* are implemented as loadable modules of the Linux kernel, in order to serve as a basis for future practices of the LIN subject. Although the development of user-space USB *drivers* with *libusb* [@libusb] is outside the scope of this TFG, the kernel drivers provided serve as a basis for the creation of user-space drivers developed with this library. 

Finally, given the flexibility provided by the Bee board [@bee-repo;@bee-board], this project has explored ways to use this board as a basis for implementing USB devices. Note that the Faculty of Computer Science has already acquired a good number of these boards for different subjects, so the reuse of this hardware as a basic block of the USB infrastructure allows further amortization of the investment made.


## Work plan

Two hardware prototypes of the infrastructure have been developed in this project. The first prototype is based on the ATTiny85 microcontroller--same chip used by the Blinkstick Strip device. The second prototype overcomes the limitations of the previous one (described in detail in chapter \ref{cap:hw}), and uses the ATMega328p microcontroller. During the project it has been necessary to become familiar with programming on the two aforementioned microcontrollers--embedded in the Digispark [@digisparkboard-image] and Nano [@atmegaboard-image] boards, respectively--as well as with a large collection of input-output devices used in the prototypes. These devices included a circular ring of RGB LEDs, a buzzer (or *buzzer* ), an OLED display, and a 7-segment display (integrated on the Bee board [@bee-repo]).

With this in mind, a work plan was carried out, consisting of the following tasks (see Fig \ref{fig:gantt}):

![Project Gantt Chart](img/gantt.png){width=90% #fig:gantt}


T1

: Study of the Digispark board and analysis of the control of the different peripherals under Arduino environment.

T2

: Study of the *Blinkstick Strip* device and its open source firmware.

T3

: In-depth analysis of the *firmware* of the V-USB project.

T4

: Development of the first version of the V-USB firmware on the Digispark board (without full peripherals support)

T5

: Study of the I/O functions using the ATTiny85 microcontroller registers.

T6

: Extension of the functionality of the V-USB firmware developed in T4 with basic device control support. 

T7

: Development of register level control (without using Arduino environment) of the LED diode of the Digispark board. 

T8

: Development of the register level control of the circular LED ring.

T9

: Development of the register level control of the *buzzer*.

T10

: Development of the register level control of the 7-segment display.

T11

: Study of design alternatives for extra circuitry associated with the micro USB connector required by the second prototype.

T12

: Design of the hardware prototype based on ATMega328p

T13

: Integration of Nano board (with ATMega328p) with micro USB connector circuitry   

T14

: Firmware modification to work with ATMega328p chip

T15

: Adaptation of debugging tool to our prototype

T16

: Development of register level control of the OLED screen 

T17

: Study of the timers included in the ATMega328p

T18

: Study of PWM signals and their generation through timers

T19

: Refactoring the V-USB *firmware* for the use of profiles.

T20

: Implementation of hardware profiles using macros

T21

: Migration of LED driver implementation to PWM

T22

: Migration of buzzer driver implementation to PWM

T23

: Assembly of the final prototype (based on ATMega328p) on punched plate

T24 

: Writing of the TFG report


During the development of the project, several meetings have been held with the TFG directors. The frequency of these meetings has been gradually increasing as the development of the project has progressed. These meetings have served to solve design problems, raise the different phases and solve multiple doubts and limitations that have arisen throughout the project.

For the management of both the code and the sources of the project memory, shared repositories have been used in the GitHub platform. The links to both repositories are:

* **Source code**. https://github.com/ggasconn/TFG_USB

* **Memory**. https://github.com/ggasconn/MemoriaTFG-USB/


## Memory organization

The remainder of this memory has been divided into the following chapters:

- **Chapter 2** describes the low-level features of the USB protocol, as well as its main abstractions, such as descriptors, *endpoints*, or URBs.

- **Chapter 3** provides a general introduction to the V-USB project and discusses the source code structure of the *firmware* provided. This chapter also discusses possible alternatives to the use of V-USB for building hardware software infrastructures such as the one proposed in this project.

- **Chapter 4** presents the characteristics of the various microcontrollers used in the project, the technical details of the boards used in various phases of the project, and the hardware aspects of the peripherals supported by our prototype.

- **Chapter 5** analyzes in detail all the elements and functionalities implemented in the *firmware* of the prototype, which constitutes an extension of the V-USB project.

- **Chapter 6** describes in detail the example USB drivers that manage the peripheral devices supported in the firmware.

- **Chapter 7** contains the main conclusions of the dissertation and lists the lines of future work.


The following five appendices can also be found at the end of this report:

* **Appendix A** lists the contributions made by each member of this Final Degree Project to the project. 

* **Appendix B** describes the instructions for analyzing USB URB packets with the Wireshark software. 

* **Appendix C** provides a guide detailing the process for *flashing* the firmware on the prototype board. 

* Finally, **Appendices D and E** contain the English translation of this introductory chapter as well as the conclusions.