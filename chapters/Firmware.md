<!-- Leave a blank line before the title -->

# Firmware desarrollado

En este capítulo se describe el funcionamiento del firmware desarrollado que da soporte a los distintos periféricos mencionados en el capítulo anterior.




## Perfiles de hardware

COMPLETAR





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
