<!-- Leave a blank line before the title -->

# ATmega328P: el microcontrolador elegido para el proyecto

En este capítulo se describen los aspectos técnicos de este microcontrolador, que es el que se ha optado finalmente por utilizar en el proyecto. Aunque en el capítulo 2 se ha hablado del ATTiny85, éste microcontrolador pesentaba dificultades a la hora de utilizar determinados pines con los periféricos, debido a la escasez de éstos.


## Características técnicas del ATmega328P

El Atmega328P de 8 bits es un circuito integrado de alto rendimiento que está basado un microcontrolador RISC, combinando 32 KB de memoria flash con capacidad *read-write* al mismo tiempo. 

![Chip ATmega](img/ATMEGA328P.jpg){width=55%}

Tiene 1 KB de memoria EEPROM, 2 KB de SRAM, 23 líneas de E/S de propósito general, 32 registros de proceso general, tres temporizadores flexibles/contadores con modo de comparación, interrupciones internas y externas, programador de modo USART, una interfaz serial orientada a byte de 2 cables, SPI puerto serial, 6-canales 10-bit Conversor A/D (canales en TQFP y QFN/MLF packages), temporizador "watchdog" programable con oscilador interno, y cinco modos de ahorro de energía seleccionables por software. El dispositivo opera entre 1.8 y 5.5 voltios. Puede alcanzar una respuesta de 1 MIPS, balanceando consumo de energía y velocidad de proceso. [@nano-pinout]

![Placa Arduino NANO con ATmega328p](img/atmega_board.png){width=40%}


## Depuración con ATmega328P

Para poder realizar una depuración del firmware que se flashee en la placa, se ha implementado una función a modo de *printk()* para poder imprimir por una consola UART mensajes de depuración. 

Una de las principales limitaciones del ATTiny85 era que únicamente tenía un puerto, el PORTB, por lo que dificultaba mucho programar otros pines para otros usos como en este caso, para una salida por puerto serie.

En el firmware se ha añadido una nueva biblioteca, ```oddebug.h```, que es la que tiene implementadas las siguientes funciones para imprimir por la consola de la UART los mensajes que deseemos:

```C
int odPrintf(char *fmt, ...) {
    va_list ap;
    char str[256];
    int n;

    va_start(ap, fmt);
    vsnprintf(str, 256, fmt, ap);
    uartPutStr(str);  // print char one by one
    va_end(ap);
    return 0;
}
```
Esta función recibe un array de char, lo almacena y elabora el string correspondiente (con el final de línea ```\0 ``` ) para que se vaya imprimiento caracter a caracter con la siguiente función: 

```C
static void uartPutStr(char *c)
{
	while (*c != '\0')
		uartPutc(*c++);
}
```
Se recorre en un bucle todo el string hasta que se llega al final. Para llamar a la función desde nuesto ```main()  ```, se hace de forma similar a la función ```printf() ```:

```C
odPrintf("v-usb device main\n");
```