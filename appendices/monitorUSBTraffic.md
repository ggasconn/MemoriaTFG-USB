<!-- Leave a blank line before the title -->

# Monitorizar el tráfico USB usando Wireshark

En este anexo se explica cómo capturar y mostrar el tráfico USB con la interfaz GUI de Wireshark que hace que sea muy fácil interactuar con ella.

Todo el software y las pruebas se han realizado bajo `Ubuntu 20.04.5 LTS` con el kernel`5.15.0-48-generic`.

## Software

Todos los paquetes necesarios se pueden instalar desde los repositorios de Ubuntu usando `apt`.

``` bash
sudo apt install wireshark libpcap0.8
```

Además, el módulo `usbmon` debe estar habilitado para acceder al tráfico usb.

```bash	
sudo modprobe usbmon
```

## Abrimos Wireshark

Como vamos a acceder al hardware físico, debemos abrir Wireshark como superusuario. Para ello, procedemos a lanzarlo desde la terminal de la siguiente manera:

```bash
sudo wireshark
```

## Seleccionamos la interfaz usbmonX correcta

Una vez que hayamos abierto Wireshark, veremos algunas interfaces *usbmon* desde las que podemos monitorear. Para seleccionar la interfaz correcta, debemos verificar a qué bus está conectado nuestro dispositivo USB. Podemos usar `lsusb` para esa tarea.

```bash
$ lsusb
Bus 001 Device 067: ID 16c0:05df Van Ooijen Technische Informatica HID device
```

En este caso, elegiremos *usbmon1* ya que nuestro dispositivo está conectado al bus 001. Además, podemos ver que el *DeviceID* es el número 67, esto será útil más adelante para el filtro de visualización.

## Empezamos a capturar tráfico

Para comenzar a capturar tráfico, solo necesitamos hacer doble clic en la interfaz *usbmon* correcta y automáticamente comenzará a mostrar líneas con los paquetes en tránsito. La mayor parte del tráfico que se muestra no nos será útil, por lo que lo mejor ahora es aplicar algunos filtros para mostrar solo el tráfico relacionado con nuestro dispositivo USB.

![Captura mostrando el tráfico USB de Wireshark capturado](img/wireshark.png)

## Filtrar el tráfico mostrado

Como se ha mencionado previamente, la mejor manera de mostrar el tráfico que nos interesa es usar filtros de visualización.

La sintaxis es muy simple, por lo que podemos escribirla nosotros mismos o hacer clic derecho en un paquete desde nuestro dispositivo y prepararlo como un filtro. En la línea de filtros, aplicamos lo siguiente:

```
usb.src == "1.67.0" || usb.dst == "1.67.0"
```

Donde el primer número corresponde al bus, el segundo al *device ID* y el tercero al *endpoint ID*.

![Ejemplo de un filtro aplicado](img/filter.png)