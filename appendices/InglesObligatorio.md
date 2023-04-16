<!-- Leave a blank line before the title -->

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