<!-- Leave a blank line before the title -->

# Tecnología USB

En este capítulo se detalla a modo de introducción el funcionamiento a bajo nivel del protocolo USB, los tipos de mensajes que se envían entre el host y dispositivo; y los diferentes extremos de comunicación con los que se cuentan (endpoints), así como el protocolo HID utilizado en el proyecto. También se explica un ejemplo de uso del driver para el Blinkstick utilizado en la asignatura Arquitectura interna de Linux y Android.


## Descriptores en USB

Todo dispositivo USB consta de una jerarquía de descriptores que indican mediante parámetros cómo es el dispositivo con el que se está comunicando, qué funcionalidades ofrece y muchos otros datos de interés para la correcta configuración del mismo por el sistema operativo en uso.

Dentro de esta jerarquía hay diferentes tipos de descriptores, cada uno de ellos se encarga de encapsular los datos correspondientes a la parte del dispositivo que describe, pero todos tienen en común que exponen un conjunto de valores concretos para unos parámetros concretos, por ejemplo, `bLenght: 54` indica que la longitud del descriptor es de 54 bytes.

Los descriptores más usados dentro de USB son:

- Device descriptors

- Configuration descriptors
- Interface descriptors
- Endpoint descriptors
- String descriptors

En los sistemas operativos que usan un kernel Linux, podemos ver los diferentes descriptores representados de forma jerárquica junto con sus valores a través del sistema de ficheros. El kernel nos expone esta información bajo la ruta `/sys/bus/usb/devices`, donde podemos encontrar todos los dispositivos USB junto con dicha información.

![Listado por consola de los dispositivos USB y ejemplo de descriptor](img/usbDescriptorExampleLinux.png)

### Device descriptors

Los descriptores de dispositivo son el tipo de descriptor más general, contiene información básica acerca del dispositivo pero a la vez muy útil. Este tipo de descriptor representa al dispositivo USB como una entidad completa, por lo que debido a su propia naturaleza, solo puede haber uno por dispositivo.

Algunos de los campos más importantes que contiene este descriptor son los siguientes:

| Parámetro       | Descripción del valor                                        |
| --------------- | ------------------------------------------------------------ |
| bLenght         | Contiene un número con el tamaño del descriptor.             |
| bDescriptorType | Indica el tipo de descriptor que es. En este caso, device.   |
| bcdUSB          | Indica el estandar USB con el que funciona el dispositivo.   |
| bMaxPacketSize  | Indica el tamaño máximo de paquete que soporta el dispositivo |
| idVendor        | Indica el VendorID asignado por la organización USB-IF.      |
| idProduct       | Es el identificador del producto dentro del VendorID.        |
| iSerialNumber   | Contiene el número de serie del dispositivo.                 |



### Configuration descriptors

Los descriptores de configuración contienen toda la información relativa al funcionamiento del dispositivo, desde la corriente máxima que consume, si es alimentado por el bus o se alimenta de manera externa, el número de interfaces que tiene y un largo etcetera. 

Los dispositivos USB pueden tener diferentes modos de trabajo, que se representan en diferentes descriptores de configuración, es por ello que un dispositivo puede tener múltiples descriptores de este tipo. Cuando el host comienza a interacturar con el dispositivo USB, recibe entre otras cosas estos descriptores y finalmente le manda una confirmación al dispositivo indicándole con cual de las configuraciones va a configurar el dispositivo.

Dentro del descriptor tenemos múltiples parámetros de entre los que podemos destacar:

| Parámetro           | Descripción del valor                                        |
| ------------------- | ------------------------------------------------------------ |
| bLenght             | Contiene un número con el tamaño del descriptor.             |
| bDescriptorType     | Indica el tipo de descriptor que es. En este caso, configuration. |
| bNumInterfaces      | Indica el número de interfaces del dispositivo.              |
| bConfigurationValue | Portará el valor de la configuración que elija el host.      |
| iConfiguration      | Indica el identificador de la configuración.                 |
| bMaxPower           | Indica la corriente máxima que puede consumir.               |



### Interface descriptors

Los descriptores de tipo interfaz podrían verse como una manera de agrupar los diferentes endpoints que son responsables de alguna funcionalidad del dispositivo.

Los dispositivos USB pueden tener múltiples interfaces, el límite máximo de interfaces es de 16 IN y 16 OUT.

De los diferentes parámetros del descriptor destacamos:

| Parámetro        | Descripción del valor                                        |
| ---------------- | ------------------------------------------------------------ |
| bLenght          | Contiene un número con el tamaño del descriptor.             |
| bDescriptorType  | Indica el tipo de descriptor que es. En este caso, interface. |
| bInterfaceNumber | Contiene el número de la interfaz                            |
| bNumEndpoints    | Indica el número de endpoints disponibles en la interfaz     |
| bInterfaceClass  | Indica el tipo de interfaz, es una clase asignada por USB-IF |

### Endpoint descriptors

Los descriptores de endpoints contienen información acerca del tipo de transferencia, la dirección de los datos, el tamaño máximo de paquete o el intervalo de polling.

Destacamos los siguientes parámetros:

| Parámetro        | Descripción del valor                                        |
| ---------------- | ------------------------------------------------------------ |
| bLenght          | Contiene un número con el tamaño del descriptor.             |
| bDescriptorType  | Indica el tipo de descriptor que es. En este caso, endpoint. |
| bEndpointAddress | Contiene la dirección del endpoint.                          |
| bmAttributes     | Lleva una máscara de bits que indican diferentes parámetros. |
| wMaxPacketSize   | Indica el tamaño máximo de paquete.                          |
| bInterval        | Intervalo de polling, expresado en frames o micro-frames.    |



### String descriptors

Estos descriptores son opcionales y contienen información legible en formato UNICODE. El resto de descriptores pueden hacer referencia a estos descriptores para ampliar algún tipo de información en formato legible por humanos.

Al estar codificado en UNICODE, estos descriptores soportan varios idiomas. Para ello, cuando se pide uno de estos descriptores, se hace con un descriptor que contiene los idiomas que se solicitan.

| Parámetro       | Descripción del valor                                      |
| --------------- | ---------------------------------------------------------- |
| bLenght         | Contiene un número con el tamaño del descriptor.           |
| bDescriptorType | Indica el tipo de descriptor que es. En este caso, string. |
| wLANGID[0]      | Código representando el idioma solicitado                  |
| ...             | ...                                                        |
| wLANGID[n]      | Código representando el idioma solicitado                  |

Una vez elegido el idioma, el texto viene en un descriptor como el siguiente:

| Parámetro       | Descripción del valor                                      |
| --------------- | ---------------------------------------------------------- |
| bLenght         | Contiene un número con el tamaño del descriptor.           |
| bDescriptorType | Indica el tipo de descriptor que es. En este caso, string. |
| bString         | Cadena con el contenido solicitado                         |




## Endpoints

Los endpoints dentro de un dispositivo USB podrían describirse como buffers de datos, los cuales pueden ser rellenados por el mismo dispositivo para enviar dichos datos al host o pueden ser llenados por el host para recibir datos en el dispositivo. De cualquier manera, los endpoints son una propiedad única del hardware USB en si mismo, independiente del sistema operativo o del host.

Los endpoints son unidireccionales, esto quiere decir que una vez se define la dirección del endpoint los datos solo pueden fluir en esa dirección. La dirección de los datos se define en el descriptor y puede ser:

- IN, siempre hace referencia a la dirección en la que los datos apuntan al host. Por ejemplo, un teclado USB envía datos al ordenador.
- OUT, hace referencia a la dirección que apunta del HOST al dispositivo. Por ejemplo, el ordenador envía datos a un dispositivo.

Existe una limitación en cuanto al número de endpoints que un dispositivo puede tener, como máximo 32 endpoints, 16 endpoints IN y 16 endpoints OUT.

Dentro de los endpoints podemos encontrar diferentes tipos definidos para diferentes usos, los más importantes son CONTROL, INTERRUPT y BULK. Cada uno de ellos está diseñado para cumplir unos requerimientos de velocidad, eficiencia y cantidad de datos enviados por lo que deben de usarse en su correcto contexto.

### Control Endpoints

Los endpoints de control se encargan de las transferencias de información relativa a la configuración del dispositivo. Son bidireccionales, es decir, pueden ser de tipo IN o de tipo OUT.

Aunque este tipo de endpoints no se suelen usar para transferencias de datos debido a sus limitaciones y uso restringido a configuración, pueden usarse para mandar algunos bytes de datos.

El endpoint 0 siempre está reservado para la configuración del dispositivo con el host y no tienen ningún descriptor asignado. La etapa de configuración podemos dividirla en tres pasos:

- SETUP, es un paquete que define el tipo de petición y el tamaño de los datos que se transmitirán en la etapa DATA.
- DATA, esta parte del proceso es opcional y si existe siempre empieza con un paquete DATA1. Después, si aún quedan datos por transferir se altera entre paquetes DATA1 y DATA0.
- STATUS, es un paquete de confirmación. Básicamente se envía en el sentido contrario un paquete DATA1 vacío.

### Interrupt Endpoints

Este tipo de transferencias se utiliza cuando la comunicación entre el dispositivo y el host es asíncrona. La manera de avisar que se quieren recibir datos es a través de un polling, cuyo intervalo de consulta es parametrizable por el usuario de 1ms  a 255ms para dispositivos low speed y de 125us a 4096ms para dispositivos high speed.

El sentido de la comunicación puede ser IN o OUT. Para comunicarse, se revisa el endpoint con el intervalo definido por el polling para ver si hay alguna interrupción pendiente de atender, si es así, se atiende. Esto es igual para ambas direcciones, cuando sea de tipo IN se enviarán los datos en un sentido y cuando sea de tipo OUT se realizará en sentido contrario.

### Bulk Endpoints

Los endpoints de tipo Bulk se usan para enviar grandes cantidades de información por el bus USB. Los escenarios en los que este tipo de endpoints son necesarios son muy variados, desde enviar un documento a una impresora para imprimir, hasta copiar un fichero de datos a un disco externo USB.

Para reservar ancho de banda, este tipo de transferencias usan espacio que otras transferencias no han usado una vez han terminado. Por ello, puede que las transferencias de este tipo se vuelvan algo lentas si no dispone de espacio en el bus.

Este tipo de transferencias solo son soportadas por dispositivos Full y High Speed y dependiendo del estandar que cumplan tienen un tamaño máximo de transferencia u otro. También cuentan con detección de errores CRC, por lo que permite asegurar la integridad de la información transmitida.

Al igual que los tipos anteriores, permite transferencias de tipo IN o OUT.


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

