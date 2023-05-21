<!-- Leave a blank line before the title -->

# Tecnología USB

Este capítulo consta de una introducción general al protocolo de comunicación USB, así como su implementación en Linux y los detalles a más bajo nivel que forman parte de esta implementación, tratando conceptos como los de *Descriptor USB*, *Endpoint* o *Clases USB*.

## ¿Qué es USB?

*USB* nace en 1996 como una manera de establecer la comunicación entre los ordenador y sus periféricos, buscando estandarizar la interfaz de conexión de estos ya que por aquél entonces los periféricos disponibles utilizaban diversas formas de conexión como puertos paralelos, puertos serie, PS2, entre otras. Las palabra *USB* origina de las siglas *Universal Serial Bus*, dicho nombre describe de una manera clara lo que se intenta conseguir con el desarrollo de este estándar.

Aunque originalmente este protocolo fue diseñado para usarse en ordenadores, hoy en día está presente en una inmensa variedad de dispositivos tales como teléfonos móviles, tabletas, adaptadores de carga, dispositivos de almacenamiento, etcétera. Esto ha ocasionado también algunas dificultades ya que su estandarización ha traído consigo un gran incremento de complejidad del protocolo debido a que debe de adaptarse para permitir la interoperabilidad entre dispositivos que en ocasiones son muy dispares entre sí.

## Subsistema USB en Linux

*USB Core* es el subsistema software que implementa todo el estándar USB en el kernel Linux. Este subsistema está segmentado en tres partes claramente diferenciadas, como se puede observar en la figura \ref{fig:usb-core}, que trabajan en conjunto para reconocer los dispositivos USB y conectarlos con el usuario, entre muchas otras tareas.

![Arquitectura del subsistema USB en Linux. [@usb-core-image]](img/usb-core.png){width=80% #fig:usb-core}

El propósito principal de *USB Core* es abstraer el hardware lo máximo posible y ofrecer al desarrollador una *API* con la que poder trabajar con estos dispositivos, provista de estructuras de datos, macros y funciones que restan complejidad a la integración de nuevos dispositivos con el kernel Linux.

Como vemos en la figura \ref{fig:usb-core}, la parte más cercana a los dispositivos USB son las controladoras de host USB. Estas controladoras son las encargadas de la comunicación directa con el dispositivo USB y son manejadas por drivers implementados en el kernel Linux. Como veremos en la sección [REF], hay diferentes tipos de controladoras USB, aunque su funcionamiento es similar.

En la siguiente capa encontramos toda la parte software dentro del kernel Linux, como vemos en la figura \ref{fig:usb-core}, podemos diferenciar tres partes clave dentro de esta capa:

- *Drivers para dispositivos USB*, son los controladores más cercanos al usuario. Abstraen el dispositivo USB y proveen al usuario de una manera de comunicarse con él a través de diferentes operaciones soportadas por el driver. En el capítulo Drivers USB [REF] se pueden encontrar muchos más detalles sobre estos controladores.

- *USB Core*, podemos catalogarlo como el núcleo de todo. Acerca al usuario los dispositivos USB abstrayendo el hardware lo máximo posible. Como hemos hablado antes, ofrece una API para desarrolladores que es la que permite controlar los dispositivos sin necesidad de conocer el hardware en detalle. También integra la comunicación con los drivers que manejan la controladora de host, encargada de la comunicación con el dispositivo.

- *Drivers para controladoras de host*, como vamos a ver en la sección dedicada a ello [REF] hay diversos tipos de controladoras de host y cada una es manejada por un driver diferente. Estos drivers son una parte esencial para garantizar el funcionamiento de los dispositivos USB en Linux, ya que se encargan de manejar las conexiones y transferencias a más bajo nivel con los dispositivos, actuando como intermediario entre el propio hardware y el kernel.

  

## Controladoras de Host USB

Las controladoras de Host USB son el hardware encargado de conectarse con los dispositivos USB físicamente y manejan todos los aspectos de más bajo nivel en la comunicación y conexión de estos. 

En los inicios de USB se crearon las controladoras *HDC, Host Control Device*, y existieron dos tipos que competían entre ellos por ser el estándar, lo que provocaba a los fabricantes que tuvieran que probar los dispositivos para ambos tipos de controladoras. Estas controladoras eran:

- *OHCI - Open Host Controller Interface*. Creada por *Compaq* y adaptada por la organización que controla el estándar USB, *USB-IF*. Servía para interactuar con dispositivos USB en versiones 1.0 y 1.1.
- *UHCI - Univeral Host Controller Interface*. Era la alternativa de *Intel*, por lo que los fabricantes que la integraban tenían que pagar a esta empresa unas tasas por su uso.

Más tarde, con la introducción de *USB 2.0*, la organización *USB-IF* promovió la creación de un único estándar para eliminar los problemas que originaba la competición entre *Compaq* e *Intel*. Este estándar se tradujo en las controladoras *EHCI, Enhanced Host Controller Interface.* Estas nuevas controladoras funcionan con dispositivos *USB 2.0* y cada una de ellas implementa 4 tipos de *HCD* virtuales para dar soporte a dispositivos de alta y baja velocidad.

Por último, se crean las controladoras más recientes, las *XHCI, Extensible Host Controller Interface.* Estas controladoras soportan los estándares *USB 3.X* y permiten mayores velocidades de transferencia y mejor eficiencia energética que sus antecesores.



## Velocidades de transferencia en dispositivos USB

El desarrollo del protocolo USB y su adopción en todo tipo de dispositivos ha traído consigo mejoras en las velocidades de transferencia motivadas por las utilidades que se le han dado al protocolo. Estas velocidades se han ido nombrando en base a la velocidad que pueden certificar y requieren de una versión USB concreta para ser oficialmente soportadas. Las velocidades que podemos encontrar actualmente son:

- *Low-Speed*, disponible desde la primera versión del protocolo, *USB 1.0*. Permite transferencias de hasta 1.5 *Mbps*.
- *Full-Speed*, disponible a partir de *USB 1.1*. Soporta velocidades de transmisión de hasta 12 *Mbps*.
- *Hi-Speed*, disponible desde la versión *USB 2.0*. Permite velocidades de hasta 480 *Mbps*.
- *SuperSpeed*, soportada por el estándar *USB 3.0*. Asegura velocidades de hasta 5 *Gbps*.
- *SuperSpeed+ y USB4*, disponible desde la versión *USB 3.1*, mejorando en las siguientes versiones la velocidad de transmisión.
  - *USB 3.1* - Hasta 10 *Gbps*
  - *USB 3.2* - Hasta 20 *Gbps*
  - *USB 4.0* - Hasta 40 *Gbps*



## Descriptores en USB

Los descriptores son estructuras de datos que describen las características y las capacidades de un dispositivo USB. Esencialmente, proporcionan todo tipo de información detallada sobre el dispositivo, como la velocidad de transferencia, el tipo de dispositivo o aspectos relativos a su configuración.

Una vez el dispositivo establece comunicación con la controladora de host, esta utiliza los descriptores para identificar el tipo de dispositivo ante el que se encuentra y cómo debe interacturar con él.

Las estructuras de datos con las que se implementan estos descriptores son resumidamente listas de variables *clave-valor*, ya que cada descriptor contiene un conjunto de parámetros para los que se define un valor concreto.

Los descriptores más usados dentro de USB son:

- *Device descriptors*
- *Configuration descriptors*
- *Interface descriptors*
- *Endpoint descriptors*
- *String descriptors*

<!--

En los sistemas operativos que usan un kernel Linux, podemos ver los diferentes descriptores representados de forma jerárquica junto con sus valores a través del sistema de ficheros. El kernel nos expone esta información bajo la ruta `/sys/bus/usb/devices`, donde podemos encontrar todos los dispositivos USB junto con dicha información.

![Listado por consola de los dispositivos USB y ejemplo de descriptor](img/usbDescriptorExampleLinux.png)

-->

### Device descriptors

Los descriptores de dispositivo son el tipo de descriptor más general, contiene información básica acerca del dispositivo pero a la vez muy útil. Este tipo de descriptor representa al dispositivo USB como una entidad completa, por lo que debido a su propia naturaleza, solo puede haber uno por dispositivo.

Algunos de los campos más importantes que contiene este descriptor son los siguientes:

| Parámetro       | Descripción del valor                                        |
| :-------------- | :----------------------------------------------------------- |
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

### Jerarquía de descriptores

Ahora que ya conocemos los tipos de descriptores que existen, es importante destacar que estos descriptores son contenidos de manera jerárquica por otros como ilustra la figura \ref{fig:jerarquia-descriptores}.

![Jerarquía de descriptores USB. [@jerarquia-descriptores-image]](img/jerarquia-descriptores.png){width=80% #fig:jerarquia-descriptores}

El descriptor de tipo *Device* es el más genérico de todos ya que describe al dispositivo en sí como un único componente hardware. Después tenemos los descriptores de tipo *Configuration*, que representan diferentes configuraciones con las que puede trabajar el dispositivo. Dentro de cada configuración pueden haber diferentes interfaces, descritas por el descriptor de tipo *Interface* y estas a su vez pueden contener diferentes endpoints, descritos por el descriptor de tipo *Endpoint*.


## Endpoints

Los endpoints dentro de un dispositivo USB podrían describirse como buffers de datos, los cuales pueden ser rellenados por el mismo dispositivo para enviar dichos datos al host o pueden ser llenados por el host para recibir datos en el dispositivo. De cualquier manera, los endpoints son una propiedad única del hardware USB en si mismo, independiente del sistema operativo o del host.

Los endpoints pueden ser de dos tipos según la dirección en la que se transfieren sus datos, con excepción del *Endpoint 0* que es bidireccional ya que así viene definido por el estándar. La dirección de los datos se especifica en el descriptor y puede ser:

- *IN*, representa la dirección de las transferencias del dispositivo USB al host.
- *OUT*, representa la dirección de las transferencias del host al dispositivo USB.

Existe una limitación en cuanto al número de endpoints que un dispositivo puede tener, como máximo 32 endpoints, 16 endpoints IN y 16 endpoints OUT.

Dentro de los endpoints podemos encontrar diferentes tipos definidos para diferentes usos, los más importantes son CONTROL, INTERRUPT y BULK. Cada uno de ellos está diseñado para cumplir unos requisitos de velocidad, eficiencia y cantidad de datos enviados por lo que deben de usarse en su correcto contexto.

### Control Endpoints

Los endpoints de control se encargan de las transferencias de información relativa a la configuración del dispositivo y pueden ser de tipo IN o de tipo OUT.

Aunque este tipo de endpoints no se suelen usar para transferencias masivas de datos debido a sus limitaciones y uso restringido a configuración, pueden usarse para mandar algunos bytes de datos.

El endpoint 0 siempre está reservado para la configuración del dispositivo con el host y no tienen ningún descriptor asignado, ya que toda su configuracion está definida por el estandar USB. De igual manera es especial en cuanto a su tipo de dirección, ya que es accesible tanto por el tipo IN como por el OUT.

<!--

La etapa de configuración podemos dividirla en tres pasos:

- SETUP, es un paquete que define el tipo de petición y el tamaño de los datos que se transmitirán en la etapa DATA.
- DATA, esta parte del proceso es opcional y si existe siempre empieza con un paquete DATA1. Después, si aún quedan datos por transferir se altera entre paquetes DATA1 y DATA0.
- STATUS, es un paquete de confirmación. Básicamente se envía en el sentido contrario un paquete DATA1 vacío.

-->

### Interrupt Endpoints

Este tipo de transferencias se utiliza cuando la comunicación entre el dispositivo y el host es asíncrona. La manera de avisar que se quieren recibir datos es a través de un *polling*, cuyo intervalo de consulta es parametrizable por el usuario de 1ms  a 255ms para dispositivos *low-Speed* y de 125us a 4096ms para dispositivos *high-Speed*.

El sentido de la comunicación puede ser *IN* o *OUT*. Para comunicarse, se revisa el endpoint con el intervalo definido por el polling para ver si hay alguna interrupción pendiente de atender, si es así, se atiende. Esto es igual para ambas direcciones, cuando sea de tipo IN se enviarán los datos en un sentido y cuando sea de tipo *OUT* se realizará en sentido contrario.

### Bulk Endpoints

Los endpoints de tipo Bulk se usan para enviar grandes cantidades de información por el bus USB. Los escenarios en los que este tipo de endpoints son necesarios son muy variados, desde enviar un documento a una impresora para imprimir, hasta copiar un fichero de datos a un disco USB.

Para reservar ancho de banda, este tipo de transferencias usan espacio que otras transferencias no han usado una vez han terminado. Por ello, puede que las transferencias de este tipo se vuelvan algo lentas si no dispone de espacio en el bus.

Este tipo de transferencias solo son soportadas por dispositivos *Full* y *High-Speed* y dependiendo del estandar que cumplan tienen un tamaño máximo de transferencia u otro. También cuentan con detección de errores *CRC*, por lo que permite asegurar la integridad de la información transmitida.

Al igual que los tipos anteriores, permite transferencias de tipo *IN* o *OUT*.


## Clases USB

La organización usb.org [@usb-org] tiene definidas una serie de códigos de clase que se utilizan para identificar la funcionalidad del dispositivo y ayudan al host a cargar un driver que sea compatible basado en esa funcionalidad.

La información se representa en tres bytes de datos hexadecimales y contiene:

- Byte 1 - Clase base. En este byte se describe el uso del dispositivo de manera general. Por ejemplo: `01h` para dispositivos de audio o `07h` para impresoras.

- Byte 2 - Subclase. Este byte describe dentro de la clase el uso concreto que se le va a dar. Por ejemplo, dentro de la clase de audio podemos encontrar `00h` para dispositivos de streaming o 03h para dispositivos MIDI.

- Byte 3 - Protocolo. En este último byte se define el protocolo a usar. Siguiendo con el ejemplo de Audio, podemos usar `00h` para USB 1.0, `20h` para USB 2.0 o `02h` para especificar que es asíncrono.

Como veíamos en el apartado anterior, esta información viaja al host a través de los descriptores y podemos encontrarla o bien en el descriptor de dispositivo, en el de interfaz o incluso en ambos.

Aunque en este proyecto solo se trabaja y se trata con la clase HID, *Human Interface Device*, hay algunas otras clases que también son muy conocidas y usadas como:

- ***Audio Class.*** Usada por dispositivos de audio.

- ***CDC Class.*** Usada por dispositivos de comunicación.

- ***Mass Storage Class.*** Usada por dispositivos de almacenamiento.

### Human Interface Device Class (HID)

La clase de dispositivos HID, codificada como `03h`,se usa principalmente para representar dispositivos que se comunican con el host y son controlados por humanos. A través de este tipo de dispositivos, el host es capaz de interpretar acciones que los humanos realizan sobre los dispositivos. Son realmente cotidianos, ya que dentro de esta categoría podemos englobar a los teclados, ratones, mandos, joysticks, botones y un largo etcétera.

Al implementar un dispositivo de este tipo es necesario seguir ciertos estándares:

- Toda la información debe moverse en *Reports*. Los trataremos más adelante pero son estructuras que contienen la información necesaria para la acción a realizar.
- Un endpoint de tipo *INTERRUPT IN* es necesario para enviar reports de tipo *Input* al host.
- El máximo número de endpoints *INTERRUPT* es uno *IN* y otro *OUT*, siendo este último opcional.
- Al poder el dispositivo enviar información a través del endpoint *INTERRUPT* en cualquier momento, el host debe de estar preparado para revisar el *polling* y recoger esta información

#### Reports

Como hemos mencionado anteriormente, los *Reports* son estructuras donde se encapsula la información que viaja en la comunicación con un dispositivo HID. Son de un tamaño fijo y previamente definido, ya que el host debe conocer esta información para generarlos.

Un dispositivo HID puede tener múltiples *Reports*, identificados por *Report IDs*, que básicamente son indentificadores hexadecimales que permiten diferenciar el tipo de información que se envía y se recibe. Todas estas estructuras tienen un uso definido, que puede ser indefinido en caso de emplearse para acciones personalizadas.

Los *Reports* se mueven a través de transferencias, y las dos más usadas dentro de esta clase de dispositivos son:

- ***GET_Report,*** permite enviar al host información.
- ***SET_Report,*** permite recibir información del host.



<!--

## Drivers USB en Linux

Los drivers son el componente software que se encarga de la comunicación entre el dispositivo USB y el host. Esencialmente, envía las instrucciones requeridas por el dispositivo para hacerlo funcionar como se espera, y su vez también permite recoger la respuesta enviada.

Todos los dispositivos, y más concretamente USB, deben de tener un driver asociado para funcionar y permitir al host comunicarse satisfactoriamente con él, en caso contrario el host no sería capaz de entender el *'lenguaje'* que habla el dispositivo y la comunicación sería imposible.

En este capítulo, veremos la arquitectura que hay debajo del USB core en los sistemas operativos Linux, los paquetes URBs que encapsulan la información de las comunicaciones, así como las diferentes APIs que nos ofrece el kernel Linux para el desarrollo de drivers USB. Terminaremos poniendo en valor el proyecto que hemos desarrollado haciendo una comparativa con el dispositivo actual de la asignatura *Arquitectura Interna de Linux y Android*.


### USB core y su arquitectura

USB Core es el mecanismo construido en el kernel Linux para el soporte del estandar USB y se encuentra en medio de los dispositivos USB y el propio sistema operativo.

Es el encargado de todo lo relacionado con USB, ya sea la enumeración de dispositivos, exponerlos a través de un árbol de ficheros y directorios, controlar la comunicación, entre otras tareas. Además, provee al programador de diferentes APIs que permiten ampliar el framework desarrollando nuevos drivers para soportar nuevos dispositivos y funcionalidades.

Como se puede ver en la imagen anterior, el USB Core es sistema que se encuentra entre la controladora USB, que es el hardware que gestiona los dispositivos y el más cercano a ellos, y los drivers, que son los encargados de comunicarse con el dispositivo.

-->

## URBs

Los *URBs* son paquetes de datos donde se encapsula una petición USB, su propio nombre hace referencia a ello, ya que *URB* son las siglas de *USB Request Block* [@urb-docs]. Esta estructura posee todos los valores relevantes para que una petición USB pueda llevarse a cabo, pudiendo entregar los datos y obtener un estado de respuesta encapsulado también en URB. 

Cuando se emplea la *API* asíncrona, cada vez que se envía uno de estos paquetes se produce una operación asíncrona, que tiene asociada una función callback que será ejecutada una vez finalice dicha operación. Además, cada dispositivo lógico soporta una cola de peticiones, por lo que el hardware puede ir llenando esta cola mientras que el driver atiende las peticiones más antiguas.

En la API USB integrada en el kernel Linux los URBs se pueden manejar de manera muy natural, esto es posible gracias a la estructura que nos ofrecen,  `struct urb`, que encapsula todos los datos que pueden llevar estos paquetes. Para la gestión de esta estructura, el kernel también nos ofrece funciones de las cuales podemos destacar las siguientes:

- `usb_alloc_urb()`, permite reservar memoria donde alojar un URB. Esta memoria luego será compartida con el controlador USB para que envíe el paquete al dispositivo.

- `usb_fill_control_urb() / usb_fill_int_urb() / usb_fill_bulk_urb()`, son las funciones que, dependiendo del tipo de endpoint, nos permiten rellenar los datos que viajarán en el paquete.

- `usb_submit_urb()`, manda el URB a la controladora de host una vez está listo para ser transmitido.

- `usb_unlink_urb()`, permite cancelar la transmisión de un URB que ya ha sido enviado.




## API síncrona y asíncrona

Dentro del USB Core que viene incorporado en el kernel Linux tenemos dos modelos de API para realizar llamadas USB. El primero de ellos es el más estandar, está soportado por todos los tipos de transferencia y es el modelo ***asíncrono***. [@usb-api-docs]

La manera de funcionar del modelo asíncrono consiste en crear un *URB*, rellenarlo y posteriormente ser enviado a la controladora USB asociado a una función *callback*. Esta función será llamada una vez se complete la transferencia y será la encargada del siguiente paso.

Construida por encima de esta API podemos encontrar la API ***síncrona***, que principalmente hace uso de todas las funciones que nos ofrece la anterior pero además añade bloqueos una vez se envían los URBs para esperar a su respuesta. Esta API oculta al desarrollador todos los detalles de reserva de memoria y envío de los URBS, ya que ofrece funciones muy sencillas para enviar los datos necesarios.

A pesar de que la API síncrona es más sencilla de usar también es mucho más ineficiente, ya que bloquea el programa en todos los puntos donde se envíen paquetes, quedando sin la posibilidad de ejecutar otro código durante el tiempo que transcurre hasta que se recibe la respuesta.

En este proyecto, se abordan ambas APIs para comparar los métodos de uso y dotar a los usuarios del dispositivo de diferentes ejemplos de uso de estas interfaces de programación que nos ofrece el kernel Linux.


## Blinkstick Strip

El desarrollo de este TFG está muy enfocado en la docencia, ya que lo que se pretende es conseguir un dispositivo que sea barato de producir y muy versátil para poder desarrollar diversos tipos de drivers y aprender el funcionamiento de las diferentes partes que se involucran en este campo.

Actualmente, en la asignatura optativa *Arquitectura Interna de Linux y Android*, se está usando un dispositivo llamado *Blinkstick* Strip [@blinkstick-page] para impartir este tipo de lecciones donde los alumnos aprenden a desarrollar drivers para el kernel Linux.

![Dispositivo Blinkstick usado actualmente en LIN [@blinkstick-page]](img/blinkstick.jpg)

Pero este dispositivo tiene diferentes aspectos negativos que hacen de él una opción no del todo aconsejable, ya que el precio de venta es demasiado elevado para lo que ofrece y la cantidad de material que hay que adquirir para un aula. También es demasiado sencillo, lo que limita en gran medida el tipo de drivers que los alumnos pueden desarrollar y la API que pueden usar, ya que este dispositivo solo admite transferencias de tipo *CONTROL*.

Por ello, nuestro dispositivo ofrece significativas ventajas tanto en precio como en versatibilidad:

- Comparado con el coste de adquirir *Blinksticks*, nuestro dispositivo es mucho más barato de producir.
- Dispone de diferentes endpoints *CONTROL* y *INTERRUPT* lo que nos permite desarrollar diferentes tipos de drivers usando ambas APIs.
- El *Blinkstick* solamente cuenta con unos cuántos diodos LEDs, mientras que nuestro dispositivo tiene ya implementados diversos periféricos de varios tipos con gran posibilidad de ampliación debido a la arquitectura del firmware y la disponibilidad de pines en el microcontrolador usado.



​    
