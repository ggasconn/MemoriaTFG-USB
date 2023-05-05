<!-- Leave a blank line before the title -->

# Proyecto V-USB

En este capítulo se describe el funcionamiento a bajo nivel de esta librería utilizada para el desarrollo del firmware, así como otras alternativas contempladas.




## Descripción

El proyecto V-USB [@objective-development] se utiliza para implementar firmware en dispositivos USB de baja velocidad con pocos recursos, entre ellos se encuentran los microcontroladores AVR (como los ATTiny o Atmel, utilizados en este proyecto para realizar las pruebas y desarrollar el firmware final). Al ser una librería que no requiere de mucho tamaño para almacenar su código, ni del funcionamiento de muchos recursos del chip, hace que sea compatible sobre dispositivos con prestaciones hardware limitadas y con arquitecturas más bien sencillas.

Esta librería ha sido probada en los chips ATTiny85 y posteriormente en ATmega328p, las ventajas han sido evidentes en el uso de un firmware basado en V-USB para estos dispositivos. El requisito más importante que se tiene en ambos chips, es el número de periféricos que se desean conectar a ellos, ya que dependiendo del alcance del proyecto, ésto puede llegar a ser una limitación. En ATmega, como se explica más adelante, esta limitación se reduce notablemente al disponer de pines suficientes para el alcance de este proyecto. 

El proyecto V-USB utiliza la interfaz USB para la comunicación entre el host (con un driver cargado que lo controle) y el dispositivo conectado, por lo que utiliza todos los elementos de la comunicación USB (endpoints, mensajes URBs, etc) que se detallarán más adelante. También hablaremos de la estructura interna del proyecto, ya que se hace uso de Report-IDs, característico de V-USB.

En cuanto a la comunicación USB, esta librería hace uso de dos endpoints en el chip ATTiny85: el endpoint 0 y endpoint 1. El endpoint 0 es usado para las transferencias de control, es decir, inicializar y configurar la interfaz USB. El endpoint 1, se usa para la transferencia de datos, es decir, para enviar y recibir los informes USB HID. [@vusb-library]

Para la configuración de los endpoints, están definidos en el archivo cabecera `usbconfig.h`. En el caso del ATTiny85, dicha configuración se puede encontrar en el directorio `usbdrv`.




## Alternativas a esta librería

Completar
