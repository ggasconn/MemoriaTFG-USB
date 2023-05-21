<!-- Leave a blank line before the title -->

# Drivers y casos prácticos

En este capítulo se explica un punto muy importante en el proyecto, que es proporcionar a los usuarios de la placa un paquete de drivers que puedan tomar de ejemplo para desarrollar y aprender sobre su funcionamiento. 

Ya que este material tiene una finalidad docente y de aprendizaje, el funcionamiento de los drivers es sencillo, pero no por ello simple de implementar. Se ofrecen 5 drivers los cuales hacen uso de nuevas funciones y tipos de datos de la API que nos proporciona *USB core*. Con estos drivers se pretende que los usuarios puedan probar y experimentar el funcionamiento de las utilidades USB que nos ofrece el kernel Linux, trabajando con ambas APIs y usando diferentes tipos de transferencias, con todo lo que conlleva.

Como se menciona en el párrafo anterior, el paquete de drivers contiene 5 ejemplos diferentes, uno por cada plantilla de funcionamiento del firmware y otro que hace uso de la API asíncrona junto con transferencias tipo *INTERRUPT*. A continuación se describe a modo de introducción la API de USB del kernel de Linux, después se explica con detalle cada uno uno de ellos explicando su funcionamiento y arquitectura interna, con ejemplos de código.



## Drivers USB en el kernel Linux

Un componente muy importante en los drivers USB es la definición de la interfaz `usb_driver` [@apiusb-kernel], que debe implementar cualquier módulo para poder registrarse y administrar controladores que interactuen con periféricos USB. Esta estructura viene definida en `<linux/usb.h>`, y se deben implementar aquellos campos que requieran definir las funciones que interactúen con el dispositivo USB. Lo hemos definido como interfaz ya que muchos de sus campos son punteros a función. Estructuras similares son `file_operations` y `proc_ops`, de las que se hablarán más adelante.

```C
struct usb_driver {
	const char *name;
	int (*probe) (struct usb_interface *intf,
		      const struct usb_device_id *id);
	void (*disconnect) (struct usb_interface *intf);
	int (*unlocked_ioctl) (struct usb_interface *intf, unsigned int code,
			void *buf);
	int (*suspend) (struct usb_interface *intf, pm_message_t message);
	int (*resume) (struct usb_interface *intf);
	int (*reset_resume)(struct usb_interface *intf);

	int (*pre_reset)(struct usb_interface *intf);
	int (*post_reset)(struct usb_interface *intf);

	const struct usb_device_id *id_table;
	const struct attribute_group **dev_groups;

	struct usb_dynids dynids;
	struct usbdrv_wrap drvwrap;
	unsigned int no_dynamic_id:1;
	unsigned int supports_autosuspend:1;
	unsigned int disable_hub_initiated_lpm:1;
	unsigned int soft_unbind:1;
};
```



Como se puede observar en la estructura `struct usb_driver`, hay diversos campos para poder implementar funciones que doten de funcionalidad al driver, o simplemente para configuración. No es necesario implementar todas las entradas, solamente aquellas que se requieran para garantizar el funcionamiento mínimo del dispositivo (como ya ocurre con la gestión de las entradas en `/proc`). Las entradas más importantes que los drivers USB deben implementar son las siguientes:

- `name`: Cadena de caracteres, representa el nombre del driver.

- `id_table`: Tabla de dispositivos compatibles con el driver, definidos mediante un *vendor-ID* y *product-ID*.

- `probe`: Este campo define la función encargada de ejecutarse cuando se detecta un dispositivo compatible con el controlador USB al conectarloal host, se encarga de inicializarlo y prepararlo para su uso. Se le pasa a esta función el descriptor del dispositivo como parámetro de la función, `struct usb_interface *`.

- `disconnect`: Esta función se ejecuta cuando se desconecta el dispositivo USB que previamente ha sido detectado y grestionado por el driver.

  

En nuestros drivers, definimos cuatro campos básicos de la interfaz `usb_driver`, como se muestra a continuación:

```C
static struct usb_driver pwnedDevice_driver = {
	.name =		"pwnedDevice",
	.probe =	pwnedDevice_probe,
	.disconnect =	pwnedDevice_disconnect,
	.id_table =	pwnedDevice_table,
};
```

En la estructura `usb_driver` definimos punteros a función en `.probe` y `.disconnect` para implementar las funciones `pwnedDevice_probe()` y `pwnedDevice_disconnect()`:

```C
/*
 * Invoked when the USB core detects a new
 * pwnedDevicestick device connected to the system.
 */
static int pwnedDevice_probe(struct usb_interface *interface,
		      			const struct usb_device_id *id) {
	struct usb_pwnedDevice *dev;

	/*
 	 * Allocate memory for a usb_pwnedDevice structure.
	 * This structure represents the device state.
	 * The driver assigns a separate structure to each pwnedDevicestick device
 	 *
	 */
	dev = kmalloc(sizeof(struct usb_pwnedDevice), GFP_KERNEL);

    // ...

	/* Initialize the various fields in the usb_pwnedDevice structure */
	kref_init(&dev->kref);
	dev->udev = usb_get_dev(interface_to_usbdev(interface));
	dev->interface = interface;

	/* save our data pointer in this interface device */
	usb_set_intfdata(interface, dev);

	/* we can register the device now, as it is ready */
	retval = usb_register_dev(interface, &pwnedDevice_class);

    // ...
    
	return 0;
}
```

Como hemos mencionado previamente, esta función es la encargada de reservar memoria e inicializar los campos de la estructura del dispositivo. Destacar que a esta función se le pasa el descritor del dispositivo como parámetro, que se utilizará para poder registrarlo posteriormente. La función invocada cuando se desconecta el dispositivo previamente gestionado por el driver es la siguiente:

```C
/*
 * Invoked when a pwnedDevicestick device is 
 * disconnected from the system.
 */
static void pwnedDevice_disconnect(struct usb_interface *interface) {
	struct usb_pwnedDevice *dev;
	int minor = interface->minor;

	dev = usb_get_intfdata(interface);
	usb_set_intfdata(interface, NULL);

	/* give back our minor */
	usb_deregister_dev(interface, &pwnedDevice_class);

	/* prevent more I/O from starting */
	dev->interface = NULL;

	/* decrement our usage count */
	kref_put(&dev->kref, pwnedDevice_delete);

    // ...
}
```

Cabe destacar que en la última función, se decrementa el contador de referencias del dispositivo. Puede llegar a eliminarlo si el valor de este contador es 0. Esta función también es encargada de invalidar la interfaz del dispositivo.

A continuación se muestran las funciones encargadas de la inicialización del módulo `pwnedDevicedrv_module_init()` y de eliminar su registro mediante la función `pwnedDevicedrv_module_cleanup()`. 

```C
/* Module initialization */
int pwnedDevicedrv_module_init(void) {
   return usb_register(&pwnedDevice_driver);
}

/* Module cleanup function */
void pwnedDevicedrv_module_cleanup(void) {
  usb_deregister(&pwnedDevice_driver);
}

module_init(pwnedDevicedrv_module_init);
module_exit(pwnedDevicedrv_module_cleanup);
```

Para llevar a cabo el registro del módulo, se invoca la función `usb_register()`, que acepta como parámetro un puntero a la interfaz `usb_driver`, definida mediante la variable `pwnedDevice_driver`. En el caso de que se produzca correctamente el registro, se devuelve 0. En caso contrario, un número negativo con el código de error. Para la descarga del módulo, se ejecuta `usb_deregister()`, que se encarga de la limpieza del módulo, aceptando un puntero a la interfaz `usb_driver`.

En la instanciación de `usb_driver`, a parte de la definición de los campos explicados previamente, se ha definido otro para los dispositivos compatibles (`.id_table =	pwnedDevice_table`). La implementación de esta tabla es la siguiente:

```C
/* Define these values to match your devices */
#define VENDOR_ID	0X20A0
#define PRODUCT_ID	0X41E5

/* table of devices that work with this driver */
static const struct usb_device_id pwnedDevice_table[] = {
	{ USB_DEVICE(VENDOR_ID,  PRODUCT_ID) },
	{ }					/* Terminating entry */
};

MODULE_DEVICE_TABLE(usb, pwnedDevice_table);
```

La tabla `pwnedDevice_table[]` almacena todos los identificadores de dispositivos `usb_device_id` que el driver puede gestionar. Cada posición del array es un descriptor del dispositivo, que se inicializa mediante la macro `USB_DEVICE()`, cuyos parámetros son el identificador del fabricante `VENDOR_ID` y el identificador del producto `PRODUCT_ID`. Cada vez que un dispositivo se conecta al host con los valores de `VENDOR_ID` y `PRODUCT_ID` definidos anteriormente, se consulta la tabla `pwnedDevice_table[]` que contiene esta información y posteriormente se ejecuta la función `probe()` del driver USB (en nuestro caso `pwnedDevice_probe()`). 



Otro de los campos importantes de un driver es la estructura de estado, ya que con esta estructura se permite la conexión simultánea de varios dispositivos del mismo tipo (asociando a cada uno una estructura que represente su estado). En nuestros drivers, hemos definido la siguiente estructura:

```C
/* Structure to hold all of our device specific stuff */
struct usb_pwnedDevice {
	struct usb_device	*udev;			/* the usb device for this device */
	struct usb_interface	*interface;		/* the interface for this device */
	struct kref		kref;
};
```
A continuación se explican los distintos campos de la estructura:

- `udev`: Es una referencia al descriptor del dispositivo físico USB, que es una estructura de datos utilizada por las funciones principales del USB para transferir datos entre el host y el dispositivo USB. Este puntero se encuentra en la mayoría de las estructuras de estado de los controladores USB y se obtiene durante la ejecución de la función `probe()`, donde se crea e inicializa la estructura de estado del dispositivo.

- `interface`: Es una referencia al descriptor de la interfaz USB que el controlador está gestionando. Se refiere a un único dispositivo lógico. Al igual que el campo `udev`, este puntero se inicializa durante la ejecución de la función `probe()`, y se encuentra en muchas estructuras de estado de los controladores USB.

- `kref`: Es un contador de referencias utilizado para gestionar la estructura de estado. En el kernel, el tipo de datos `struct kref` se utiliza para implementar contadores de referencia de objetos del núcleo. Estos contadores son importantes para determinar cuándo es seguro liberar la memoria de un objeto. El tipo de datos `struct kref` proporciona operaciones seguras (desde el punto de vista de la concurrencia en el kernel) para incrementar y decrementar el contador de referencias. Se inicializa con la función `kref_init()` y se incrementa con `kref_get()` y se decrementa con `kref_put()`. 

  El contador de referencia `kref` en los drivers se utiliza para dos propósitos principales: 

  1.  Gestionar correctamente la memoria de la estructura de estado.
  2.  Resolver problemas de concurrencia asociados con la desconexión física del dispositivo cuando el controlador está en uso. Básicamente, la memoria para el objeto de estado se asigna dinámicamente. Se solicita memoria cuando el dispositivo se conecta al sistema (en la función `probe()`) y, en ese momento, el valor interno del contador es 1. El contador de referencias se incrementa con `kref_get()` en el código del controlador cada vez que un proceso de usuario está utilizando el dispositivo, y se decrementa con `kref_put()` cuando el proceso deja de usarlo. Gracias al correcto manejo del contador, la memoria de la estructura de estado se libera únicamente cuando el contador alcanza el valor cero al decrementarse.

Cabe destacar que sin la ejecución de `usb_set_intfdata(interface, dev)` en la función `pwnedDevice_probe()` mostrada anteriormente, no sería posible recuperar la estructura de estado desde las funciones del driver que implementan operaciones sobre ficheros especiales de caracteres, y por lo tanto no se podrían realizar transferencias de datos entre el dispositivo y el kernel, por ejemplo, en la función `pwnedDevice_write()`. 

Como hemos adelantado en el párrafo anterior, otro de los aspectos importantes en drivers es poder interactuar con el driver desde el espacio de usuario. Para ello, es necesario registrar los dispositivos reconocidos (uno por cada dispositivo) en una clase del Linux Device Model (LDM). Es por ello que USB Core asigna major numbers diferentes para clases distintas, dentro de, por ejemplo, `input`, `usb`, `ttyACM`, `tty_usb`, etc. [@linuxusb-devices].

```C
extern int usb_register_dev(struct usb_interface *intf,
                struct usb_class_driver *class_driver);
```

La clase `usb` en Linux abarca la mayoría de los dispositivos USB. Estos dispositivos se identifican con un número identificativo, el `major number`, cuyo valor es 180. Para utilizar esta clase y su `major number`, el controlador básico del dispositivo USB registra cada dispositivo conectado mediante la función `usb_register_dev()`, que realiza las siguientes acciones:

1. Registra el dispositivo en la clase `usb` y le asigna un número `minor number`.
2. Automáticamente crea un archivo de dispositivo en la ubicación `/dev`, interactuando con el servicio `Udev`.
3. Asigna una interfaz de operaciones (`file_operations`) al archivo de dispositivo, definiendo las funciones que se ejecutarán cuando se realicen operaciones sobre estos ficheros de caracteres (por ejemplo, `ẁrite()` o `read()`).



La estructura `usb_class_driver` se define como sigue:

```C
struct usb_class_driver {
    char *name;
    char *(*devnode)(struct device *dev, umode_t *mode);
    const struct file_operations *fops;
    int minor_base;
};
```

- `name`: Es una cadena de caracteres que contiene el nombre del dispositivo y también se utiliza para codificar el nombre de su archivo especial asociado.

- `devnode`: Es una función que debe ser definida en el controlador y se utiliza para indicar:
  1. La ubicación relativa dentro de `/dev` donde se creará el archivo de dispositivo.
  2. Los permisos que se asignarán al archivo de dispositivo (usando el parámetro `mode`).
  
- `file_operations`: Es un puntero que apunta a las operaciones que el controlador ejecutará cuando un programa acceda al dispositivo. El controlador debe instanciar la estructura `file_operations` e implementar las operaciones correspondientes.

- `minor_base`: Indica el valor inicial en el rango de números `minor_numer` asignados para este controlador.



El siguiente fragmento de código muestra la definición de la estructura global de tipo `struct usb_class_driver`, que se pasa como parámetro a la función `usb_register_dev()` en `pwnedDevice_probe()`:

```C
#define USB_MINOR_BASE    0

/*
 * Operations associated with the character device 
 * exposed by driver
 * 
 */
static const struct file_operations pwnedDevice_fops = {
	.owner =	THIS_MODULE,
	.write =	pwnedDevice_write,	 	/* write() operation on the file */
	.read =		pwnedDevice_read,			/* read() operation on the file */
	.open =		pwnedDevice_open,			/* open() operation on the file */
	.release =	pwnedDevice_release, 		/* close() operation on the file */
};

/* 
 * Return permissions and pattern enabling udev 
 * to create device file names under /dev
 * 
 * For each pwnedDevicestick connected device a character device file
 * named /dev/usb/pwnedDevicestick<N> will be created automatically  
 */
char* set_device_permissions(struct device *dev, umode_t *mode) {
	if (mode)
		(*mode)=0666; /* RW permissions */
 	return kasprintf(GFP_KERNEL, "usb/%s", dev_name(dev)); /* Return formatted string */
}


/*
 * usb class driver info in order to get a minor number from the usb core,
 * and to have the device registered with the driver core
 */
static struct usb_class_driver pwnedDevice_class = {
	.name =		"pwnedDevice%d",  /* Pattern used to create device files */	
	.devnode=	set_device_permissions,	
	.fops =		&pwnedDevice_fops,
	.minor_base =	USB_MINOR_BASE,
};
```

Como puede observarse en la estructura `pwnedDevice_class`, el campo `name` denota un patrón. Según este patrón el nombre de los ficheros de dispositivo serán *pwnedDevice0*, *pwnedDevice1*, *pwnedDevice2*, etc.

El segundo campo, `devnode`, se establece con la dirección de la función `set_device_permissions()` que se define anteriormente en el código. El valor de retorno de esta función indica a `Udev` dónde debe crear el archivo especial de dispositivo dentro de `/dev`. En este caso, la implementación especifica que la ruta será `/dev/usb`. Además, el parámetro de retorno `mode` se utiliza para indicar los permisos del archivo de dispositivo, que en esta implementación se establecen en `666` (permisos de lectura y escritura para todos).

El tercer campo, `fops`, se inicializa con la dirección que nos proporciona la variable `pwnedDevice_fops`. En la inicialización de esta variable, se asocian cuatro operaciones al controlador: `pwnedDevice_write()`, `pwnedDevice_read()`,  `pwnedDevice_open()` y `pwnedDevice_release()`. Estas operaciones se invocan cuando se ejecutan las llamadas al sistema `write()`, `read()``open()` y `close()` desde un programa de usuario en relación a cualquier archivo especial de caracteres asociado al controlador.

Por último, el campo `minor_base` se inicializa con una macro definida al principio (`#define USB_MINOR_BASE    0`). [@blinkjc-description]



## BasicInterrupt - Driver asíncrono con comunicación INTERRUPT IN

### Descripción del funcionamiento 

Este módulo del kernel implementa un driver USB utilizando el endpoint de tipo *INTERRUPT IN* del dispositivo USB, así como la API asíncrona del kernel. [@drivers-linux]

El funcionamiento del módulo es muy sencillo. Al cargar el módulo, se ejecuta la función `pwnedDevice_probe()` del driver, que reserva espacio para las distintas estructuras del driver, utilizadas para la interfaz y los paquetes URBs, así como el descubrimiento de endpoints. 

```C
dev->int_in_urb = usb_alloc_urb(0, GFP_KERNEL); // Memory allocate for URB
```

Una vez cargado en el kernel, expone un dispositivo de caracteres que tiene implementada la operación de lectura. Si lanzamos una llamada `read()`, por ejemplo con `cat`, iniciaremos la rutina que envía el URB de tipo *INTERRUPT IN* al dispositivo pidiéndole que le rellene el buffer enviado con datos.

```C
retval = usb_submit_urb(dev->int_in_urb, GFP_KERNEL);
```

Como se usa la API asíncrona de USB la llamada a `read()` devuelve siempre 0 bytes, ya que no se sabe con exactitud cuándo se va a devolver el *URB* relleno con datos y al ser una función no bloqueante no esperamos a que llegue. El resultado de esta solicitud de datos al dispositivo se retorna a través de la función de callback, que se invoca una vez el URB vuelve desde el dispositivo al host. La cabecera de la función es la siguiente: `pwnedDevice_int_in_callback(struct urb *urb)`.

Como este driver está programado usando la API asíncrona que nos ofrece el kernel, toda la gestión de *URBs*, así como el descubrimiento de endpoints y todo el control de transferencia es gestionado por el driver. A continuación, veremos varios puntos relevantes del código que marcan una diferencia en el uso de esta API y del tipo de transferencia *INTERRUPT*.

### Descubrimiento de endpoints

El descubrimiento de endpoints es de lo primero que se hace cuando un driver ha sido asociado a un dispositivo, esta asociación se hace mediante el *ProductID* y *VendorID*.

Cuando el *USB Core* asigna el driver al dispositivo lo primero que se ejecuta es la función `pwnedDevice_probe()`, que ejecuta todas las tareas necesarias de inicialización, entre ellas el descubrimiento de endpoints.

```C
static int pwnedDevice_probe(struct usb_interface *interface,
		      const struct usb_device_id *id) {
	// ...

    /* Initialize the various fields in the usb_pwnedDevice structure */
    kref_init(&dev->kref);
    dev->udev = usb_get_dev(interface_to_usbdev(interface));
    dev->interface = interface;
    iface_desc = interface->cur_altsetting;

    for (i = 0; i < iface_desc->desc.bNumEndpoints; ++i) {  // Endpoint discover
        endpoint = &iface_desc->endpoint[i].desc;

        if (((endpoint->bEndpointAddress & USB_ENDPOINT_DIR_MASK)
             == USB_DIR_IN)
            && ((endpoint->bmAttributes & USB_ENDPOINT_XFERTYPE_MASK)
                == USB_ENDPOINT_XFER_INT))
            dev->int_in_endpoint = endpoint;
    }

    // ...
}
```

Como argumento de esta función *probe* recibimos una descripción de la interfaz que está siendo conectada con el driver, en ella podemos ver los diferentes tipos de descriptores USB que muestran las características del dispositivo. Dentro de estos descriptores podemos llegar a los descriptores de interfaces donde buscaremos un endpoint de tipo *INTERRUPT* y en sentido *IN*, en caso de no encontrarse el driver no puede funcionar y se aborta la inicialización.

### Reserva de URBs

Dentro de la función de *probe* que comentaba la sección anterior, también se realiza la reserva de memoria para el *URB* que se va a enviar al dispositivo para ser rellenado con información.

```C
// Endpoint discover

/* Request IN URB */
dev->int_in_urb = usb_alloc_urb(0, GFP_KERNEL);

/* save our data pointer in this interface device */
usb_set_intfdata(interface, dev);

/* we can register the device now, as it is ready */
retval = usb_register_dev(interface, &pwnedDevice_class);
```

Esta acción se hace a través de la función `usb_alloc_urb()` que devuelve un puntero al trozo de memoria que se ha destinado al *URB* solicitado. Este trozo de memoria será reutilizado y compartido con la controladora USB durante toda la vida del driver. Cabe destacar que es único por dispositivo, ya que el driver soporta varios dispositivos a la vez, cada uno de ellos reservará su propio *URB*.

### Montaje de un URB

Cuando el driver recibe una operación de lectura, `read()`, en la callback de lectura `pwnedDevice_read()` se rellena el URB reservado previamente con información y se lleva a la controladora USB para que sea enviado al dispositivo.

```C
/* Send URB. */
usb_fill_int_urb(dev->int_in_urb, dev->udev,
                 usb_rcvintpipe(dev->udev,
                                dev->int_in_endpoint->bEndpointAddress),
                 dev->int_in_buffer,
                 le16_to_cpu(0x0008),
                 pwnedDevice_int_in_callback,
                 dev,
                 dev->int_in_endpoint->bInterval);
```

Los datos introducidos al URB mediante la función que nos aporta el kernel `usb_fill_int_urb()`, a la cual le mandamos la información que queremos hacerle llegar, así como diferentes parámetros obligatorios y un buffer que rellenará el dispositivo con los datos que le solicitamos.

```C
retval = usb_submit_urb(dev->int_in_urb, GFP_KERNEL);
```

Una vez todos los datos están correctos, tenemos que llamar a la función `usb_submit_urb()` para decirle a la controladora que ese URB está listo para ser enviado y proceder a ello.

### Gestión de la respuesta

Al usarse la API asíncrona después del envío del driver no hay ningún bloqueo que sirva de espera a la respuesta del dispositivo, es por esto que la respuesta se gestiona desde una función externa que es llamada cuando el URB vuelve relleno. Esta función es `pwnedDevice_int_in_callback()`.

Dentro de la función se hacen dos cosas muy sencillas, lo primero es comprobar el estado de la transferencia ya que en caso de haberse producido algún fallo los datos devueltos puede que no sean coherentes. En caso de que todo haya ido bien, se extraen los datos del buffer que previamente se reservó para rellenarse con información por el dispositivo, y se muestra por la salida de información del kernel usando `printk()`.

```C
/*
* Callback function. Executed when INT IN URB returns back with data.
*/
static void pwnedDevice_int_in_callback(struct urb *urb) {
	struct usb_pwnedDevice *dev = urb->context;

	if (urb->status) {  // Status of transfer
		if (urb->status == -ENOENT ||
		    urb->status == -ECONNRESET ||
		    urb->status == -ESHUTDOWN) {
			printk(KERN_INFO "Error on callback!");
			return;
		}
	}

	if (urb->actual_length > 0)
		printk(KERN_INFO "Transfered data: %s\n", dev->int_in_buffer);	
}
```

Este tipo de dispositivo tiene la limitación de que el tamaño máximo de transferencia mediante *INTERRUPT* es de 8 bytes, esta limitación la impone el estándar *USB* para dispositivos *Low-Speed.*



## DisplaysDriver

Este driver ha sido desarrollado para interactuar con los periféricos que muestran al usuario información a través de pantallas. En este momento están soportados por el firmware el display de 7 segmentos y la pantalla OLED, por lo que el driver es capaz de enviar información a ambos.

El driver utiliza la API síncrona que nos ofrece *USB Core* y todas las transferencias van sobre el tipo *CONTROL*.

Una vez cargado el módulo que hace de driver en el kernel, este expone un dispositivo de caracteres bajo la ruta `/dev/usb/displaysx`, donde *x* hace referencia al índice del dispositivo que se encuentra conectado, ya que el driver soporta más de un dispositivo a la vez.

### Operación de lectura

Cuando el dispositivo recibe una llamada `read()`, independientemente del origen, se ejecuta la callback `displays_read()`. Como la plantilla *DISPLAYS* no puede ofrecer ninguna información útil de lectura sobre sus periféricos, cuando una operación de este tipo llega al dispositivo devuelve una frase de 33 bytes a modo de respuesta.

```C
static ssize_t displays_read(struct file *file, char *user_buffer,
			  				size_t count, loff_t *ppos) {
    // ...
    
    unsigned char* message;	
    
    // ...

    wValue = 0x1;
    msgSize = 33;

    retval = usb_control_msg_recv(dev->udev,	
                0, 
                USB_REQ_CLEAR_FEATURE, 
                USB_DIR_IN | USB_TYPE_CLASS | USB_RECIP_DEVICE,
                wValue,	/* wValue = Descriptor index */
                0x0, 	/* wIndex = Endpoint # */
                message,	/* Pointer to the message */ 
                msgSize, /* message's size in bytes */
                0, /* Dale lo necesario*/
                GFP_DMA);

    message[33]='\n';
    message[34]='\0';
    
    // ...
}
```

```bash
~$ cat /dev/usb/displays0
Hello, World! I'm pwnedDevice ;)
```



### Operación de escritura

Las operaciones de escritura se ejecutan cuando el dispositivo recibe una llamada `write()`, esto dispara dentro del driver la ejecución de la callback `displays_write()` que puede funcionar de diferente manera según los argumentos que reciba.

En caso de recibir como datos el comando *oled* seguido de una frase, esta frase será enviada al dispositivo mediante el *ReportID* 3 y automáticamente será mostrada en la pantalla oled.

Por el contrario, si como datos de la llamada viene el comando *7s* seguido de un dígito en hexadecimal, este será enviado al display de 7 segmentos.

```C
static ssize_t displays_write(struct file *file, const char *user_buffer,
			  				size_t count, loff_t *ppos) {

    // ...
	char* strcfg = NULL;
	
	if ((strcfg=kmalloc(count + 1, GFP_KERNEL)) == NULL)
		return -ENOMEM;

	if (copy_from_user(strcfg, user_buffer, count)){
		retval=-EFAULT;
		goto out_error;
	}

	// ...

	if (strstr(strcfg, "oled ") != NULL) {
		strcpy(buffer, strcfg + 5); // Remove oled command
		dataSize = count - 5;
		wValue = 3;  // Report-ID
	} else if (sscanf(strcfg,"7s %x", &digit) == 1) {
		buffer[0] = 4; // ReportID
		buffer[1] = digit;
		dataSize = 2;
		wValue = 4;  // Report-ID
	}

	if (wValue != 0) {
		retval = usb_control_msg(dev->udev,	
                usb_sndctrlpipe(dev->udev, 00), /* Specify endpoint #0 */
                USB_REQ_SET_CONFIGURATION, 
                USB_DIR_OUT| USB_TYPE_CLASS | USB_RECIP_INTERFACE,
                wValue,	/* wValue */
                0, 	/* wIndex=Endpoint # */
                buffer,	/* Pointer to the message */ 
                dataSize, /* message's size in bytes */
                0);
	}
	
	// ...
}
```



## LedPartyDriver

*LedPartyDriver* es el driver USB encargado de la comunicación con los periféricos cuando el dispositivo se encuentra flasheado con la plantilla *LED_PARTY*.

Permite enviar comandos para controlar el anillo de led, ya sea el anillo completo o  un led concreto.

### Operación de lectura

Cuando este driver recibe una operación de lectura, lanza una transferencia de tipo *CONTROL* al *ReportID* 2 del dispositivo y devuelve la configuración actual de colores del anillo LED.

El dispositivo transfiere un buffer de 4 bytes de datos donde los 3 últimos son el color del anillo codificado en *RGB*. Con esta información el driver construye la cadena, *R: valor G: valor B: valor*, y la devuelve al usuario.

```C
static ssize_t ledParty_read(struct file *file, char *user_buffer,
			  				size_t count, loff_t *ppos) {

    // ...
    
	unsigned char* message;	
	message = kmalloc(MAX_LEN_MESSAGE, GFP_DMA);

	/* zero fill*/
	memset(message, 0, MAX_LEN_MESSAGE);

	wValue = 0x2;  // Report-ID
	msgSize = 4;

	retval = usb_control_msg_recv(dev->udev,	
                0, 
                USB_REQ_CLEAR_FEATURE, 
                USB_DIR_IN | USB_TYPE_CLASS | USB_RECIP_DEVICE,
                wValue,	/* wValue = Descriptor index */
                0x0, 	/* wIndex = Endpoint # */
                message,	/* Pointer to the message */ 
                msgSize, /* message's size in bytes */
                0, /* Dale lo necesario*/
                GFP_DMA);

	// ...

	sprintf(message, "R: %d G: %d B: %d \n", message[1], message[2], message[3]);

    // ...
}
```

```bash
$ cat /dev/usb/ledParty0
R: 0 G: 0 B: 0
```



### Operación de escritura

Este driver soporta dos diferentes comandos de escritura a través de la función `write()`.

Cuando la función de callback `ledParty_write()`es ejecutada con el comando *setFullColor* seguido de una combinación de color separada por dos puntos, *R:G:B*, se envía una transferencia de tipo *CONTROL* con el color indicado al *ReportID* 1 y el anillo cambia de color por completo.

Otro de los comandos disponibles es *setLedColor* seguido del número de led a cambiar y el color, *nLed:R:G:B*. Este comando envía la información al *ReportID* 2, que cambia el led indicado por el color que recibe en la transferencia. A continuación se ilustra la implementación de esta función.

```C
static ssize_t ledParty_write(struct file *file, const char *user_buffer,
			  				size_t count, loff_t *ppos) {

    // ...
    
	int r,g,b,led = 0;
	char* strcfg = NULL;  // user input 
    char *buffer;

    // ...

	if (sscanf(strcfg,"setFullColor %d:%d:%d", &r, &g, &b) == 3 
			&& (r >= 0 && r <= 255) && (g >= 0 && g <= 255) 
        		&& (b >= 0 && b <= 255)) {		
		buffer[0] = 1; // ReportID
		buffer[1] = r;
		buffer[2] = g;
		buffer[3] = b;
		dataSize = 4;
		wValue = 1;
	} else if (sscanf(strcfg,"setLedColor %d:%d:%d:%d", &led, &r, &g, &b) == 4 
			&& (r >= 0 && r <= 255) && (g >= 0 && g <= 255) 
               && (b >= 0 && b <= 255) && (led >= 0 && led <= LED_SIZE)) {
		buffer[0] = 2; // ReportID
		buffer[1] = led;
		buffer[2] = r;
		buffer[3] = g;
		buffer[4] = b;
		dataSize = 5;
		wValue = 2;
	}

	if (wValue != 0) {
		retval = usb_control_msg(dev->udev,	
                usb_sndctrlpipe(dev->udev, 00), /* Specify endpoint #0 */
                USB_REQ_SET_CONFIGURATION, 
                USB_DIR_OUT| USB_TYPE_CLASS | USB_RECIP_INTERFACE,
                wValue,	/* wValue */
                0, 	/* wIndex=Endpoint # */
                buffer,	/* Pointer to the message */ 
                dataSize, /* message's size in bytes */
                0);
	}
	
	// ...
}
```



## PWMDriver

Este driver es compatible cuando el dispositivo se encuentra flasheado con la plantilla *PWM*. Esta plantilla expone un led y un buzzer capaces de ser controlados mediante señales *PWM* generadas por el microcontrolador, y el driver permite comunicar el host con estos dispositivos.

### Operación de lectura

Al igual que en el driver *DisplaysDriver*, este módulo no dispone de información relevante de lectura desde el microcontrolador. Por ello cuando llega una operación `read()` a la función de callback, `pwm_read()`, se envía una transferencia de tipo *CONTROL* al dispositivo que devuelve una cadena a modo de respuesta.

```C
static ssize_t pwm_read(struct file *file, char *user_buffer,
			  				size_t count, loff_t *ppos) {

    // ...
    
	unsigned char* message;	

    // ...

	wValue = 0x1;  // Report-ID
	msgSize = 33;

	retval = usb_control_msg_recv(dev->udev,	
                0, 
                USB_REQ_CLEAR_FEATURE, 
                USB_DIR_IN | USB_TYPE_CLASS | USB_RECIP_DEVICE,
                wValue,	/* wValue = Descriptor index */
                0x0, 	/* wIndex = Endpoint # */
                message,	/* Pointer to the message */ 
                msgSize, /* message's size in bytes */
                0, /* Dale lo necesario*/
                GFP_DMA);

	// ...

	message[33]='\n';
	message[34]='\0';
	
	if (copy_to_user(user_buffer, message + 1, strlen(message + 1))){
		retval=-EFAULT;
		goto out_error;
	}

    // ...
}
```



### Operación de escritura

Para la operación de escritura, este driver también soporta varios comandos que interactúan con los dos periféricos soportados.

El primero de ellos es *blinkLed* seguido de un *booleano* que indica si se desea parar (1) o reanudar (0) el parpadeo del led. Esta transferencia también se hace usando el tipo *CONTROL* y envía un paquete al *ReportID* 5.

El segundo comando que soporta el driver es *buzzer* seguido de la frecuencia deseada y de si se quiere parar (1) o reanudar (0) el sonido. Igual que el comando anterior, envía una transferencia de tipo *CONTROL* pero esta vez al *ReportID* 6. Esta función del driver es muy útil de usar ya que escribiendo un programa de usuario, como el que se puede encontrar en el directorio `scripts/`del proyecto, se pueden generar melodías haciendo variar las frecuencias y tiempos de vibración del buzzer.

```C
static ssize_t pwm_write(struct file *file, const char *user_buffer,
			  				size_t count, loff_t *ppos) {

    // ...
    
	int freq,stop = 0;
	char* strcfg = NULL;
	char *buffer;

	// ...

	if (sscanf(strcfg,"blinkLed %d", &stop) == 1) {
		buffer[0] = 5; // ReportID
		buffer[1] = stop;
		dataSize = 2;
		wValue = 5;
	} else if (sscanf(strcfg,"buzzer %d %d", &freq, &stop) == 2) {
		buffer[0] = 6; // ReportID

		if (stop) {
			buffer[1] = 1;
			dataSize = 2;
		} else {
			buffer[1] = 0;
			buffer[2] = (freq >> 24) & 0xFF;
			buffer[3] = (freq >> 16) & 0xFF;
			buffer[4] = (freq >> 8) & 0xFF;
			buffer[5] = freq & 0xFF;
			dataSize = 6;
		}

		wValue = 6;
	}

	if (wValue != 0) {
		retval = usb_control_msg(dev->udev,	
                usb_sndctrlpipe(dev->udev, 00), /* Specify endpoint #0 */
                USB_REQ_SET_CONFIGURATION, 
                USB_DIR_OUT| USB_TYPE_CLASS | USB_RECIP_INTERFACE,
                wValue,	/* wValue */
                0, 	/* wIndex=Endpoint # */
                buffer,	/* Pointer to the message */ 
                dataSize, /* message's size in bytes */
                0);
	}
	
	// ...
}
```



## AllDevicesDriver

Este driver es el más potente de todos, esencialmente es una combinación de todos ellos para cuando el dispositivo se encuentra flasheado con la plantilla *ALL_DEVICES*.

Permite controlar todos los periféricos, así como acceder a las dos operaciones de lectura que ofrece el dispositivo.

