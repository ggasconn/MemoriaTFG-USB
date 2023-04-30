<!-- Leave a blank line before the title -->

# Desarrollo del prototipo hardware

En este capítulo se describe con detalles de bajo nivel los periféricos y hardware utilizado para el desarrollo del proyecto.




## Microcontroladores

En el desarrollo del proyecto, se han utilizado dos microcontroladores sobre los que se han probado el firmware desarrollado. A continuación, se explica cada uno de ellos, así como una explicación general sobre el origen de esta familia de microcontroladores.



### Familia de microcontroladores AVR

La familia de microcontroladores AVR provienen del fabricante Atmel, diseñados para el desarrollo de sistemas empotrados y la ejecución de aplicaciones relacionados con robótica y una gran variedad de usos dentro del sector industrial.

Esta familia de microcontroladores AVR están caracterizados por su bajo consumo energético, además de tener una alta velocidad de procesamiento y una interfaz de programación relativamente fácil de usar. Cuentan con una arquitectura RISC, permitiendo un procesamiento más rápido y un uso eficiente de los recursos. [@avr-products]

Los microcontroladores AVR suelen tener una memoria flash para el almacenamiento de programas, SRAM para el almacenamiento de datos y una gran variedad de periféricos como temporizadores, UART y ADC para interactuar con dispositivos externos. Muchos de estos microcontroladores son usados en Arduino para el desarrollo de proyectos basados en la interfaz USB. [@avr-info]



### ATTiny85

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



### ATmega328p

El Atmega328P de 8 bits es un circuito integrado de alto rendimiento que está basado un microcontrolador RISC, combinando 32 KB de memoria flash con capacidad *read-write* al mismo tiempo. 

![Chip ATmega](img/ATMEGA328P.jpg){width=55%}

Tiene 1 KB de memoria EEPROM, 2 KB de SRAM, 23 líneas de E/S de propósito general, 32 registros de proceso general, tres temporizadores flexibles/contadores con modo de comparación, interrupciones internas y externas, programador de modo USART, una interfaz serial orientada a byte de 2 cables, SPI puerto serial, 6-canales 10-bit Conversor A/D (canales en TQFP y QFN/MLF packages), temporizador "watchdog" programable con oscilador interno, y cinco modos de ahorro de energía seleccionables por software. El dispositivo opera entre 1.8 y 5.5 voltios. Puede alcanzar una respuesta de 1 MIPS, balanceando consumo de energía y velocidad de proceso. [@nano-pinout]

![Placa Arduino NANO con ATmega328p](img/atmega_board.png){width=40%}





## Placas que utilizan estos microcontroladores

A continuación se detallan las dos placas sobre las que se han hecho pruebas en este proyecto.



### Digispark con ATTiny85

La placa Digispark cuenta con una interfaz USB, necesaria para este proyecto.
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



### NANO con ATmega328p y puerto serie

Arduino Nano se utiliza para desarrollar proyectos en los que se requiere un tamaño reducido, bajo consumo de energía y una amplia gama de opciones de E/S. Está basada en el microcontrolador ATmega328p, en el que destaca su asequibilidad y facilidad de uso.

El chip ATmega328p es un microcontrolador AVR de 8 bits que funciona hasta a 16 MHz con 32 KB de memoria flash, 2 KB de SRAM y 1 KB de EEPROM. También incluye diversos periféricos, como convertidores analógico-digitales, salidas de modulación por ancho de pulsos (PWM) e interfaces de comunicación serie.

![Pinout de la placa NANO con ATmega328p](img/nanopinout.png){width=50%}



La placa Nano tiene un formato compacto de tamaño 45 mm x 18 mm), lo que resulta idóneo para su uso en proyectos pequeños que requieren portabilidad y ahorro de espacio. A pesar de su reducido tamaño, cuenta con una gran variedad de opciones de E/S, incluidos 14 pines de entrada/salida digital, 8 pines de entrada analógica y 6 pines PWM. Cuenta también con un puerto USB para programación y alimentación, y un botón de reset para reiniciar la placa. [@nano-info]

Una de las características más atractivas de la placa Nano es su compatibilidad con Arduino IDE, aunque nosotros desarrollamos y cargamos el firmware directamente sin utilizar este entorno, como se explica en otro apartados. 

La placa Nano se utiliza, en general, para una amplia gama de proyectos, incluyendo robótica, automatización, registro de datos y aplicaciones de sensores. También es una opción para la creación de prototipos y pruebas, ya que se puede integrar fácilmente en circuitos breadboard o perfboard. En este proyecto resulta realmente útil, comparando con Digispark, ya que el número de pines de entrada y salida es notablemente mayor que la anterior, lo que resulta útil para realizar pruebas con varios periféricos conectados a la vez.





## Periféricos con soporte en el proyecto

COMPLETAR: placa de expansión genérica, integra algunos periféricos y permite la conexión de otros. 



### Anillo de LEDs de colores

Para probar el firmware, en la primera versión se ha hecho uso de la tira de leds circular fabricado por la empresa NeoPixel [@LED-strip]. Dichos leds se encuentran en disposición circular con colores RGB direccionables individualmente.

![Tira de LEDs circular de NeoPixel sobre una placa Arduino](img/LED-Strip.jpg){width=45% #fig:label3}

Estos LEDs están conectados en cadena y cada LED está controlado por una sola línea de datos. La línea de datos usa un protocolo de comunicación específico llamado *One-Wire Protocol*, que se implementa a través de software y se usa para controlar el color y el brillo de cada LED individual.

La tira de LED circular se alimenta con una fuente de alimentación de 5 V que proporciona la placa Digispark. Cada LED consume una cierta cantidad de energía, y el consumo total de energía de la tira depende de la cantidad total de LEDs y la configuración de brillo.

La tira de LEDs circular generalmente está controlada por un microcontrolador, en nuestro caso el ATTiny85 se encarga de ello, aunque también existen interfaces de comunicación para estas tiras de LEDs especializado, que envía los datos de color y brillo a los LED mediante el protocolo *One-Wire*. El microcontrolador se puede programar para crear una amplia variedad de efectos de iluminación, incluidos ciclos de color, animaciones y patrones.



#### Smart Shift Register en NeoPixel LED

Cada LED individual en la tira de LEDs circular de NeoPixel contiene un registro de cambio inteligente, con un pequeño microcontrolador y un LED RGB (Red, Green, Blue). El microcontrolador es responsable de controlar el color y el brillo de los LED y comunicarse con los otros LED de la cadena.

![Disposición de los pines y del controlador en cada LED](img/pines_neopixel.jpg){width=50% #fig:label3}

El Smart Shift Register LED utiliza un protocolo de comunicación específico llamado *One-Wire Protocol* para recibir datos del microcontrolador. Dicho protocolo se basa en otro protocolo serie de bajo nivel que utiliza una sola línea de datos para transmitirlos entre el microcontrolador y los LED.

Para enviar datos al LED Smart Shift Register, el microcontrolador envía una serie de pulsos en la línea de datos, que el LED interpreta como una secuencia de comandos. Los comandos incluyen instrucciones para configurar el color y el brillo de cada LED individual, así como para configurar el tiempo y la sincronización de la cadena de LEDs.

El LED de registro de desplazamiento inteligente también incluye un conjunto de registros de desplazamiento, que se utilizan para almacenar los datos de color y brillo de cada LED de la cadena. Los registros de desplazamiento permiten que el LED de registro de desplazamiento inteligente reciba y almacene datos del microcontrolador, y luego pase esos datos al siguiente LED de la cadena.

Además de los registros de desplazamiento, el Smart Shift Register LED también incluye un conjunto de controladores de corriente constante, que se utilizan para controlar el brillo de cada LED individual. Los controladores de corriente aseguran que cada LED reciba una cantidad constante de energía, independientemente del número de LEDs en la cadena o la configuración de brillo. [@smsreg-video]



#### Protocolo One-Wire

El protocolo *One-Wire* es un protocolo de comunicación serie de bajo nivel que se utiliza para comunicarse con dispositivos que tienen una sola línea de datos. Fue desarrollado por *Dallas Semiconductor* en la década de los 90 y ahora se usa en una amplia gama de aplicaciones, incluidos sensores de temperatura, tiras de LED y EEPROM.

Dicho protocolo está diseñado para ser lo más simple, confiable y con bajo costo. Utiliza una sola línea de datos, que es bidireccional y normalmente está conectada al microcontrolador. La línea de datos también está conectada a uno o más dispositivos *One-Wire*, que suelen ser sensores u otros dispositivos de bajo consumo.

![Animación sobre la disposición de los bits dentro del controlador de cada LED](img/onewire.png){width=70% #fig:label3}

Para comunicarse con un dispositivo *One-Wire*, el microcontrolador envía una serie de pulsos en la línea de datos, que el dispositivo interpreta como comandos. Los pulsos se generan utilizando una secuencia de tiempo específica, que incluye una combinación de niveles de voltaje alto y bajo en la línea de datos.

El protocolo *One Wire* utiliza una variedad de comandos diferentes para comunicarse con el dispositivo, incluidos comandos para leer y escribir datos, establecer opciones de configuración y enviar señales de control. Estos comandos se envían en una secuencia específica y se cronometran de acuerdo con las especificaciones del protocolo.

Una de las características más importantes es que incluye un mecanismo CRC (comprobación de redundancia cíclica) incorporado, que se utiliza para garantizar la integridad de los datos que se transmiten. El mecanismo CRC utiliza un algoritmo matemático para generar una suma de verificación para los datos, que luego se transmite junto a ellos. El dispositivo receptor puede usar el algoritmo para generar la suma de verificación para los datos recibidos, y compararla con la suma de verificación transmitida para verificar que los datos se hayan transmitido correctamente.



### Display 7 segmentos

COMPLETAR



### Pantalla LCD OLED

Otro de los periféricos usado para el desarrollo del firmware, es una pantalla LCD tipo OLED, en el que se muestran cadenas de texto. 

![Pantalla OLED utilizada en el proyecto](img/LCD_OLED.png){width=40%}

Para su implementación en el firmware del dispositivo, se hace uso de la biblioteca ```oled.h```. El cambio fundamental es en la función ```uchar usbFunctionWrite(uchar *data, uchar len)```  en el que se ha añadido un nuevo report ID (el reportID 4) que hace uso de esta librería para mostrar una cadena de texto guardada en el array ```uchar *data``` que le llega por parámetro a la función.

```C
} else if (reportId == 4) {
    display7sSet(data[1]);

    return 1;
}
```

En la función del firmware ```uchar usbFunctionRead(uchar *data, uchar len)``` no se ha implementado ninguna condición ya que no se lee ningún dato a través de la pantalla.

La función que se encarga de inicializar el dispositivo ```usbMsgLen_t usbFunctionSetup(uchar data[8])``` hace lo siguiente:
```C
 } else if (reportId == 4) {

    bytesRemaining = 1;
    currentAddress = 0 ;

    return USB_NO_MSG;

 }
```

En el que retornando la macro ```USB_NO_MSG``` indicamos al firmware que debe ejecutarse la función de escritura mencionada previamente con el *reportID* correspondiente (La función de Setup es la primera que se ejecutará nada más encender el dispositivo). 

Para hacer uso de la pantalla, se hace uso de la librería ```libusb```, por lo que en el main declaramos algo como lo siguiente (se obvia la comprobación de errores para que el código no sea extenso):
```C
#define MSG "Hello, World! :) Sent from the computer"

int main() {
    libusb_device_handle *handle;
    libusb_init(NULL);

    // Try to find and open device with PID:VID
    handle = libusb_open_device_with_vid_pid(NULL, 0x20a0, 0x41e5);
	// Check errors
    
    result = libusb_detach_kernel_driver(handle, 0);
	// Check errors
    
    result = libusb_claim_interface(handle, 0);
	// Check errors

    setText(handle, 0x4);

    libusb_release_interface(handle, 0);
    libusb_close(handle);
    libusb_exit(NULL);

    return 0;
}
```
La función ```setText()``` ejecuta el siguiente código:

```C
void setText(libusb_device_handle *handle, unsigned int reportID) {
    int res = -1;
    unsigned char data[41];

    data[0] = reportID; //0x4

    for (int i = 1; i < 40; i++)
        data[i] = MSG[i - 1];
    
    res = libusb_control_transfer(handle, // device
                                    LIBUSB_REQUEST_TYPE_CLASS | 
                                  LIBUSB_RECIPIENT_INTERFACE |
                                  LIBUSB_ENDPOINT_OUT, // bmRequestType
                                    0x9,  // bRequest 0x1 -> GET  0x9 -> SET
                                    0x0003, // wValue
                                    0, // wIndex
                                    data, // data
                                    41, // wLength
                                    5000); // timeout

    if (res < 0)
        printf(">>> [ERROR]: Cannot send URB! Code: %d\n", res);
}
```

En esta función, llama a la función ```libusb_control_transfer``` utilizando una petición SET al *report ID* que le llega por parámetro y que en el main habíamos establecido en 0x4. Cabe destacar que el array ```data[]``` contiene el mensaje y en la posición 0 el *report ID*, por lo que en el primer bucle for se adapta el array para que en la primera posición esté el 0x4. 



### Sensor de temperatura

Completar



### Placa Bee 2.0

COMPLETAR: Relación con LIN