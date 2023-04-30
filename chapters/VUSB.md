<!-- Leave a blank line before the title -->

# Proyecto V-USB

En este capítulo se describe el funcionamiento a bajo nivel de esta librería utilizada para el desarrollo del firmware, así como otras alternativas contempladas.




## Descripción

COMPLETAR

La librería V-USB [@objective-development] se usa para la implementación de software para dispositivos USB de baja velocidad que utilizan microcontroladores AVR (como los ATTiny o Atmel). Permite que estos microcontroladores actúen como un dispositivo USB, permitiendo la comunicación con el driver del host u otros dispositivos utilizando este protocolo. Más adelante se detallan algunos aspectos de esta comunicación, como el uso que hace esta librería de los report-id o los endpoints.

La gran ventaja de V-USB con respecto a otras librerías es el pequeño tamaño y bajo consumo que hace de los recursos disponibles. Para su funcionamiento, se requiere unos de pocos kilobytes de código firmware y una pequeña cantidad de memoria RAM para funcionar. Esto es especialmente ventajoso para dispositivos como el ATTiny85. Además, esta librería no necesita de ningún componente hardware adicional, sólamente un chip con interfaz USB, reduciendo notablemente su costo y diseño del circuito. 

Para la comunicación USB, esta librería hace uso de dos endpoints en el chip ATTiny85: el endpoint 0 y endpoint 1. El endpoint 0 es usado para las transferencias de control, es decir, inicializar y configurar la interfaz USB. El endpoint 1, se usa para la transferencia de datos, es decir, para enviar y recibir los informes USB HID. [@vusb-library]

Para la configuración de los endpoints, están definidos en el archivo cabecera `usbconfig.h`. En el caso del ATTiny85, dicha configuración se puede encontrar en el directorio `usbdrv`.




## Alternativas a esta librería

Completar
