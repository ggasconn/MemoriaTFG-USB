<!-- Leave a blank line before the title -->

# Hardware y firmware utilizado con USB

En este capítulo se detalla el uso de la librería utilizada para el desarrollo del firmware, que está basado en V-USB. También se explican los detalles hardware del microchip utilizado para ejecutar el firmware, y la placa embebida que utiliza la interfaz USB para la comunicación con el driver de nuestro sistema operativo.



## Microcontroladores AVR

La familia de microcontroladores AVR provienen del fabricante Atmel, diseñados para el desarrollo de sistemas empotrados y la ejecución de aplicaciones relacionados con robótica y una gran variedad de usos dentro del sector industrial.

Esta familia de microcontroladores AVR están caracterizados por su bajo consumo energético, además de tener una alta velocidad de procesamiento y una interfaz de programación relativamente fácil de usar. Cuentan con una arquitectura RISC, permitiendo un procesamiento más rápido y un uso eficiente de los recursos. [@avr-products]

Los microcontroladores AVR suelen tener una memoria flash para el almacenamiento de programas, SRAM para el almacenamiento de datos y una gran variedad de periféricos como temporizadores, UART y ADC para interactuar con dispositivos externos. Muchos de estos microcontroladores son usados en Arduino para el desarrollo de proyectos basados en la interfaz USB. [@avr-info]




## Microcontrolador ATTiny85

Este es el microcontrolador usado en el desarrolo del firmware USB para el control de un anillo circular de leds junto a una pantalla LCD OLED y un sensor de temperatura. 

Como se ha descrito previamente, este microcontrolador forma parte de la familia de AVR. Es un microchip de 8 bits de bajo consumo y alto rendimiento, cuenta con 8 kilobytes de memoria flash, 512 bytes de EEPROM y 512 bytes de SRAM. Tiene 6 pines de entrada/salida de propósito general (GPIOs), que se pueden usar para distintos propósitos, como entrada o salida digital, PWM o entrada analógica. También tiene un oscilador interno con frecuencias ajustables, eliminando la necesidad de un cristal o resonador externo. [@avr-attiny85] [@attiny85-datasheet]

![Chip ATTiny85](img/ATTiny85.jpg){width=30% #fig:label1}

El ATTiny85 es un microcontrolador versátil y se puede usar en una gran variedad de aplicaciones, como sistemas de control, monitoreo de sensores y dispositivos que funcionan con baterías. Uno de los usos más populares de ATTiny85 (y es en el que se ha trabajado en este proyecto) es el desarrollo de proyectos pequeños y de bajo costo.

Para facilitar el desarrolo de estos proyectos, se ha añadido compatibilidad con este microcontrolador al entorno Arduino, lo que facilita en gran manera el programar e interactuar con dispositivos externos.

![Integración del entorno Arduino con ATTiny85](img/attiny85_arduino.png){width=55% #fig:label2}

En cuanto a los componentes hardware que integra, contamos con dos temporizadores 8 bits con PWM, un convertidor analógido-digital (ADC) de 8 canales, y una interfaz serie que se puede configurar para usarse como SPI, I2C o UART. Para la pantalla OLED, se ha utilizado el protocolo I2C con sus correspondientes librerías en C, que posteriormente se detallará.

Un factor a destacar de este microchip es el bajo consumo que requiere para funcionar, por lo que para futuras versiones del proyecto se puede integrar una batería para demostrar que no necesita de una cantidad muy grande de energía y puede aprovechar al máximo la duración de la batería. [@attiny85-desc]

A continuación, se detalla el esquema de la configuración de cada uno de los pines para el chip.

![Detalle de los pines del ATTiny85](img/attiny85_pines.png){width=60% #fig:label3}




## Librería V-USB

La librería V-USB [@objective-development] se usa para la implementación de software para dispositivos USB de baja velocidad que utilizan microcontroladores AVR (como los ATTiny o Atmel). Permite que estos microcontroladores actúen como un dispositivo USB, permitiendo la comunicación con el driver del host u otros dispositivos utilizando este protocolo. Más adelante se detallan algunos aspectos de esta comunicación, como el uso que hace esta librería de los report-id o los endpoints.

La gran ventaja de V-USB con respecto a otras librerías es el pequeño tamaño y bajo consumo que hace de los recursos disponibles. Para su funcionamiento, se requiere unos de pocos kilobytes de código firmware y una pequeña cantidad de memoria RAM para funcionar. Esto es especialmente ventajoso para dispositivos como el ATTiny85. Además, esta librería no necesita de ningún componente hardware adicional, sólamente un chip con interfaz USB, reduciendo notablemente su costo y diseño del circuito. 

Para la comunicación USB, esta librería hace uso de dos endpoints en el chip ATTiny85: el endpoint 0 y endpoint 1. El endpoint 0 es usado para las transferencias de control, es decir, inicializar y configurar la interfaz USB. El endpoint 1, se usa para la transferencia de datos, es decir, para enviar y recibir los informes USB HID. [@vusb-library]

Para la configuración de los endpoints, están definidos en el archivo cabecera `usbconfig.h`. En el caso del ATTiny85, dicha configuración se puede encontrar en el directorio `usbdrv`. A continuación, se destacan los puntos más importantes de la configuración software utilizada en la librería.

En el siguiente fragmento de código, se encuentra la configuración definida para el puerto de entrada/salida usado en la configuración USB (para el ATTiny85, se utiliza el puerto B), junto con los bits usados para las señales D+ y D-. También se establece en la última línea la frecuencia a la que trabajará el dispositivo USB.

```C
#define USB_CFG_IOPORTNAME      B
#define USB_CFG_DMINUS_BIT     3
#define USB_CFG_DPLUS_BIT       4
#define USB_CFG_CLOCK_KHZ       (F_CPU/1000)
```
La configuración establecida de puertos y bits para las señales D+ y D- usados en el ATmega difiere, por lo que quedaría de la siguiente forma:

```C
#define USB_CFG_IOPORTNAME      D
/* This is the port where the USB bus is connected (we use for ATmega configuration). When you configure it to
 * "D", the registers PORTD, PIND and DDRD will be used.
 * When it is configured to "B", the registers PORTB, PINB and DDRB will be used (in this case for ATTiny).
 */
#define USB_CFG_DMINUS_BIT     4
/* This is the bit number in USB_CFG_IOPORT where the USB D- line is connected.
 * This may be any bit in the port.
 */
#define USB_CFG_DPLUS_BIT       2
/* This is the bit number in USB_CFG_IOPORT where the USB D+ line is connected.
 * This may be any bit in the port. Please note that D+ must also be connected
 * to interrupt pin INT0! [You can also use other interrupts, see section
 * "Optional MCU Description" below, or you can connect D- to the interrupt, as
 * it is required if you use the USB_COUNT_SOF feature. If you use D- for the
 * interrupt, the USB interrupt will also be triggered at Start-Of-Frame
 * markers every millisecond.]
```



A continuación se muestra la configuración para las interrupciones en los endpoints, en este caso al estar la variable a 1 se hace uso de interrupciones. En la macro `USB_CFG_INTR_POLL_INTERVAL` se indica el intervalo en milisegundos para las interrupciones por polling. La siguiente configuración es la que se ha utilizado para la placa Digispark con ATTiny:

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


Para el uso de la placa Nano con ATmega, utilizamos la siguiente configuración en el firmware (destacar la macro ```USB_CFG_HAVE_INTRIN_ENDPOINT``` donde se indica que se va a hacer uso de dos tipos de endpoints, de tipo control y interrupción:

```C
/* --------------------------- Functional Range ---------------------------- */

#define USB_CFG_HAVE_INTRIN_ENDPOINT    1
/* Define this to 1 if you want to compile a version with two endpoints: The
 * default control endpoint 0 and an interrupt-in endpoint (any other endpoint
 * number).
 */
#define USB_CFG_HAVE_INTRIN_ENDPOINT3   0
/* Define this to 1 if you want to compile a version with three endpoints: The
 * default control endpoint 0, an interrupt-in endpoint 3 (or the number
 * configured below) and a catch-all default interrupt-in endpoint as above.
 * You must also define USB_CFG_HAVE_INTRIN_ENDPOINT to 1 for this feature.
 */
#define USB_CFG_EP3_NUMBER              3
/* If the so-called endpoint 3 is used, it can now be configured to any other
 * endpoint number (except 0) with this macro. Default if undefined is 3.
 */
#define USB_CFG_IMPLEMENT_HALT          0
/* Define this to 1 if you also want to implement the ENDPOINT_HALT feature
 * for endpoint 1 (interrupt endpoint). Although you may not need this feature,
 * it is required by the standard. We have made it a config option because it
 * bloats the code considerably.
 */
#define USB_CFG_SUPPRESS_INTR_CODE      0
/* Define this to 1 if you want to declare interrupt-in endpoints, but don't
 * want to send any data over them. If this macro is defined to 1, functions
 * usbSetInterrupt() and usbSetInterrupt3() are omitted. This is useful if
 * you need the interrupt-in endpoints in order to comply to an interface
 * (e.g. HID), but never want to send any data. This option saves a couple
 * of bytes in flash memory and the transmit buffers in RAM.
 */
#define USB_CFG_INTR_POLL_INTERVAL      10
/* If you compile a version with endpoint 1 (interrupt-in), this is the poll
 * interval. The value is in milliseconds and must not be less than 10 ms for
 * low speed devices.
 */
#define USB_CFG_IS_SELF_POWERED         0
/* Define this to 1 if the device has its own power supply. Set it to 0 if the
 * device is powered from the USB bus.
 */
#define USB_CFG_MAX_BUS_POWER           100
/* Set this variable to the maximum USB bus power consumption of your device.
 * The value is in milliamperes. [It will be divided by two since USB
 * communicates power requirements in units of 2 mA.]
 */
```
```C
#define USB_CFG_IMPLEMENT_FN_WRITE      1
/* Set this to 1 if you want usbFunctionWrite() to be called for control-out
 * transfers. Set it to 0 if you don't need it and want to save a couple of
 * bytes.
 */
#define USB_CFG_IMPLEMENT_FN_READ       1
/* Set this to 1 if you need to send control replies which are generated
 * "on the fly" when usbFunctionRead() is called. If you only want to send
 * data from a static buffer, set it to 0 and return the data from
 * usbFunctionSetup(). This saves a couple of bytes.
 */
#define USB_CFG_IMPLEMENT_FN_WRITEOUT   0
/* Define this to 1 if you want to use interrupt-out (or bulk out) endpoints.
 * You must implement the function usbFunctionWriteOut() which receives all
 * interrupt/bulk data sent to any endpoint other than 0. The endpoint number
 * can be found in 'usbRxToken'.
 */
#define USB_CFG_HAVE_FLOWCONTROL        0
/* Define this to 1 if you want flowcontrol over USB data. See the definition
 * of the macros usbDisableAllRequests() and usbEnableAllRequests() in
 * usbdrv.h.
 */
#define USB_CFG_LONG_TRANSFERS          0
/* Define this to 1 if you#define USB_CFG_HAVE_INTRIN_ENDPOINT    1
/* Define this to 1 if you want to compile a version with two endpoints: The
 * default control endpoint 0 and an interrupt-in endpoint (any other endpoint
 * number).
 */
#define USB_CFG_HAVE_INTRIN_ENDPOINT3   0
/* Define this to 1 if you want to compile a version with three endpoints: The
 * default control endpoint 0, an interrupt-in endpoint 3 (or the number
 * configured below) and a catch-all default interrupt-in endpoint as above.
 * You must also define USB_CFG_HAVE_INTRIN_ENDPOINT to 1 for this feature.
 */
#define USB_CFG_EP3_NUMBER              3
/* If the so-called endpoint 3 is used, it can now be configured to any other
 * endpoint number (except 0) with this macro. Default if undefined is 3.
 */
#define USB_CFG_IMPLEMENT_HALT          0
/* Define this to 1 if you also want to implement the ENDPOINT_HALT feature
 * for endpoint 1 (interrupt endpoint). Although you may not need this feature,
 * it is required by the standard. We have made it a config option because it
 * bloats the code considerably.
 */
#define USB_CFG_SUPPRESS_INTR_CODE      0
/* Define this to 1 if you want to declare interrupt-in endpoints, but don't
 * want to send any data over them. If this macro is defined to 1, functions
 * usbSetInterrupt() and usbSetInterrupt3() are omitted. This is useful if
 * you need the interrupt-in endpoints in order to comply to an interface
 * (e.g. HID), but never want to send any data. This option saves a couple
 * of bytes in flash memory and the transmit buffers in RAM.
 */
#define USB_CFG_INTR_POLL_INTERVAL      10
/* If you compile a version with endpoint 1 (interrupt-in), this is the poll
 * interval. The value is in milliseconds and must not be less than 10 ms for
 * low speed devices.
 */
#define USB_CFG_IS_SELF_POWERED         0
/* Define this to 1 if the device has its own power supply. Set it to 0 if the
 * device is powered from the USB bus.
 */
#define USB_CFG_MAX_BUS_POWER           100
/* Set this variable to the maximum USB bus power consumption of your device.
 * The value is in milliamperes. [It will be divided by two since USB
 * communicates power requirements in units of 2 mA.]
 */
#define USB_CFG_IMPLEMENT_FN_WRITE      1
/* Set this to 1 if you want usbFunctionWrite() to be called for control-out
 * transfers. Set it to 0 if you don't need it and want to save a couple of
 * bytes.
 */
#define USB_CFG_IMPLEMENT_FN_READ       1
/* Set this to 1 if you need to send control replies which are generated
 * "on the fly" when usbFunctionRead() is called. If you only want to send
 * data from a static buffer, set it to 0 and return the data from
 * usbFunctionSetup(). This saves a couple of bytes.
 */
#define USB_CFG_IMPLEMENT_FN_WRITEOUT   0
/* Define this to 1 if you want to use interrupt-out (or bulk out) endpoints.
 * You must implement the function usbFunctionWriteOut() which receives all
 * interrupt/bulk data sent to any endpoint other than 0. The endpoint number
 * can be found in 'usbRxToken'.
 */
#define USB_CFG_HAVE_FLOWCONTROL        0
/* Define this to 1 if you want flowcontrol over USB data. See the definition
 * of the macros usbDisableAllRequests() and usbEnableAllRequests() in
 * usbdrv.h.
*/

#define USB_COUNT_SOF                   0
/* define this macro to 1 if you need the global variable "usbSofCount" which
 * counts SOF packets. This feature requires that the hardware interrupt is
 * connected to D- instead of D+.
 */

#define USB_CFG_CHECK_DATA_TOGGLING     0
/* define this macro to 1 if you want to filter out duplicate data packets
 * sent by the host. Duplicates occur only as a consequence of communication
 * errors, when the host does not receive an ACK. Please note that you need to
 * implement the filtering yourself in usbFunctionWriteOut() and
 * usbFunctionWrite(). Use the global usbCurrentDataToken and a static variable
 * for each control- and out-endpoint to check for duplicate packets.
 */
#define USB_CFG_HAVE_MEASURE_FRAME_LENGTH   1
/* define this macro to 1 if you want the function usbMeasureFrameLength()
 * compiled in. This function can be used to calibrate the AVR's RC oscillator.
 */
```



En el siguiente código se muestra la configuración para establecer diversos parámetros como el Vendor-ID o el Device-ID, a parte de los distintos nombres para el dispositivo, o el número de serie.

```C
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



### ReportIDs en V-USB
Para dotar de funcionalidad al firmware, éste se comunica mediante mensajes que son nombrados mediante un número hexadecimal, al que se les asigna un uso y un tiempo de espera (en bytes) que debe esperar en caso de que llegue un mensaje con dicho ID. En el siguiente ejemplo del array de ReportIDs, se declaran 3 ReportIDs de uso indefinido (ya que se usan para funciones personalizadas). El primero es de 8 bytes, cuando llega un mensaje con el ID=1 es un mensaje de cambio de color; el segundo y tercer ID son mensajes que contienen 32 bytes y es texto que puede almacenar la EEPROM, se usan para el número de serie o el nombre del dispositivo. [@usb-reports]

![Array de ReportIDs utilizados en nuestro proyecto](img/array_reportids.png){width=80%}




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

Es una placa realmente compacta, con un tamaño de 25 mm x 18 mm. Incluye una interfaz USB para programación y comunicación con el host, así como un regulador de voltaje y LED de encendido. [@digispark-board]

![Placa de desarrollo Digispark](img/digispark_board.jpg){width=35% #fig:label2}

Una característica única de la placa Digispark es que viene preprogramada con un cargador de arranque que permite programarla a través de USB usando el IDE de Arduino (llamado Micronucleous Bootloader). Con ello se puede programar código desde Arduino fácilmente a la placa sin necesidad de un programador independiente. Aunque para el desarrollo de nuestro proyecto, se ha optado por utilizar la librería V-USB sin el entorno de Arduino, por lo que se requiere de un compilador de AVR y un programador externo. [@setup-digispark]




## Placa NANO con ATmega y puerto serie

Arduino Nano se utiliza para desarrollar proyectos en los que se requiere un tamaño reducido, bajo consumo de energía y una amplia gama de opciones de E/S. Está basada en el microcontrolador ATmega328p, en el que destaca su asequibilidad y facilidad de uso.

El chip ATmega328p es un microcontrolador AVR de 8 bits que funciona hasta a 16 MHz con 32 KB de memoria flash, 2 KB de SRAM y 1 KB de EEPROM. También incluye diversos periféricos, como convertidores analógico-digitales, salidas de modulación por ancho de pulsos (PWM) e interfaces de comunicación serie.

![Pinout de la placa NANO con ATmega328p](img/nanopinout.png){width=50%}



La placa Nano tiene un formato compacto de tamaño 45 mm x 18 mm), lo que resulta idóneo para su uso en proyectos pequeños que requieren portabilidad y ahorro de espacio. A pesar de su reducido tamaño, cuenta con una gran variedad de opciones de E/S, incluidos 14 pines de entrada/salida digital, 8 pines de entrada analógica y 6 pines PWM. Cuenta también con un puerto USB para programación y alimentación, y un botón de reset para reiniciar la placa. [@nano-info]

Una de las características más atractivas de la placa Nano es su compatibilidad con Arduino IDE, aunque nosotros desarrollamos y cargamos el firmware directamente sin utilizar este entorno, como se explica en otro apartados. 

La placa Nano se utiliza, en general, para una amplia gama de proyectos, incluyendo robótica, automatización, registro de datos y aplicaciones de sensores. También es una opción para la creación de prototipos y pruebas, ya que se puede integrar fácilmente en circuitos breadboard o perfboard. En este proyecto resulta realmente útil, comparando con Digispark, ya que el número de pines de entrada y salida es notablemente mayor que la anterior, lo que resulta útil para realizar pruebas con varios periféricos conectados a la vez.

