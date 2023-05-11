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


## Clases USB

La organización usb.org [@usb-org] tiene definidas unas serie de códigos de clase que se utilizan para identificar la funcionalidad del dispositivo y ayudan al host a cargar un driver que sea compatible basado en esa funcionalidad.

La información se representa en tres bytes y contiene:

- Byte 1 - Clase base

- Byte 2 - Subclase

- Byte 3 - Protocolo

Como veíamos en el apartado anterior, esta información viaja al host a través de los descriptores y podemos encontrarla o bien en el descriptor de dispositivo, en el de interfaz o incluso en ambos.

Aunque en este proyecto solo se trabaja y se trata con la clase HID, *Human Interface Device*, hay algunas otras clases que también son muy conocidas y usadas como:

- ***Audio Class.*** Usada por dispositivos de audio.

- ***CDC Class.*** Usada por dispositivos de comunicación.

- ***Mass Storage Class.*** Usada por dispositivos de almacenamiento.

### Human Interface Device Class (HID)

La clase de dispositivos HID, codificada como `03h`, es principalmente usada por dispositivos que se comunican con el host y son controlados por humanos.

A través de este tipo de dispositivos, el host es capaz de interpretar acciones que los humanos realizan sobre los dispositivos. Son realmente cotidianos, ya que dentro de esta categoría podemos englobar a los teclados, ratones, mandos, joysticks, botones y un largo etcétera.

Al implementar un dispositivo de este tipo es necesario seguir ciertos estándares:

- Toda la información debe moverse en *Reports*, los trataremos más adelante pero son estructuras que contienen la información necesaria para la acción a realizar.
- Un endpoint de tipo *INTERRUPT IN* es necesario para enviar reports de tipo *Input* al host.
- El máximo número de endpoints *INTERRUPT* es uno *IN* y otro *OUT*, siendo este último opcional.
- Al poder el dispositivo enviar información a través del endpoint *INTERRUPT* en cualquier momento, el host debe de estar preparado para revisar el polling y recoger esta información

#### Reports

Como hemos mencionado anteriormente, los *Reports* son estructuras donde se encapsula la información que viaja en la comunicación con un dispositivo HID. Son de un tamaño fijo y previamente definido, ya que el host debe conocer esta información para generarlos.

Un dispositivo HID puede tener múltiples *Reports*, identificados por *Report IDs*, que básicamente son indentificadores en hexadecimales que permiten diferenciar el tipo de información que se envía y se recibe. Todas estas estructuras tienen un uso definido, que puede ser indefinido en caso de usarse para acciones personalizadas.

Los *Reports* se mueven a través de transferencias, y las dos más usadas dentro de esta clase de dispositivos son:

- ***GET_Report,*** permite enviar al host información.
- ***SET_Report,*** permite recibir información del host.


## Drivers USB en Linux

Los drivers son el componente software que se encarga de la comunicación entre el dispositivo USB y el host. Esencialmente, envía las instrucciones requeridas por el dispositivo para hacerlo funcionar como se espera, y su vez también permite recoger la respuesta enviada.

Todos los dispositivos, y más concretamente USB, deben de tener un driver asociado para funcionar y permitir al host comunicarse satisfactoriamente con él, en caso contrario el host no sería capaz de entender el *'lenguaje'* que habla el dispositivo y la comunicación sería imposible.

En este capítulo, veremos la arquitectura que hay debajo del USB core en los sistemas operativos Linux, los paquetes URBs que encapsulan la información de las comunicaciones, así como las diferentes APIs que nos ofrece el kernel Linux para el desarrollo de drivers USB. Terminaremos poniendo en valor el proyecto que hemos desarrollado haciendo una comparativa con el dispositivo actual de la asignatura *Arquitectura Interna de Linux y Android*.


### USB core y su arquitectura

USB Core es el mecanismo construido en el kernel Linux para el soporte del estandar USB y se encuentra en medio de los dispositivos USB y el propio sistema operativo.

Es el encargado de todo lo relacionado con USB, ya sea la enumeración de dispositivos, exponerlos a través de un árbol de ficheros y directorios, controlar la comunicación, entre otras tareas. Además, provee al programador de diferentes APIs que permiten ampliar el framework desarrollando nuevos drivers para soportar nuevos dispositivos y funcionalidades.

![Arquitectura del subsistema USB en Linux. Referencia a las transparencias de la asignatura LIN](img/usb-core.png)

Como se puede ver en la imagen anterior, el USB Core es sistema que se encuentra entre la controladora USB, que es el hardware que gestiona los dispositivos y el más cercano a ellos, y los drivers, que son los encargados de comunicarse con el dispositivo.


### URBs

Como mencionábamos en la introducción, los *URBs* son paquetes de datos donde se encapsula una petición USB, su propio nombre hace referencia a ello, ya que *URB* son las siglas de *USB Request Block* [@urb-docs].

Estas estructuras poseen todos los valores relevantes para que una petición USB pueda llevarse a cabo, pudiendo entregar los datos y obtener un estado de respuesta encapsulado también en URB. 

Cada vez que se envía uno de estos paquetes se produce una operación asíncrona, que tiene asociada una función callback que será ejecutada una vez finalice dicha operación. Además, cada dispositivo lógico soporta una cola de peticiones, por lo que el hardware puede ir llenando esta cola mientras que el driver atiende las peticiones más antiguas.

En la API USB integrada en el kernel Linux los URBs se pueden manejar de manera muy natural, esto es posible gracias a la estructura que nos ofrecen,  `struct urb`, que encapsula todos los datos que pueden llevar estos paquetes. Para el de esta estructura, el kernel también nos ofrece funciones de las cuales podemos destacar las siguientes:

- `usb_alloc_urb()`, permite reservar memoria donde alojar un URB. Esta memoria luego será compartida con el controlador USB para que envíe el paquete al dispositivo.

- `usb_fill_control_urb() / usb_fill_int_urb() / usb_fill_bulk_urb()`, son las funciones que, dependiendo del tipo de petición, nos permiten rellenar los datos que viajarán en el paquete.

- `usb_submit_urb()`, manda el URB a la controladora de host una vez está listo para ser transmitido.

- `usb_unlink_urb()`, permite cancelar la transmisión de un URB que ya ha sido enviado.




### API síncrona y asíncrona

Dentro del USB Core que viene incorporado en el kernel Linux tenemos dos modelos de API para realizar llamadas USB. El primero de ellos es el más estandar, está soportado por todos los tipos de transferencia y es el modelo ***asíncrono***. [@usb-api-docs]

La manera de funcionar del modelo asíncrono consiste en crear un *URB*, rellenarlo y posteriormente ser enviado a la controladora USB asociado a una función callback. Esta función será llamada una vez se complete la transferencia y será la encargada del siguiente paso.

Construida por encima de esta API podemos encontrar la API ***síncrona***, que principalmente hace uso de todas las funciones que nos ofrece la anterior pero además añade bloqueos una vez se envían los URBs para esperar a su respuesta. Esta API oculta al usuario todos los detalles de reserva de memoria y envío de los URBS, ya que ofrece funciones muy sencillas para enviar los datos necesarios.

A pesar de que la API síncrona es más sencilla de usar también es mucho más ineficiente, ya que bloquea el programa en todos los puntos donde se envíen paquetes, quedando sin la posibilidad de ejecutar otros trozos de código en el tiempo que vuelve la respuesta.

En este proyecto, se abordan ambas APIs para comparar los métodos de uso y dotar a los usuarios del dispositivo de diferentes ejemplos de uso de estas interfaces de programación que nos ofrece el kernel Linux.


### Blinkstick

Completar

​    
