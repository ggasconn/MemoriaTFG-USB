<!-- Leave a blank line before the title -->

# Periféricos utilizados para el desarrollo del firmware

En este capítulo se describe con detalles de bajo nivel los periféricos utilizados en el firmware desarrollado.




## Anillo de LEDs de colores

Para probar el firmware, en la primera versión se ha hecho uso de la tira de leds circular fabricado por la empresa NeoPixel. Dichos leds se encuentran en disposición circular con colores RGB direccionables individualmente.

![Tira de LEDs circular de NeoPixel sobre una placa Arduino](img/LED-Strip.jpg){width=45% #fig:label3}

Estos LEDs están conectados en cadena y cada LED está controlado por una sola línea de datos. La línea de datos usa un protocolo de comunicación específico llamado *One-Wire Protocol*, que se implementa a través de software y se usa para controlar el color y el brillo de cada LED individual.

La tira de LED circular se alimenta con una fuente de alimentación de 5 V que proporciona la placa Digispark. Cada LED consume una cierta cantidad de energía, y el consumo total de energía de la tira depende de la cantidad total de LEDs y la configuración de brillo.

La tira de LEDs circular generalmente está controlada por un microcontrolador, en nuestro caso el ATTiny85 se encarga de ello, aunque también existen interfaces de comunicación para estas tiras de LEDs especializado, que envía los datos de color y brillo a los LED mediante el protocolo *One-Wire*. El microcontrolador se puede programar para crear una amplia variedad de efectos de iluminación, incluidos ciclos de color, animaciones y patrones.

### Smart Sift Register en NeoPixel LED

Cada LED individual en la tira de LEDs circular de NeoPixel contiene un registro de cambio inteligente, con un pequeño microcontrolador y un LED RGB (Red, Green, Blue). El microcontrolador es responsable de controlar el color y el brillo de los LED y comunicarse con los otros LED de la cadena.

![Disposición de los pines y del controlador en cada LED](img/pines_neopixel.jpg){width=50% #fig:label3}

El Smart Shift Register LED utiliza un protocolo de comunicación específico llamado *One-Wire Protocol* para recibir datos del microcontrolador. Dicho protocolo se basa en otro protocolo serie de bajo nivel que utiliza una sola línea de datos para transmitirlos entre el microcontrolador y los LED.

Para enviar datos al LED Smart Shift Register, el microcontrolador envía una serie de pulsos en la línea de datos, que el LED interpreta como una secuencia de comandos. Los comandos incluyen instrucciones para configurar el color y el brillo de cada LED individual, así como para configurar el tiempo y la sincronización de la cadena de LEDs.

El LED de registro de desplazamiento inteligente también incluye un conjunto de registros de desplazamiento, que se utilizan para almacenar los datos de color y brillo de cada LED de la cadena. Los registros de desplazamiento permiten que el LED de registro de desplazamiento inteligente reciba y almacene datos del microcontrolador, y luego pase esos datos al siguiente LED de la cadena.

Además de los registros de desplazamiento, el Smart Shift Register LED también incluye un conjunto de controladores de corriente constante, que se utilizan para controlar el brillo de cada LED individual. Los controladores de corriente aseguran que cada LED reciba una cantidad constante de energía, independientemente del número de LEDs en la cadena o la configuración de brillo.



! Referencia al vídeo https://www.youtube.com/watch?v=PPVi3bI7_Z4

### Protocolo One-Wire

El protocolo *One-Wire* es un protocolo de comunicación serie de bajo nivel que se utiliza para comunicarse con dispositivos que tienen una sola línea de datos. Fue desarrollado por *Dallas Semiconductor* en la década de los 90 y ahora se usa en una amplia gama de aplicaciones, incluidos sensores de temperatura, tiras de LED y EEPROM.

Dicho protocolo está diseñado para ser lo más simple, confiable y con bajo costo. Utiliza una sola línea de datos, que es bidireccional y normalmente está conectada al microcontrolador. La línea de datos también está conectada a uno o más dispositivos *One-Wire*, que suelen ser sensores u otros dispositivos de bajo consumo.

![Animación sobre la disposición de los bits dentro del controlador de cada LED](img/onewire.png){width=70% #fig:label3}

Para comunicarse con un dispositivo *One-Wire*, el microcontrolador envía una serie de pulsos en la línea de datos, que el dispositivo interpreta como comandos. Los pulsos se generan utilizando una secuencia de tiempo específica, que incluye una combinación de niveles de voltaje alto y bajo en la línea de datos.

El protocolo *One Wire* utiliza una variedad de comandos diferentes para comunicarse con el dispositivo, incluidos comandos para leer y escribir datos, establecer opciones de configuración y enviar señales de control. Estos comandos se envían en una secuencia específica y se cronometran de acuerdo con las especificaciones del protocolo.

Una de las características más importantes es que incluye un mecanismo CRC (comprobación de redundancia cíclica) incorporado, que se utiliza para garantizar la integridad de los datos que se transmiten. El mecanismo CRC utiliza un algoritmo matemático para generar una suma de verificación para los datos, que luego se transmite junto a ellos. El dispositivo receptor puede usar el algoritmo para generar la suma de verificación para los datos recibidos, y compararla con la suma de verificación transmitida para verificar que los datos se hayan transmitido correctamente.



## Pantalla LCD OLED






## Sensor de temperatura

