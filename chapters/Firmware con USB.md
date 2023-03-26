<!-- Leave a blank line before the title -->

# Introducción al hardware y firmware utilizado con USB

En este capítulo se detalla el uso de la librería utilizada para el desarrollo del firmware, que está basado en V-USB. También se explican los detalles hardware del microchip utilizado para ejecutar el firmware, y la placa embebida que utiliza la interfaz USB para la comunicación con el driver de nuestro sistema operativo.



## Microcontroladores AVR

La familia de microcontroladores AVR provienen del fabricante Atmel, diseñados para el desarrollo de sistemas empotrados y la ejecución de aplicaciones relacionados con robótica y una gran variedad de usos dentro del sector industrial.

Esta familia de microcontroladores AVR están caracterizados por su bajo consumo energético, además de tener una alta velocidad de procesamiento y una interfaz de programación relativamente fácil de usar. Cuentan con una arquitectura RISC, permitiendo un procesamiento más rápido y un uso eficiente de los recursos.

Los microcontroladores AVR suelen tener una memoria flash para el almacenamiento de programas, SRAM para el almacenamiento de datos y una gran variedad de periféricos como temporizadores, UART y ADC para interactuar con dispositivos externos. Muchos de estos microcontroladores son usados en Arduino para el desarrollo de proyectos basados en la interfaz USB.




## Microcontrolador ATTiny85

Este es el microcontrolador usado en el desarrolo del firmware USB para el control de un anillo circular de leds junto a una pantalla LCD OLED y un sensor de temperatura. 

Como se ha descrito previamente, este microcontrolador forma parte de la familia de AVR. Es un microchip de 8 bits de bajo consumo y alto rendimiento, cuenta con 8 kilobytes de memoria flash, 512 bytes de EEPROM y 512 bytes de SRAM. Tiene 6 pines de entrada/salida de propósito general (GPIOs), que se pueden usar para distintos propósitos, como entrada o salida digital, PWM o entrada analógica. También tiene un oscilador interno con frecuencias ajustables, eliminando la necesidad de un cristal o resonador externo.

![Chip ATTiny85](img/ATTiny85.jpg){width=30% #fig:label1}

El ATTiny85 es un microcontrolador versátil y se puede usar en una gran variedad de aplicaciones, como sistemas de control, monitoreo de sensores y dispositivos que funcionan con baterías. Uno de los usos más populares de ATTiny85 (y es en el que se ha trabajado en este proyecto) es el desarrollo de proyectos pequeños y de bajo costo.

Para facilitar el desarrolo de estos proyectos, se ha añadido compatibilidad con este microcontrolador al entorno Arduino, lo que facilita en gran manera el programar e interactuar con dispositivos externos.

![Integración del entorno Arduino con ATTiny85](img/attiny85_arduino.png){width=55% #fig:label2}

En cuanto a los componentes hardware que integra, contamos con dos temporizadores 8 bits con PWM, un convertidor analógido-digital (ADC) de 8 canales, y una interfaz serie que se puede configurar para usarse como SPI, I2C o UART. Para la pantalla OLED, se ha utilizado el protocolo I2C con sus correspondientes librerías en C, que posteriormente se detallará.

Un factor a destacar de este microchip es el bajo consumo que requiere para funcionar, por lo que para futuras versiones del proyecto se puede integrar una batería para demostrar que no necesita de una cantidad muy grande de energía y puede aprovechar al máximo la duración de la batería.

A continuación, se detalla el esquema de la configuración de cada uno de los pines para el chip.

![Detalle de los pines del ATTiny85](img/attiny85_pines.png){width=60% #fig:label3}




## Librería V-USB

La librería V-USB [@objective-development] se usa para la implementación de software para dispositivos USB de baja velocidad que utilizan microcontroladores AVR (como los ATTiny o Atmel). Permite que estos microcontroladores actúen como un dispositivo USB, permitiendo la comunicación con el driver del host u otros dispositivos utilizando este protocolo. Más adelante se detallan algunos aspectos de esta comunicación, como el uso que hace esta librería de los report-id o los endpoints.

La gran ventaja de V-USB con respecto a otras librerías es el pequeño tamaño y bajo consumo que hace de los recursos disponibles. Para su funcionamiento, se requiere unos de pocos kilobytes de código firmware y una pequeña cantidad de memoria RAM para funcionar. Esto es especialmente ventajoso para dispositivos como el ATTiny85. Además, esta librería no necesita de ningún componente hardware adicional, sólamente un chip con interfaz USB, reduciendo notablemente su costo y diseño del circuito. 

Para la comunicación USB, esta librería hace uso de dos endpoints en el chip ATTiny85: el endpoint 0 y endpoint 1. El endpoint 0 es usado para las transferencias de control, es decir, inicializar y configurar la interfaz USB. El endpoint 1, se usa para la transferencia de datos, es decir, para enviar y recibir los informes USB HID.

Para la configuración de los endpoints, están definidos en el archivo cabecera `usbconfig.h`. En el caso del ATTiny85, dicha configuración se puede encontrar en el directorio `usbdrv`. A continuación, se destacan los puntos más importantes de la configuración software utilizada en la librería.

En el siguiente fragmento de código, se encuentra la configuración definida para el puerto de entrada/salida usado en la configuración USB (para el ATTiny85, se utiliza el puerto B), junto con los bits usados para las señales D+ y D-. También se establece en la última línea la frecuencia a la que trabajará el dispositivo USB.

```C
#define USB_CFG_IOPORTNAME      B
#define USB_CFG_DMINUS_BIT     3
#define USB_CFG_DPLUS_BIT       4
#define USB_CFG_CLOCK_KHZ       (F_CPU/1000)
```
A continuación se muestra la configuración para las interrupciones en los endpoints, en este caso al estar la variable a 1 se hace uso de interrupciones. En la macro `USB_CFG_INTR_POLL_INTERVAL` se indica el intervalo en milisegundos para las interrupciones por polling.

```C
#define USB_CFG_HAVE_INTRIN_ENDPOINT    1
#define USB_CFG_HAVE_INTRIN_ENDPOINT3   1
#define USB_CFG_EP3_NUMBER              3
#define USB_CFG_SUPPRESS_INTR_CODE      0
#define USB_CFG_INTR_POLL_INTERVAL      100
```

```C
#define USB_CFG_IS_SELF_POWERED         0
#define USB_CFG_MAX_BUS_POWER           40
#define USB_CFG_IMPLEMENT_FN_WRITE      1
#define USB_CFG_IMPLEMENT_FN_READ       1
#define USB_CFG_IMPLEMENT_FN_WRITEOUT   0
#define USB_CFG_HAVE_FLOWCONTROL        0
#define USB_CFG_LONG_TRANSFERS          0
#define USB_COUNT_SOF                   1
```
En el siguiente párrafo de código, se muestra la configuración definida para establecer diversos parámetros como el Vendor-ID o el Device-ID, a parte de los distintos nombres para el dispositivo, o el número de serie.

```C
#define USB_CFG_CHECK_DATA_TOGGLING     0
#define USB_CFG_HAVE_MEASURE_FRAME_LENGTH   1
#define USB_CFG_VENDOR_ID       0xA0, 0x20
#define USB_CFG_DEVICE_ID       0xe5, 0x41 
#define USB_CFG_DEVICE_VERSION  0x00, 0x02

#define USB_CFG_VENDOR_NAME     'x', 'G', 'G', 'C'
#define USB_CFG_VENDOR_NAME_LEN 4

#define USB_CFG_DEVICE_NAME     
'p', 'w', 'n', 'e', 'd', 'D', 'e', 'v', 'i', 'c', 'e'
#define USB_CFG_DEVICE_NAME_LEN 11

#define USB_CFG_SERIAL_NUMBER   
    'N', 'I', 'T', 'R', 'A', 'M', '0', '1', '-', '1', '.', '4'   
#define USB_CFG_SERIAL_NUMBER_LEN   12 
#define USB_CFG_DEVICE_CLASS        0
#define USB_CFG_DEVICE_SUBCLASS     0
#define USB_CFG_INTERFACE_CLASS     3
#define USB_CFG_INTERFACE_SUBCLASS  0
#define USB_CFG_INTERFACE_PROTOCOL  0
#define USB_CFG_HID_REPORT_DESCRIPTOR_LENGTH    58
```
Las siguientes líneas muestran la configuración relacionada con el manejo de interrupciones para el dispositivo USB. La macro `USB_INTR_CFG` nos indica la máscara de bits para el cambio de puerto de E/S para permitir interrupciones en el pin D+. Así, la macro `USB_INTR_CFG_SET` especifica el registro de desplazamiento para dicho cambio. 

```C
#ifndef SIG_INTERRUPT0
#define SIG_INTERRUPT0                 _VECTOR(1)
#endif
#define USB_INTR_CFG            PCMSK
#define USB_INTR_CFG_SET        (1<<USB_CFG_DPLUS_BIT)
#define USB_INTR_ENABLE_BIT     PCIE
#define USB_INTR_PENDING_BIT    PCIF
#define USB_INTR_VECTOR         SIG_PIN_CHANGE
```
La macro `USB_INTR_ENABLE_BIT` especifica el bit en el registro de control de interrupción de cambio de pin para habilitar las interrupciones de cambio de pin. Por otra parte, `USB_INTR_PENDING_BIT` especifica el bit que indica la presencia de una interrupción pendiente.


## Placa Digispark con interfaz USB

La placa Digispark cuenta con el chip ATTiny85, es la placa elegida para desarrollar el firmware de este proyecto. 

Los detalles técnicos con los que cuenta esta placa son los siguientes:

- Microcontroller: ATTiny85
- Operating Voltage: 5V
- Input Voltage (recommended): 7-12V
- Input Voltage (limits): 5-16V
- Digital I/O Pins: 6
- PWM Channels: 3
- Analog Input Channels: 4
- Flash Memory: 8 KB (of which 2.5 KB is used by the bootloader)
- SRAM: 512 bytes
- EEPROM: 512 bytes

Es una placa realmente compacta, con un tamaño de 25 mm x 18 mm. Incluye una interfaz USB para programación y comunicación con el host, así como un regulador de voltaje y LED de encendido. 

![Placa de desarrollo Digispark](img/digispark_board.jpg){width=55% #fig:label2}

Una característica única de la placa Digispark es que viene preprogramada con un cargador de arranque que permite programarla a través de USB usando el IDE de Arduino (llamado Micronucleous Bootloader). Con ello se puede programar código desde Arduino fácilmente a la placa sin necesidad de un programador independiente. Aunque para el desarrollo de nuestro proyecto, se ha optado por utilizar la librería V-USB sin el entorno de Arduino, por lo que se requiere de un compilador de AVR y un programador externo.