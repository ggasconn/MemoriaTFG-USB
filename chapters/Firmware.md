<!-- Leave a blank line before the title -->

# Firmware: Diseño del código {#sec:firmware}

El firmware es el código fuente que se ejecuta dentro del microcontrolador responsable de controlar los periféricos con los que interactúa, así como de gestionar un extremo de la comunicación USB. En este capítulo se describen con detalle las partes y características más importantes del firmware, así como las librerías auxiliares utlizadas para los periféricos

## Motivación

El diseño del firmware juega un papel muy importante dentro de lo que se pretende conseguir con este proyecto, ya que tiene la capacidad de soportar y exponer diferentes periféricos a través de un mismo dispositivo USB gracias a plantillas configurables por el usuario o el docente. Esto brinda la posibilidad de tener "diferentes" dispositivos con los que interactuar y que de cara al desarrollo de drivers sean completamente distintos. Esta flexibilidad nos permite también incorporar de manera sencilla placas de expansión como Bee, que nos proporciona múltiples periféricos en un mismo espacio haciendo el hardware mucho más flexible.

Dentro del entorno docente, esto permite disponer de un gran abanico de opciones para realizar prácticas en las asignaturas donde se imparta el desarrollo de drivers para dispositivos USB, ya que con un mismo dispositivo físico, podemos tener periféricos con maneras diferentes de ser accedidos y que permiten explorar muchas partes de la API USB presente en el kernel Linux. La manera en la que se ha diseñado la forma de seleccionar qué periféricos están disponibles también es otro punto a recalcar, ya que al configurarse a través de macros, `define`, permite a profesores y usuarios de la infraestructura escoger un subconjunto de prácticas que le interesen sin necesidad de tener el conocimiento de cómo programar el dispositivo.

## Perfiles de hardware

Una de las partes claves del diseño del código es la segmentación en perfiles según los periféricos que se deseen utilizar. Esto permite crear diferentes tipos de dispositivos simplemente configurando la opción en el fichero correspondiente. Dependiendo de la opción seleccionada, por ejemplo, pueden solo habilitarse las funciones y *Report IDs* relativos a la iluminación LED o a las pantallas y displays, quedando dispositivos con funcionalidades completamente diferentes pero usando el mismo código.

La configuración de estos perfiles se realiza a través de una plantilla que contiene los diferentes perfiéricos agrupados según su funcionalidad. Lo primero que encontramos en ella son dos macros definidas para activar (*ON*) o desactivar (*OFF*) de manera amigable los perfiles. A continuación, están definidos los perfiles que actualmente están soportados junto con su estado, *ON* u *OFF*. Y por último, y de gran importancia, para permitir la  flexibilidad en el hardware, se encuentran definidas una serie de macros que indican dónde están conectados todos los periféricos, de esta manera podríamos cambiar de placa o simplemente mover un periférico de puerto con tan solo variar el valor en este fichero. El siguiente ejemplo de código ilustra de manera clara las macros que se explican en este párrafo:

```C
#define ON  1
#define OFF  0

#define ALL_DEVICES  ON
#define LED_PARTY  OFF || ALL_DEVICES
#define DISPLAYS  OFF || ALL_DEVICES
#define PWM  OFF || ALL_DEVICES

#if LED_PARTY == 1
    // LED ring
    #define LED_RING_PORT B
    #define LED_RING_PIN 0
    #define BUILTIN_LED PB5
    #define RX_LED PD0
#endif

#if DISPLAYS == 1
    // OLED
    #define OLED_PORT PORTB
    #define OLED_PIN DDRB
    #define OLED_SCL PB2
    #define OLED_SDA PB0

    // Display 7seg
    #define D7S_PORT PORTB
    #define D7S_PIN DDRB
    #define	D7S_DATA 0
    #define	D7S_RCLK 1
    #define	D7S_SRCLK 2
#endif

#if PWM == 1
    #define BUZZER_PIN PORTB1
    #define PWM_LED PORTB1
#endif
```

El fichero plantilla que orquesta toda la configuración de los perfiles es `deviceconfig.h` y se encuentra en la raíz del directorio `firmware/` . No hay limitación de perfiles a elegir, todos los que se marquen como `ON` serán incluidos en el código compilado. Actualmente hay disponibles 4 perfiles:

- LED_PARTY. Cuando se habilita esta plantilla el dispositivo expone mediante los *Report IDs* 1 y 2 de tipo *SET* un anillo de LED configurable, el cual contiene 12 LEDs que se pueden configurar por separado creando escenas de color muy distintas. También expone en el *Report ID* 1 de tipo *GET* la capacidad de devolver el color actual del anillo cuando este se encuentra del mismo color en todos los LEDs.
- DISPLAYS. Esta pantilla habilita periféricos como el display 7 segmentos o la pantalla OLED, los hace accesibles a través de los *Report IDs* 3 y 4, ambos de tipo *SET*. Estos displays consiguen imprimir el dígito hexadecimal o la cadena enviada desde el host, respectivamente.
- PWM. En esta plantilla se trabaja con los TIMERs hardware que nos ofrece el chip que estamos usando y se exponen al usuario dos periféricos. Por un lado contiene un buzzer, que puede ser establecido a una frecuencia de vibración produciendo un tono diferente dependiendo del valor, esto permite crear melodías variando la frecuencia y espera de dicho tono. Y por otro lado, expone un diodo LED con capacidad de parpadear siendo controlado por este hardware. Esta aplicación de los TIMERs es una manera excelente de darle uso ya que libera a la CPU del control. Expone el led en el *Report ID* 5 y el buzzer en el 6, ambos de tipo *SET*.
- ALL_DEVICES. Es la plantilla más versatil ya que habilita todos los periféricos implementados.

Además de estos periféricos, el *Report ID* 2 de tipo *GET* siempre está accesible y devuelve al usuario un buffer de 32 bytes con un string concreto. Esto es muy útil para practicar la recepción desde el host enviados por el dispositivo.

La manera en la que se ha implementado esta funcionalidad ha sido usando compilación condicional. Como hemos visto, en el fichero mencionado se encuentra definido todo lo que el usuario desea tener y una vez se compila el proyecto solo se cogen las partes especificadas, ahorrando de esta manera también memoria flash que en estos chips no suele ser muy abundante.

## main.cpp

El fichero `main.cpp` es el punto central de la implementación, en él se maneja tanto la comunicación y respuesta USB, como la inicialización de los periféricos, reserva de memoria necesaria, entre otras acciones. También contiene el bucle principal del firmware, donde se realiza una encuenta para comprobar si hay mensajes pendientes de responder, además de atender las interrupciones cuando paquetes de tipo INTERRUPT IN están esperando ser atendidos. El siguiente código muestra el bucle principal:

```C
int main(void) {
    
	...
        
	for(;;){				/* main event loop */
		wdt_reset();
		usbPoll();

		/* Check if there's any INTERRUPT IN URB to fill in */
		if(usbInterruptIsReady())
            // Fill URB buffer with data;
			usbSetInterrupt(INT_IN_MSG, INTERRUPT_TRANSFER_SIZE); 
	}

	return 0;
}

```



Además en este fichero podemos encontrar implementadas diferentes funciones de la librería V-USB, que han sido previamente definidas en los ficheros de cabecera de la propia librería.

En los siguientes apartados, se describen tres de las funciones clave en la comunicación USB.

### Función `usbFunctionSetup`

Esta función es el principal punto de entrada cuando se empieza a procesar un paquete USB procedente del host. La función recibe como parámetro 8 bytes de datos que contienen información como el *ReportID* al que va dirigido el paquete, el número de bytes recibidos, también conocido en terminología USB como *wLenght* o el tipo de transmisión, *bRequest*, que puede ser de tipo SET o GET en nuestro caso.

Dentro de esta función se examinan los diferentes bytes para ir tomando decisiones sobre qué código ejecutar. Se analiza primero la clase de la petición junto con el tipo de transferencia, para posteriormente comprobar qué *ReportID* es el que ha recibido.

En caso de que el código que haya que ejecutar para realizar la petición con éxito no sea demasiado simple o necesite más de 8 bytes de datos se debe de ejecutar la función `usbFunctionRead(uchar)` en transferencias de lectura o `usbFunctionWrite(uchar)` en caso de escritura, esto ocurre por que la función de *setup* se llama una sola vez con 8 bytes de datos y el diseño del framework obliga a realizar el procesamiento de transferencias superiores a 8 bytes de datos en las funciones correspondientes. Estas funciones serán llamadas automáticamente por V-USB tantas veces como sea necesario en trozos de 8 bytes hasta completar la longitud total del mensaje. Para que esto suceda, la función debe devolver la macro `USB_NO_MSG`. Por otro lado, si la petición puede completarse en esta función se devolverá el número de bytes que se han escrito en el buffer que será devuelto al host. En el siguiente trozo de código se puede observar el tratamiento de algunos de los ReportIDs implementados y la invocación de las funciones write y read por transferencias superiores a 8 bytes:

```C
extern "C" usbMsgLen_t usbFunctionSetup(uchar data[8]) {
	usbRequest_t *rq = (usbRequest_t *)data;
	reportId = rq->wValue.bytes[0];

    /* HID class request */
	if((rq->bmRequestType & USBRQ_TYPE_MASK) == USBRQ_TYPE_CLASS){ 
        /* wValue: ReportType (highbyte), ReportID (lowbyte) */
		if(rq->bRequest == USBRQ_HID_GET_REPORT) { 
			 usbMsgPtr = replyBuffer;

			if (reportId == 1) {
				odPrintf("Received GET on ReportID 1\n");

				replyBuffer[0] = 2; // report id

				bytesRemaining = 33;
				currentAddress = 0;

				return USB_NO_MSG; /* use usbFunctionRead() to obtain data */

			} 
			#if LED_PARTY == 1
			else if(reportId == 2) { // Device colors
				odPrintf("Received GET on ReportID 2\n");

				replyBuffer[0] = 1; // report id
				replyBuffer[1] = r;
				replyBuffer[2] = g;
				replyBuffer[3] = b;

				return 4; /* Return the data from this function */
			}
			#endif

			return 0;

		}
        
        ...
            
	}

	return 0;	/* return no data back to host */
}
```



### Funciones `usbFunctionRead` y `usbFunctionWrite`

Como se mencionaba en la sección anterior, estas funciones son llamadas por el core de V-USB cuando en la función `usbFunctionSetup()` se devuelve el valor *USB_NO_MSG*. 

Estas funciones se llaman con 8 bytes de datos que residen dentro del buffer que se envía como argumento junto con la longitud del mismo. La longitud total de los datos de la transferencia en curso se puede encontrar en el campo *wLenght* accesible desde la función de setup. De esta manera podemos calcular cuántos bytes nos quedan por leer y así acabar la transacción o si por el contrario tenemos que esperar más datos, esto es útil para lecturas de más de 8 bytes.

Si durante la ejecución de la función se produce algún error debemos retornar el valor -1, que se convertirá en un paquete *STALL*, dicho paquete indica al host que ha habido un error del que no ha sido posible recuperarse durante la transferencia y se ha abortado.

En la función de lectura, cuando se quieren seguir leyendo datos se debe devolver el valor 0, en caso de haber finalizado se devolverá 1. Algo similar se ha de hacer en la función de escritura, pero en este caso siempre se debe devolver 1 cuando la transferencia haya finalizado con éxito. El código que viene a continuación, pertenece a la operación de lectura e ilustra de manera más clara como trabajan dichas funciones:

```C
uchar usbFunctionRead(uchar *data, uchar len) {
	if (reportId == 1) {
		if(len > bytesRemaining)
			len = bytesRemaining;

		if (currentAddress == 0) {
			memcpy(&data[1], &MSG[currentAddress], len);
			currentAddress += (len - 1);
			bytesRemaining -= (len - 1);
		} else {
			memcpy(data, &MSG[currentAddress], len);
			currentAddress += len;
			bytesRemaining -= len;
		}

		return len;
	}

	return 0;
}
```

Es importante recalcar que ambas funciones deben de ser previamente definidas por V-USB y para ello hay que activar dos macros que haran disponible la cabecera para su implementación, `USB_CFG_IMPLEMENT_FN_WRITE` y `USB_CFG_IMPLEMENT_FN_READ`.

## usbconfig.h

El fichero *usbconfig.h* proviene del proyecto V-USB, es obligatorio incluirlo ya que contiene toda la configuración del dispositivo a emular. Cuando comenzamos la fase de codificación tuvimos que refactorizarlo de forma sustancial, ya que dependiendo de los valores definidos se creará un dispositivo diferente.

Este fichero contiene numerosas macros que controlan diversos aspectos de la configuración hardware, ya que como comentamos en otro punto de esta memoria, V-USB soporta muchos tipos de microcontroladores y es ampliamente configurable, pero en esta sección repasaremos algunos de los más importantes.

Un conector USB sencillo tiene 4 líneas, 2 encargadas de la alimentación y otras dos de los datos. En este fichero se definen el puerto y los pines del microcontrolador responsables de este control.

- `USB_CFG_IOPORTNAME `, este parámetro define el puerto al que se conectan las líneas de datos.

- `USB_CFG_DMINUS_BIT`, este parámetro define el pin del microcontrolador donde está conectada la línea negativa del conector.

- `USB_CFG_DPLUS_BIT`, en este se define el pin donde se conecta la línea positiva del conector.

```C
#define USB_CFG_IOPORTNAME      D
#define USB_CFG_DMINUS_BIT      4
#define USB_CFG_DPLUS_BIT       2
```

La frecuencia a la que trabaja el microcontrolador también es un parámetro clave, ya que esta implementación de USB es emulada por software la frecuencia debe de ser muy estricta para cumplir con los tiempos de la comunicación. Se define en la macro `USB_CFG_CLOCK_KHZ`.

Como hemos visto anteriormente, el dispositivo que hemos implementado pertenece a la clase HID, que debe de cumplir algunos requisitos concretos. Estos requisitos los definimos en las siguientes macros:

- `USB_CFG_HAVE_INTRIN_ENDPOINT`, esta macro nos permite activar un endpoint de tipo INTERRUPT IN, necesario en HID.

- `USB_CFG_INTERFACE_CLASS` , otro punto muy importante es definir la clase del dispositivo. Como en nuestro caso se trata de HID, esta macro llevará el valor 3.

  ```C
  #define USB_CFG_HAVE_INTRIN_ENDPOINT    1
  /* 
  Define this to 1 if you want to compile a version with two endpoints: 
  The default control endpoint 0 and an interrupt-in endpoint ...
  */
  #define USB_CFG_INTERFACE_CLASS         3
  /* 
  See USB specification ...
   - HID class is 3, no subclass and protocol required (but may be useful!)
  ...
  */
  ```
  
  

Otros dos valores muy importantes en la implementación de un dispositivo USB son el ProductID y VendorID, ya que sin ellos sería imposible asociar un dispositivo con un driver. Por ello, dentro de este fichero podemos definirlos bajo las macros: `USB_CFG_VENDOR_ID` y `USB_CFG_DEVICE_ID`.

Por último, otros dos valores interesantes de configurar son el nombre del fabricante y del dispositivo, que posteriormente serán mostrados una vez el host reconozca el dispositivo. Podemos hacer esto dándole valor a las siguientes macros:  `USB_CFG_VENDOR_NAME` y `USB_CFG_DEVICE_NAME`.



```C
/* -------------------------- Device Description --------------------------- */
#define  USB_CFG_VENDOR_ID      0xA0, 0x20
#define  USB_CFG_DEVICE_ID      0xe5, 0x41 
#define USB_CFG_VENDOR_NAME     'x', 'G', 'G', 'C'
#define USB_CFG_DEVICE_NAME     'p', 'w', 'n', 'e', 'd', 'D', 'e', 'v', 'i', 'c', 'e'
```



##  utils.h

En este fichero se implementan diferentes funciones que nos permiten controlar los componentes más básicos como pueden ser los diodos leds y el buzzer. 

Aquí también se trabaja con los timers que nos ofrece el hardware en las funciones `blinkPWM()` y `harwarePWMBeep()`, que son las encargadas de empezar a parpadear el led con frecuencia de 1 segundo y de generar una señal PWM con la frecuencia indicada por parámetro, respectivamente.

## Otras librerías usadas

Durante el proyecto, hemos usado también dos librerías de la comunidad que nos han permitido integrar en el software tanto el anillo de leds como el display de tipo OLED.

Estas librerías son:

- `oled/`, se encarga de controlar el display OLED mediante el protocolo de comunicación *i2c* que solo usa dos líneas de datos para la transmisión de la información. [@wagiminator-profile]
- `light_ws2812/`, esta librería implementa el protocolo *one-wire* que nos permite controlar el anillo de led. [@ws2812-library]

## Depuración del código del microcontrolador {#sec:debug}

La depuración en microcontroladores de este tipo puede llegar a ser todo un reto. Tenemos que tener en cuenta que un chip de esta clase no es más que una diminuta compleja pieza de hardware que ejecuta el código que previamente le ha sido cargado en una memoria flash, siendo imposible trazarlo como se podría hacer en otro entorno más depurable como puede ser *GDB*.

Por ello, hay que buscar maneras auxiliares de depuración. En los inicios utilizabamos el parpadeo de un LED para indicar que el procesador estaba ejecutando ciertas partes del código, pero por temas de temporización esta manera no era del todo efectiva, además de la poca información que daba.

Con el cambio de chip al *ATMega328p* pudimos hacer uso de la interfaz UART que lleva la placa que elegimos y establecer una línea de comunicación por puerto serie con el dispositivo. V-USB nos ofrece una interfaz de depuración simple pero efectiva en el fichero `oddebug.c` con la que podemos aprovechar este hardware, esencialmente pone a funcionar un dispositivo serie por el que posteriormente expondremos cadenas de caracteres que usaremos para debuggear el código.

A este fichero añadimos una función que nos resultó muy cómoda para poder imprimir cadenas de caracteres y es la función `int odPrintf(char *fmt, ...)`. De esta manera, pudimos incluir trazas distribuidas por el código para ver los diferentes flujos de ejecución, así como el valor de determinadas variables, de manera muy sencilla como muestra el siguiente código:

```C
odDebugInit();
odPrintf("V-USB device initializing... ");
...
odPrintf("OK!\n");

```



