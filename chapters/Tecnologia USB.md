<!-- Leave a blank line before the title -->

# Tecnología USB

En este capítulo se detalla a modo de introducción el funcionamiento a bajo nivel del protocolo USB, los tipos de mensajes que se envían entre el host y dispositivo; y los diferentes extremos de comunicación con los que se cuentan (endpoints), así como el protocolo HID utilizado en el proyecto. También se explica un ejemplo de uso del driver para el Blinkstick utilizado en la asignatura Arquitectura interna de Linux y Android.



## Descriptores en USB

Completar



## Endpoints

Completar



## Protocolo HID

Completar



### Report-ID

Completar!

Los report-ID son mensajes que mediante un número hexadecimal, se asigna un uso y un tiempo de espera (en bytes) que debe esperar en caso de que llegue un mensaje con dicho ID. En el siguiente ejemplo del array de ReportIDs, se declaran 3 ReportIDs de uso indefinido (ya que se usan para funciones personalizadas). El primero es de 8 bytes, cuando llega un mensaje con el ID=1 es un mensaje de cambio de color; el segundo y tercer ID son mensajes que contienen 32 bytes y es texto que puede almacenar la EEPROM, se usan para el número de serie o el nombre del dispositivo. [@usb-reports]

![Array de ReportIDs utilizados en nuestro proyecto](img/array_reportids.png){width=80%}



## Drivers USB en Linux

Completar



### USB core y su arquitectura

Completar



### URBs

Completar



### API síncrona y asíncrona

Completar



### Blinkstick

Completar

