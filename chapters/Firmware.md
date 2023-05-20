<!-- Leave a blank line before the title -->

# Firmware: Diseño del código

El firmware es el código fuente que se ejecuta dentro del microcontrolador responsable de controlar los periféricos con los que interactúa, así como de gestionar un extremo de la comunicación USB gracias a V-USB. En este capítulo se describen con detalle las partes y características más importantes del firmware, así como las librerías auxiliares utlizadas para los periféricos

## Motivación

El diseño del firmware juega un papel muy importante dentro de lo que se pretende conseguir con este proyecto, ya que tiene la capacidad de soportar y exponer diferentes periféricos a través de un mismo dispositivo USB gracias a plantillas configurables por el usuario o el docente. Esto brinda la posibilidad de tener "diferentes" dispositivos con los que interactuar y que de cara al desarrollo de drivers sean completamente distintos. Esta flexibilidad nos permite también incorporar de manera sencilla placas de expansión como Bee, usada actualmente en la Facultad de Informática, que nos proporciona múltiples periféricos en un mismo espacio haciendo el hardware mucho más flexible.

Dentro del entorno docente, esto permite disponer de un gran abanico de opciones para realizar prácticas en las asignaturas donde se imparta el desarrollo de drivers para dispositivos USB, ya que con un mismo dispositivo físico, podemos tener periféricos con maneras diferentes de ser accedidos y que permiten explorar muchas partes de la API USB presente en el kernel Linux. La manera en la que se ha diseñado la forma de seleccionar qué periféricos están disponibles también es otro punto a recalcar, ya que al configurarse a través de macros, `define`, permite a profesores y usuarios de la infraestructura escoger un subconjunto de prácticas que le interesen sin necesidad de tener el conocimiento de cómo programar el dispositivo.

## Perfiles de hardware

Una de las partes claves del diseño del código es la segmentación en perfiles según los periféricos que se deseen utilizar. Esto permite crear diferentes tipos de dispositivos simplemente configurando la opción en el fichero correspondiente. Dependiendo de la opción seleccionada, por ejemplo, pueden solo habilitarse las funciones y *Report IDs* relativos a la iluminación LED o a las pantallas y displays, quedando dispositivos con funcionalidades completamente diferentes pero usando el mismo código. No hay limitación de plantillas a elegir, todas las que se marquen como `ON` serán incluidas en el código compilado.

El fichero que orquesta toda la configuración de las plantillas es `deviceconfig.h` y se encuentra en la raíz del directorio `firmware/`. En él se definen mediante el uso de macros y sentencias condicionales los diferentes periféricos implementados, así como sus conexiones.

Actualmente se encuentran disponibles 4 plantillas:

- LED_PARTY. Cuando se habilita esta plantilla el dispositivo expone mediante los *Report IDs* 1 y 2 de tipo *SET* un anillo de LED configurable. También expone en el *Report ID* 1 de tipo *GET* la capacidad de devolver el color actual del anillo.
- DISPLAYS. Esta pantilla habilita periféricos como el display 7 segmentos o la pantalla OLED, los hace accesibles a través de los *Report IDs* 3 y 4, ambos de tipo *SET*.
- PWM. En esta plantilla se trabaja con los TIMERs hardware que nos ofrece el chip que estamos usando y se expone al usuario un buzzer y un led controlado por este hardware, que es una manera excelente de darle uso ya que libera a la CPU del control. Expone el led en el *Report ID* 5 y el buzzer en el 6, ambos de tipo *SET*.
- ALL_DEVICES. Es la plantilla más versatil ya que habilita todos los periféricos implementados.

![Muestra de código relativo a la elección de la plantilla](img/firmwareTemplate.png)

Además de estos periféricos, el *Report ID* 2 de tipo *GET* siempre está accesible y devuelve al usuario un buffer de 32 bytes con un string concreto. Esto es muy útil para practicar la recepción desde el host enviados por el dispositivo.

La manera en la que se ha implementado esta funcionalidad ha sido usando compilación condicional. Como hemos visto, en el fichero mencionado se encuentra definido todo lo que el usuario desea tener y una vez se compila el proyecto solo se cogen las partes especificadas, ahorrando de esta manera también memoria flash que en estos chips no suele ser muy abundante.

## main.cpp

El fichero `main.cpp` es el punto central de la implementación, en él se maneja tanto la comunicación y respuesta USB, como la inicialización de los periféricos, reserva de memoria necesaria, entre otras acciones.

En este fichero podemos encontrar implementadas diferentes funciones de la librería V-USB, que han sido previamente definidas en los ficheros de cabecera de la propia librería.

En los siguientes apartados, se describen tres de las funciones clave en la comunicación USB.

### Función `usbMsgLen_t usbFunctionSetup(uchar)`

Esta función es el principal punto de entrada cuando se empieza a procesar un paquete USB procedente del host. La función recibe como parámetro 8 bytes de datos que contienen información como el *ReportID* al que va dirigido el paquete, el número de bytes recibidos, también conocido en terminología USB como *wLenght* o el tipo de transmisión, *bRequest*, que puede ser de tipo SET o GET en nuestro caso.

Dentro de esta función se examinan los diferentes bytes para ir tomando decisiones sobre qué código ejecutar. Se mira primero la clase de la petición junto con el tipo de transferencia, para posteriormente comprobar qué *ReportID* es que ha recibido.

En caso de que el código que haya que ejecutar para realizar la petición con éxito no sea demasiado simple o necesite más de 8 bytes de datos se debe de ejecutar la función `usbFunctionRead(uchar)` en transferencias de lectura o `usbFunctionWrite(uchar)` en caso de escritura, estas funciones serán llamadas tantas veces como sea necesario en trozos de 8 bytes hasta completar la longitud total del mensaje. Para que esto suceda, la función debe devolver la macro `USB_NO_MSG`. Por otro lado, si la petición puede completarse en esta función se devolverá el número de bytes que se han escrito en el buffer que será devuelto al host.

### Funciones `uchar usbFunctionRead(uchar*, uchar)` y `uchar usbFunctionWrite(uchar*, uchar)`

Como mencionaba la sección anterior, estas funciones son llamadas por el core de V-USB cuando en la función `usbFunctionSetup()` se devuelve el valor *USB_NO_MSG*. 

Estas funciones se llaman con 8 bytes de datos que residen dentro del buffer que se envía como argumento junto con la longitud del mismo. La longitud total de los datos de la transferencia en curso se puede encontrar en el campo *wLenght* accesible desde la función de setup, de esta manera podemos calcular cuántos bytes nos quedan por leer y así acabar la transacción o si por el contrario tenemos que esperar más datos, esto es útil para lecturas de más de 8 bytes.

Si durante la ejecución de la función se produce algún error debemos retornar el valor -1, que se convertira en un *STALL* informando al host del fallo.

En la función de lectura, cuando se quieren seguir leyendo datos se debe devolver el valor 0, en caso de haber finalizado se devolverá 1. Similar pasa con la función de escritura, pero en este caso siempre se debe devolver 1 cuando la transferencia haya finalizado con éxito.

Es importante recalcar, que ambas funciones deben de ser previamente definidas por V-USB y para ello hay que activar dos macros que haran disponible la cabecera para su implementación, `USB_CFG_IMPLEMENT_FN_WRITE` y `USB_CFG_IMPLEMENT_FN_READ`.

## usbconfig.h

Este fichero proviene del proyecto V-USB, es obligatorio incluirlo ya que contiene toda la configuración del dispositivo a emular. Cuando comenzamos la fase de codificación tuvimos que darle un gran repaso, ya que dependiendo de lo que esté definido se creará un dispositivo diferente.

Este fichero contiene muchísimos ajustes, ya que como comentamos en otro punto de esta memoria, V-USB soporta muchos tipos de microcontroladores y es ampliamente configurable, pero en esta sección repasaremos algunos de los más importantes.

Un conector USB sencillo tiene 4 líneas, 2 encargadas de la alimentación y otras dos de los datos. En este fichero se definen que puertos del microcontrolador son los responsables de este control.

- `USB_CFG_DMINUS_BIT`, este parámetro define el puerto del microcontrolador donde está conectada la línea negativa del conector.

- `USB_CFG_DPLUS_BIT`, en este se define el puerto donde se conecta la línea positiva del conector.

La frecuencia a la que trabaja el microcontrolador también es un parámetro clave, ya que esta implementación de USB es emulada por software la frecuencia debe de ser muy estricta para cumplir con los tiempos de la comunicación. Se define en la macro `USB_CFG_CLOCK_KHZ`.

Como hemos visto anteriormente, el dispositivo que hemos implementado pertenece a la clase HID, que debe de cumplir algunos requisitos concretos. Estos requisitos los definimos en las siguientes macros:

- `USB_CFG_HAVE_INTRIN_ENDPOINT`, esta macro nos permite activar un endpoint de tipo INTERRUPT IN, necesario en HID.
- `USB_CFG_INTERFACE_CLASS` , otro punto muy importante es definir la clase del dispositivo. Como en nuestro caso se trata de HID, esta macro llevará el valor 3.

Otros dos valores muy importantes en la implementación de un dispositivo USB son el ProductID y VendorID, ya que sin ellos sería imposible asociar un dispositivo con un driver. Por ello, dentro de este fichero podemos definirlos bajo las macros: `USB_CFG_VENDOR_ID` y `USB_CFG_DEVICE_ID`.

Por último, otros dos valores interesantes de configurar son el nombre del fabricante y del dispositivo, que posteriormente serán mostrados una vez el host reconozca el dispositivo. Podemos hacer esto dándole valor a las siguientes macros:  `USB_CFG_VENDOR_NAME` y `USB_CFG_DEVICE_NAME`.

##  utils.h

En este fichero se implementan diferentes funciones que nos permiten controlar los componentes más básicos como pueden ser los diodos leds y el buzzer. 

Aquí también se trabaja con los timers que nos ofrece el hardware en las funciones `blinkPWM() y `harwarePWMBeep(uint16_t)`, que son las encargadas de empezar a parpadear el led con frecuencia de 1 segundo y de generar una señal PWM con la frecuencia indicada por parámetro.

Es un fichero relativamente sencillo pero de gran uso, ya que libera del fichero de ejecución principal el código de estas funciones.

## Otras librerías usadas

Durante el proyecto, hemos usado también dos librerías de la comunidad que nos han permitido integrar en el software tanto el anillo de leds como el display de tipo OLED.

Estas librerías son:

- `oled/`, se encarga de controlar el display OLED mediante el protocolo de comunicación *i2c* que solo usa dos líneas de datos para la transmisión de la información. [@wagiminator-profile]
- `light_ws2812/`, esta librería implementa el protocolo *one-wire* que nos permite controlar el anillo de led. [@ws2812-library]

## Depuración del código del microcontrolador

La depuración en microcontroladores de este tipo puede llegar a ser todo un reto. Tenemos que tener en cuenta que un chip de esta clase no es más que una diminuta compleja pieza de hardware que ejecuta el código que previamente le ha sido cargado en una memoria flash, siendo imposible trazarlo como se podría hacer en otro entorno más cercano al usuario como puede ser GDB.

Por ello, hay que buscar maneras auxiliares de depuración. En los inicios utilizabamos un diodo led para indicar que el procesador estaba ejecutando ciertas partes del código, pero por temas de temporización esta manera no era del todo efectiva, además de la poca información que daba.

Con el cambio de chip al ATMega328p pudimos hacer uso del conversor serie que lleva la placa que elegimos y establecer una línea de comunicación por puerto serie con el dispositivo. V-USB nos ofrece una interfaz de debug simple pero efectiva en el fichero `oddebug.c` con la que podemos aprovechar este hardware, esencialmente pone a funcionar un dispositivo serie por el que posteriormente expondremos cadenas de caracteres que usaremos para debuggear el código.

A este fichero añadimos una función que nos resultó muy cómoda para poder imprimir cadenas de caracteres y es la función `int odPrintf(char *fmt, ...)`.

De esta manera, pudimos incluir trazas distribuidas por el código para ver los diferentes flujos de ejecución, así como el valor de determinadas variables.

![Muestra de código con el que se inicializa la interfaz de debug y se envía una cadena de información](img/debugInitialize.png)

