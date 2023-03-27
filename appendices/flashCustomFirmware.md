<!-- Leave a blank line before the title -->

# Guía para flashear un firmware propio

La placa *Digispark ATTiny85* tiene de forma predeterminada un bootloader llamado *Micronucleus* que puede comunicarse con la máquina host a través de una comunicación serie, lo que permite al usuario la opción de flashear código a través del puerto USB y evita el uso de un programador *USBASP*.

Esta característica puede ser útil, pero hay ciertos casos en los que es posible que necesitemos actualizar un gestor de arranque personalizado, firmware *C/C++* o cualquier tipo de programa que necesite borrar completamente la ROM, y esto no se puede hacer usando Micronucleus.



## Hardware

* **ICSP Programmer**: lo usaremos para hablar con la ROM integrado en el chip. *USBASP* es el que se va a usar, pero cualquier programador con pines *MISO/MOSI/SCK/RST* funcionará, incluso con un *Arduino* flasheado con el sketch *ArduinoISP*.
* **Digispark ATTiny85**



## Software

* **Arduino IDE**: usaremos las herramientas de *AVR* integradas en el entorno.



## Diagrama de pines

| ISCP Programmer | Digispark ATTiny85 |
| :-------------: | :----------------: |
|      MOSI       |        PB0         |
|      MISO       |        PB1         |
|       SCK       |        PB2         |
|      RESET      |        PB5         |
|       5V        |         5V         |
|       GND       |        GND         |



## Cómo flashear un sketch de Arduino

Lo primero, es hacer click en el menú *Tools* y seleccionar la placa correcta y la CPU. Después, seleccionamos el programador que estamos usando.

![Menú de herramientas de Arduino IDE](img/usbasp.png){width=45%}



Una vez que hayamos establecido los parámetros correctos, sólo queda iniciar el proceso de flasheo del firmware y confiar en que todo funcione correctamente. Si ha ido bien, el entorno nos mostrará un mensaje como el siguiente:

![Mensaje de salida una vez flasheado el firmware](img/flashDoneArduino.png)



## Flashear un firmware propio a partir de un archivo .hex

Actualizar un archivo .hex existente puede ser útil si queremos instalar un *firmware/cargador de arranque* precompilado o cargar un programa *C/C++*.

*Arduino* viene con algunas utilidades *AVR* que podemos usar desde nuestra terminal para compilar y flashear código a *microcontroladores AVR*.

Lo primero es ubicar nuestro directorio *Arduino IDE*, a continuación se muestra una forma de hacerlo:

```bash
$ ls -l `which arduino`
lrwxrwxrwx root root 73 B Tue Jul 12 16:44:01 2022 
/usr/local/bin/arduino ⇒ /home/ggc/arduino-1.8.19/arduino
```

Una vez que sepamos dónde se encuentra, buscamos las siguientes utilidades y archivos:

* ```arduino-1.8.19/hardware/tools/avr/bin/avrdude```
* ```arduino-1.8.19/hardware/tools/avr/etc/avrdude.conf```

El siguiente paso es flashear el archivo .hex usando la línea de comandos con los siguientes argumentos:

* **-C**, archivo de configuración del entorno *Arduino IDE* por defecto
* **-v**, verbose output
* **-p**, microcontrolador
* **-c**, programador
* **-P**, puerto
* **-U**, instrucción. Sintaxis: *memtype*:*op*:*filename*[:*format*]

```bash	
arduino-1.8.19/hardware/tools/avr/bin/avrdude 
-Carduino-1.8.19/hardware/tools/avr/etc/avrdude.conf 
-v -pattiny85 -cusbasp -Pusb -Uflash:w:main.hex:i 
```

Para encontrar más argumentos del programa ```avrdude``` y una explicación más profunda, se recomienda visitar el siguiente enlace: https://www.nongnu.org/avrdude/user-manual/avrdude_3.html

Una vez terminado el proceso, se mostrará un mensaje como el siguiente:

![Comando de confirmación de AVRDUDE una vez terminado el proceso](img/avrDone.png)