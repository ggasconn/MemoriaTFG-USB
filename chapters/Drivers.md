<!-- Leave a blank line before the title -->

# Drivers y casos prácticos {#sec:drivers}

En este capítulo se describe el paquete de drivers desarrollados como ejemplo para este proyecto. Ya que este material tiene una finalidad docente y de aprendizaje, el funcionamiento de los drivers es sencillo, pero no por ello simple de implementar. Se ofrecen 5 drivers los cuales hacen uso de nuevas funciones y tipos de datos de la API que nos proporciona *USB core*, además de un driver específico que hace uso de la API asíncrona junto con transferencias tipo *INTERRUPT* *IN*. Con estos drivers se pretende que los usuarios puedan probar y experimentar el funcionamiento de las utilidades USB que nos ofrece el kernel Linux, trabajando con ambas APIs y usando diferentes tipos de transferencias.

A continuación se describe a modo de introducción la API de USB del kernel de Linux, después se explica con detalle cada uno uno de ellos explicando su funcionamiento y arquitectura interna, con ejemplos de código.



## Drivers USB en el kernel Linux

Un componente muy importante en los drivers USB es la definición de la interfaz `usb_driver` [@apiusb-kernel], que debe implementar cualquier módulo para poder registrarse y administrar controladores que interactuen con periféricos USB. Esta estructura viene definida en `<linux/usb.h>`, y se deben implementar aquellos campos que requieran definir las funciones que interactúen con el dispositivo USB. Lo hemos definido como interfaz ya que muchos de sus campos son punteros a función. Estructuras similares son `file_operations` y `proc_ops`, de las que se hablarán más adelante.

```{.c .numberLines}
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

- `name`, línea 2: Cadena de caracteres, representa el nombre del driver.

- `id_table`, línea 15: Tabla de dispositivos compatibles con el driver, definidos mediante un *vendor-ID* y *product-ID*.

- `probe`, línea 3: Este campo define la función encargada de ejecutarse cuando se detecta un dispositivo compatible con el controlador USB al conectarlo al host, se encarga de inicializarlo y prepararlo para su uso. Se pasa como parámetro a esta función el descriptor del dispositivo, de tipo`struct usb_interface *`.

- `disconnect`, línea 5: Esta función se ejecuta cuando se desconecta el dispositivo USB que previamente ha sido gestionado por el driver.

  

En nuestros drivers, definimos cuatro campos básicos de la interfaz `usb_driver`, como se muestra a continuación:

```{.c .numberLines}
static struct usb_driver pwnedDevice_driver = {
	.name =		"pwnedDevice",
	.probe =	pwnedDevice_probe,
	.disconnect =	pwnedDevice_disconnect,
	.id_table =	pwnedDevice_table,
};
```

En la estructura `usb_driver` definimos punteros a función en `.probe` y `.disconnect` (líneas 3 y 4) para implementar las funciones `pwnedDevice_probe()` y `pwnedDevice_disconnect()`:

```{.c .numberLines}
static int pwnedDevice_probe(struct usb_interface *interface,
		      			const struct usb_device_id *id) {
	struct usb_pwnedDevice *dev;
	int retval = -ENOMEM;

	/*
 	 * Allocate memory for a usb_pwnedDevice structure.
	 * This structure represents the device state.
	 * The driver assigns a separate structure to each pwnedDevicestick device
 	 *
	 */
	dev = kmalloc(sizeof(struct usb_pwnedDevice), GFP_KERNEL);

	if (!dev) {
		dev_err(&interface->dev, "Out of memory\n");
		goto error;
	}

	/* Initialize the various fields in the usb_pwnedDevice structure */
	kref_init(&dev->kref);
	dev->udev = usb_get_dev(interface_to_usbdev(interface));
	dev->interface = interface;

	/* save our data pointer in this interface device */
	usb_set_intfdata(interface, dev);

	/* we can register the device now, as it is ready */
	retval = usb_register_dev(interface, &pwnedDevice_class);
	if (retval) {
		/* something prevented us from registering this driver */
		dev_err(&interface->dev,
			"Not able to get a minor for this device.\n");
		usb_set_intfdata(interface, NULL);
		goto error;
	}

	/* let the user know what node this device is now attached to */	
	dev_info(&interface->dev,
		 "PwnedDevice now available via pwnedDevice%d",
		 interface->minor);
	return 0;

error:
	if (dev)
		/* this frees up allocated memory */
		kref_put(&dev->kref, pwnedDevice_delete);
	return retval;
}
```

Como hemos mencionado previamente, esta función es la encargada de reservar memoria e inicializar los campos de la estructura del dispositivo, como podemos observar en el código anterior entre las líneas 12 y 22. Destacar que a esta función tiene como parámetro el descriptor del dispositivo, que se utilizará para poder registrarlo posteriormente. La función invocada cuando se desconecta el dispositivo previamente gestionado por el driver es la siguiente:

```{.c .numberLines}
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

	dev_info(&interface->dev, "PwnedDevice device #%d has been disconnected", minor);
}
```

En la última función, en la línea 15, se decrementa el contador de referencias del dispositivo. Puede llegar a eliminarlo si el valor de este contador es 0. Esta función también es encargada de invalidar la interfaz del dispositivo.

A continuación se muestran las funciones encargadas de la inicialización del módulo `pwnedDevicedrv_module_init()` y de eliminar su registro mediante la función `pwnedDevicedrv_module_cleanup()`. 

```{.c .numberLines}
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

Para llevar a cabo el registro del módulo, se invoca la función `usb_register()` en la línea 3, que acepta como parámetro un puntero a la interfaz `usb_driver`, definida mediante la variable `pwnedDevice_driver`. En el caso de que se produzca correctamente el registro, se devuelve 0. En caso contrario, un número negativo con el código de error. Para la descarga del módulo, se ejecuta `usb_deregister()` en la línea 8, que se encarga de la limpieza del módulo, aceptando un puntero a la interfaz `usb_driver`.

En la instanciación de `usb_driver`, a parte de la definición de los campos explicados previamente, se ha definido otro para los dispositivos compatibles (`.id_table =	pwnedDevice_table`, línea 5 en la estructura definida al comienzo de la sección). La implementación de esta tabla es la siguiente:

```{.c .numberLines}
/* Define these values to match your devices */
#define VENDOR_ID	0X20A0
#define PRODUCT_ID	0X41E5

/* table of devices that work with this driver */
static const struct usb_device_id pwnedDevice_table[] = {
	{ USB_DEVICE(VENDOR_ID, PRODUCT_ID) },
	{ }					/* Terminating entry */
};
MODULE_DEVICE_TABLE(usb, pwnedDevice_table);
```

La tabla `pwnedDevice_table[]` definida en la linea 6 del código anterior almacena todos los identificadores de dispositivos `usb_device_id` que el driver puede gestionar. Cada posición del array es un descriptor del dispositivo, que se inicializa mediante la macro `USB_DEVICE()` definida en la línea 7, cuyos parámetros son el identificador del fabricante `VENDOR_ID` y el identificador del producto `PRODUCT_ID`, definidos en las líneas 2 y 3. Cada vez que un dispositivo se conecta al host con los valores de `VENDOR_ID` y `PRODUCT_ID` definidos anteriormente, se consulta la tabla `pwnedDevice_table[]` que contiene esta información y posteriormente se ejecuta la función `probe()` del driver USB (en nuestro caso `pwnedDevice_probe()`). 



Otro de los campos importantes de un driver es la estructura de estado, ya que con esta estructura se permite la conexión simultánea de varios dispositivos del mismo tipo (asociando a cada uno una estructura que represente su estado). En nuestros drivers, hemos definido la siguiente estructura:

```{.c .numberLines}
/* Structure to hold all of our device specific stuff */
struct usb_pwnedDevice {
	struct usb_device	*udev;			/* the usb device for this device */
	struct usb_interface	*interface;		/* the interface for this device */
	struct kref		kref;
};
```
A continuación se explican los distintos campos de la estructura:

- `udev`, línea 3: Es una referencia al descriptor del dispositivo físico USB, que es una estructura de datos utilizada por las funciones principales del USB para transferir datos entre el host y el dispositivo USB. Este puntero se encuentra en la mayoría de las estructuras de estado de los controladores USB y se obtiene durante la ejecución de la función `probe()`, donde se crea e inicializa la estructura de estado del dispositivo.

- `interface`, línea 4: Es una referencia al descriptor de la interfaz USB que el controlador está gestionando. Se refiere a un único dispositivo lógico. Al igual que el campo `udev`, este puntero se inicializa durante la ejecución de la función `probe()`, y se encuentra en muchas estructuras de estado de los controladores USB.

- `kref`, línea 5: Es un contador de referencias utilizado para gestionar la estructura de estado. En el kernel, el tipo de datos `struct kref` se utiliza para implementar contadores de referencia de objetos del núcleo. Estos contadores son importantes para determinar cuándo es seguro liberar la memoria de un objeto. El tipo de datos `struct kref` proporciona operaciones seguras (desde el punto de vista de la concurrencia en el kernel) para incrementar y decrementar el contador de referencias. Se inicializa con la función `kref_init()` (línea 20 de la función `pwnedDevice_probe()`), se incrementa con `kref_get()` y se decrementa con `kref_put()` (línea 15 de la función `pwnedDevice_disconnect()` y 46 de `pwnedDevice_probe()` ejecutándose en este último caso únicamente cuando la función falla). 

  El contador de referencia `kref` en los drivers se utiliza para dos propósitos principales: 

  1.  Gestionar correctamente la memoria de la estructura de estado.
  2.  Resolver problemas de concurrencia asociados con la desconexión física del dispositivo cuando el controlador está en uso. Básicamente, la memoria para el objeto de estado se asigna dinámicamente. Se solicita memoria cuando el dispositivo se conecta al sistema (en la función `pwnedDevice_probe()`, línea 20) y, en ese momento, el valor interno del contador es 1. El contador de referencias se incrementa con `kref_get()` en el código del controlador cada vez que un proceso de usuario está utilizando el dispositivo, y se decrementa con `kref_put()` cuando el proceso deja de usarlo. Gracias al correcto manejo del contador, la memoria de la estructura de estado se libera únicamente cuando el contador alcanza el valor cero al decrementarse.

Cabe destacar que sin la ejecución de `usb_set_intfdata(interface, dev)` en la línea 25 de la función `pwnedDevice_probe()` mostrada anteriormente, no sería posible recuperar la estructura de estado desde las funciones del driver que implementan operaciones sobre ficheros especiales de caracteres, y por lo tanto no se podrían realizar transferencias de datos entre el dispositivo y el kernel, por ejemplo, en la función `pwnedDevice_write()`. 

Como hemos adelantado en el párrafo anterior, otro de los aspectos importantes es poder interactuar con el driver desde el espacio de usuario. Para ello, es necesario registrar los dispositivos reconocidos (uno por cada dispositivo) en una clase del Linux Device Model (LDM), encargándose de la asignación de *major numbers* el propio kernel de Linux, y registrándolos en distintas clases dentro de, por ejemplo, `input`, `usb`, `ttyACM`, `tty_usb`, etc. [@linuxusb-devices].

```{.c .numberLines}
extern int usb_register_dev(struct usb_interface *intf,
            		struct usb_class_driver *class_driver);
```

La clase `usb` en Linux abarca la mayoría de los dispositivos USB. Estos dispositivos se identifican con un número identificativo, el `major number`, cuyo valor es 180. Para utilizar esta clase y su `major number`, el controlador básico del dispositivo USB registra cada dispositivo conectado mediante la función `usb_register_dev()`, definida en la línea 28 de la función `pwnedDevice_probe()`, que realiza las siguientes acciones:

1. Registra el dispositivo en la clase `usb` y le asigna un número `minor number`.
2. Automáticamente crea un archivo de dispositivo en la ubicación `/dev`, interactuando con el servicio `Udev`.
3. Asigna una interfaz de operaciones (`file_operations`) al archivo de dispositivo, definiendo las funciones que se ejecutarán cuando se realicen operaciones sobre estos ficheros de caracteres (por ejemplo, `ẁrite()` o `read()`).



La estructura `usb_class_driver` se define como sigue:

```{.c .numberLines}
struct usb_class_driver {
    char *name;
    char *(*devnode)(struct device *dev, umode_t *mode);
    const struct file_operations *fops;
    int minor_base;
};
```

- `name`, línea 2: Es una cadena de caracteres que contiene el nombre del dispositivo y también se utiliza para codificar el nombre de su archivo especial asociado.

- `devnode`, línea 3: Es una función que debe ser definida en el controlador y se utiliza para indicar:
  1. La ubicación relativa dentro de `/dev` donde se creará el archivo de dispositivo.
  2. Los permisos que se asignarán al archivo de dispositivo (usando el parámetro `mode`).
  
- `file_operations`, línea 4: Es un puntero que apunta a las operaciones que el controlador ejecutará cuando un programa acceda al dispositivo. El controlador debe instanciar la estructura `file_operations` e implementar las operaciones correspondientes.

- `minor_base`, línea 5: Indica el valor inicial en el rango de números `minor_numer` asignados para este controlador.



El siguiente fragmento de código muestra la definición de la estructura global de tipo `struct usb_class_driver`, que se pasa como parámetro a la función `usb_register_dev()` a través de la línea 28 en `pwnedDevice_probe()`:

```{.c .numberLines}
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

Como puede observarse en la estructura `pwnedDevice_class` definida desde la línea 34 hasta la 39, el campo `name` denota un patrón. Según este patrón el nombre de los ficheros de dispositivo serán *pwnedDevice0*, *pwnedDevice1*, *pwnedDevice2*, etc.

El segundo campo, `devnode`, se inicializa con la dirección de la función `set_device_permissions()` que se define anteriormente en el código. El valor de retorno de esta función indica a `udev` dónde debe crear el archivo especial de dispositivo dentro de `/dev`. En este caso, la implementación especifica que la ruta será `/dev/usb`. Además, el parámetro de retorno `mode` se utiliza para indicar los permisos del archivo de dispositivo, que en esta implementación se establecen en `666` (permisos de lectura y escritura para todos).

El tercer campo, `fops`, se inicializa con la dirección que nos proporciona la variable `pwnedDevice_fops`. En la inicialización de esta variable, se asocian cuatro operaciones al controlador: `pwnedDevice_write()`, `pwnedDevice_read()`,  `pwnedDevice_open()` y `pwnedDevice_release()`. Estas operaciones se invocan cuando se ejecutan las llamadas al sistema `write()`, `read()`, `open()` y `close()` desde un programa de usuario en relación a cualquier archivo especial de caracteres asociado al controlador.

Por último, el campo `minor_base` se inicializa con una macro definida al principio (`#define USB_MINOR_BASE    0`). [@blinkjc-description]



## BasicInterrupt - Driver asíncrono con comunicación INTERRUPT IN 

Este módulo del kernel implementa un driver USB utilizando el endpoint de tipo *INTERRUPT IN* del dispositivo USB, así como la API asíncrona del kernel. [@drivers-linux]

El funcionamiento del módulo es muy sencillo. Al cargar el módulo, se ejecuta la función `pwnedDevice_probe()` del driver, que reserva espacio para las distintas estructuras del driver, utilizadas para la interfaz y los paquetes URBs, así como el descubrimiento de endpoints. 

```{.c .numberLines}
/*
 * Invoked when the USB core detects a new
 * pwnedDevicestick device connected to the system.
 */
static int pwnedDevice_probe(struct usb_interface *interface,
		      const struct usb_device_id *id)
{
	struct usb_pwnedDevice *dev;
	struct usb_host_interface *iface_desc;
	struct usb_endpoint_descriptor *endpoint;
	int retval = -ENOMEM;
	int i;

	/*
 	 * Allocate memory for a usb_pwnedDevice structure.
	 * This structure represents the device state.
	 * The driver assigns a separate structure to each pwnedDevicestick device
 	 *
	 */
	dev = kmalloc(sizeof(struct usb_pwnedDevice), GFP_KERNEL);

	if (!dev) {
		dev_err(&interface->dev, "Out of memory\n");
		goto error;
	}

	/* Initialize the various fields in the usb_pwnedDevice structure */
	kref_init(&dev->kref);
	dev->udev = usb_get_dev(interface_to_usbdev(interface));
	dev->interface = interface;

	iface_desc = interface->cur_altsetting;

	for (i = 0; i < iface_desc->desc.bNumEndpoints; ++i) {
		endpoint = &iface_desc->endpoint[i].desc;

		if (((endpoint->bEndpointAddress & USB_ENDPOINT_DIR_MASK)
		     == USB_DIR_IN)
		    && ((endpoint->bmAttributes & USB_ENDPOINT_XFERTYPE_MASK)
		        == USB_ENDPOINT_XFER_INT))
			dev->int_in_endpoint = endpoint;
	}

	if (! dev->int_in_endpoint) {
		pr_err("could not find interrupt in endpoint");
		goto error;
	}


	/* Request IN URB */
	dev->int_in_urb = usb_alloc_urb(0, GFP_KERNEL);
	if (!dev->int_in_urb) {
		printk(KERN_INFO "Error allocating URB");
		retval = -ENOMEM;
		goto error;
	}

	/* save our data pointer in this interface device */
	usb_set_intfdata(interface, dev);

	/* we can register the device now, as it is ready */
	retval = usb_register_dev(interface, &pwnedDevice_class);
	if (retval) {
		/* something prevented us from registering this driver */
		dev_err(&interface->dev,
			"Not able to get a minor for this device.\n");
		usb_set_intfdata(interface, NULL);
		goto error;
	}

	/* let the user know what node this device is now attached to */	
	dev_info(&interface->dev,
		 "PwnedDevice now available via pwnedDevice-%d",
		 interface->minor);
	return 0;

error:
	if (dev->int_in_urb)
		usb_free_urb(dev->int_in_urb);

	if (dev)
		/* this frees up allocated memory */
		kref_put(&dev->kref, pwnedDevice_delete);

	return retval;
}
```

La línea 51 del anterior código realiza la llamada a la función para la reserva de memoria del URB una vez inicializados todos sus campos, y después de haber realizado la búsqueda de endpoints mediante el bucle definido entre las líneas 34 y 42.

Una vez cargado en el kernel, expone un dispositivo de caracteres que tiene implementada la operación de lectura. Cuando un proceso hace una operación de `read()` sobre el nodo, por ejemplo con `cat`, iniciaremos la rutina que envía el URB de tipo *INTERRUPT IN* al dispositivo pidiéndole que le rellene el buffer enviado con datos, como se puede observar en el siguiente fragmento de código. El envío del URB se hace efectivo a través de la llamada a la función `usb_submit_urb()` de la línea 21.

```{.c .numberLines}
static ssize_t pwnedDevice_read(struct file *file, char *user_buffer,
			  size_t count, loff_t *ppos)
{
	struct usb_pwnedDevice *dev;
	int retval = 0;

	dev = file->private_data;
	
	if (*ppos > 0)
		return 0;

	/* Send URB. */
	usb_fill_int_urb(dev->int_in_urb, dev->udev,
	                 usb_rcvintpipe(dev->udev,
	                                dev->int_in_endpoint->bEndpointAddress),
	                 dev->int_in_buffer,
	                 le16_to_cpu(0x0008),
	                 pwnedDevice_int_in_callback,
	                 dev,
	                 dev->int_in_endpoint->bInterval);
	retval = usb_submit_urb(dev->int_in_urb, GFP_KERNEL);

	if (retval != 0)
		return retval;

	return 0; // No bytes returned. Async USB API used.
}
```

Como se usa la API asíncrona de USB la llamada a `read()` devuelve siempre 0 bytes, ya que no se sabe con exactitud cuándo se va a devolver el *URB* relleno con datos y al ser una función no bloqueante no esperamos a que llegue. El resultado de esta solicitud de datos al dispositivo se retorna a través de la función de callback, que se invoca una vez el URB vuelve desde el dispositivo al host. La cabecera de la función es la siguiente: `pwnedDevice_int_in_callback(struct urb *urb)`, función explicada más adelante.

Como este driver está programado usando la API asíncrona que nos ofrece el kernel, toda la gestión de *URBs*, así como el descubrimiento de endpoints y todo el control de transferencia es gestionado por el driver. A continuación, veremos varios puntos relevantes del código que marcan una diferencia en el uso de esta API y del tipo de transferencia *INTERRUPT*.

### Descubrimiento de endpoints

El descubrimiento de endpoints es de lo primero que se hace cuando un driver ha sido asociado a un dispositivo, esta asociación se hace mediante el *ProductID* y *VendorID*.

Cuando el *USB Core* asigna el driver al dispositivo lo primero que se ejecuta es la función `pwnedDevice_probe()`, que ejecuta todas las tareas necesarias de inicialización, entre ellas el descubrimiento de endpoints, que se realiza entre las líneas 28 y 47.

```{.c .numberLines}
static int pwnedDevice_probe(struct usb_interface *interface,
		      const struct usb_device_id *id);
```

Como argumento de esta función `pwnedDevice_probe()` recibimos una descripción de la interfaz que está siendo conectada con el driver, en ella podemos ver los diferentes tipos de descriptores USB que muestran las características del dispositivo. Dentro de estos descriptores podemos llegar a los descriptores de interfaces donde buscaremos un endpoint de tipo *INTERRUPT* y en sentido *IN*, en caso de no encontrarse el driver no puede funcionar y se aborta la inicialización.

### Reserva de URBs

Dentro de la función `pwnedDevice_probe()` también se realiza la reserva de memoria para el *URB* que se va a enviar al dispositivo para ser rellenado con información, tal y como viene definido entre las líneas 51 y 59 de dicha función.

Esta acción se hace a través de la función `usb_alloc_urb()` (línea 51) que devuelve un puntero a una región de la memoria que se ha destinado al *URB* solicitado. Esta región será reutilizada y compartida con la controladora USB durante toda la vida del driver. Cabe destacar que es único por dispositivo, ya que el driver soporta varios dispositivos a la vez, cada uno de ellos reservará su propio *URB*.

### Montaje de un URB

Cuando el driver recibe una operación tipo `read()`, en la función de lectura `pwnedDevice_read()` se rellena el URB reservado previamente, y se lleva a la controladora USB para que sea enviado al dispositivo, tal y como viene definido en el código de dicha función entre las líneas 13 y 21.

Los datos introducidos al URB mediante la función que nos aporta el kernel `usb_fill_int_urb()` (línea 13), la cual tiene como parámetros la información que queremos hacerle llegar, así como otros diferentes pero obligatorios, y un buffer que rellenará el dispositivo con los datos que le solicitamos.

Una vez todos los datos están correctos, tenemos que llamar a la función `usb_submit_urb()` (línea 21) para decirle a la controladora que ese URB está listo para ser enviado y proceder a ello.

### Gestión de la respuesta

Al usarse la API asíncrona después del envío que realiza el driver, no hay ningún bloqueo que sirva de espera a la respuesta del dispositivo, es por esto que la respuesta se gestiona desde una función externa que es llamada cuando el URB vuelve relleno. Esta función es `pwnedDevice_int_in_callback()`.

Dentro de la función se realizan dos tareas sencillas. Lo primero es comprobar el estado de la transferencia, ya que en caso de haberse producido algún fallo los datos devueltos puede que no sean coherentes. Esta tarea se realiza entre las líneas 7 y 14 del siguiente código. En caso de que todo haya ido bien, se extraen los datos del buffer que previamente se reservó para rellenarse con información por el dispositivo, y se muestra por la salida de información del kernel usando `printk()`, tarea realizada en las líneas 16 y 17.

```{.c .numberLines}
/*
* Callback function. Executed when INT IN URB returns back with data.
*/
static void pwnedDevice_int_in_callback(struct urb *urb) {
	struct usb_pwnedDevice *dev = urb->context;

	if (urb->status) {
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

El driver utiliza la API síncrona que nos ofrece *USB Core* y todas las transferencias van sobre el tipo *CONTROL*. Este driver y los posteriores comparten las mismas funciones de inicialización, cambiando las funciones para operaciones de `read()` y `write()`.

Una vez cargado el módulo que hace de driver en el kernel, este expone un dispositivo de caracteres bajo la ruta `/dev/usb/displaysx`, donde *x* hace referencia al índice del dispositivo que se encuentra conectado, ya que el driver soporta más de un dispositivo a la vez.

### Operación de lectura

Cuando el dispositivo recibe una llamada `read()`, independientemente del origen, se ejecuta la función `displays_read()`. Como el perfil *DISPLAYS* no implementa ninguna información útil de lectura sobre sus periféricos, cuando una operación de este tipo llega al dispositivo devuelve una frase de 33 bytes a modo de respuesta. El mensaje llega a través del URB que se recibe en la función `usb_control_msg_recv()` de la línea 23, copiándose al espacio de usuario de forma segura mediante la llamada `copy_to_user()` de la línea 41.

```{.c .numberLines}
static ssize_t pwnedDevice_read(struct file *file, char *user_buffer,
			  				size_t count, loff_t *ppos) {
	struct usb_pwnedDevice *dev;
	int retval = 0;
	unsigned char* message;	
	int nr_bytes = 0;
	unsigned int wValue = 0;
	int msgSize = 0;

	dev = file->private_data;
	
	if (*ppos>0)
		return 0;

	message = kmalloc(MAX_LEN_MESSAGE, GFP_DMA);

	/* zero fill*/
	memset(message, 0, MAX_LEN_MESSAGE);

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

	if (retval)
		goto out_error;

	message[33]='\n';
	message[34]='\0';
	nr_bytes = strlen(message + 1); // Remove reportID
	
	if (copy_to_user(user_buffer, message + 1, strlen(message + 1))){
		retval=-EFAULT;
		goto out_error;
	}

	kfree(message);
	(*ppos) += nr_bytes;

	return nr_bytes;

out_error:
	if (message)
		kfree(message);

	return retval;	
}
```

A continuación se muestra la salida de la *shell* cuando se ejecuta la operación de lectura sobre el fichero en `/dev`

```bash
~$ cat /dev/usb/displays0
Hello, World! I'm pwnedDevice ;)
```



### Operación de escritura

Las operaciones de escritura se ejecutan cuando el dispositivo recibe una llamada `write()`, esto dispara dentro del driver la ejecución de la función `displays_write()` que puede funcionar de diferente manera según los argumentos que reciba.

En caso de recibir como datos el comando *oled* seguido de una frase, esta frase será enviada al dispositivo mediante el *ReportID* 3 y automáticamente será mostrada en la pantalla OLED (línea 28). Por el contrario, si como datos de la llamada viene el comando *7s* seguido de un dígito en hexadecimal, este será enviado al display de 7 segmentos (línea 32).

```{.c .numberLines}
static ssize_t pwnedDevice_write(struct file *file, const char *user_buffer,
			  				size_t count, loff_t *ppos) {
    
	struct usb_pwnedDevice *dev;
	int retval = 0;
	unsigned int digit = 0;
	char* strcfg = NULL;
	unsigned int wValue = 0;
	int dataSize = 0;
	char *buffer;

	dev = file->private_data;
	
	if ((strcfg=kmalloc(count + 1, GFP_KERNEL)) == NULL)
		return -ENOMEM;

	if (copy_from_user(strcfg, user_buffer, count)){
		retval=-EFAULT;
		goto out_error;
	}
	strcfg[count] = '\0';

	if ((buffer = kmalloc(MAX_LEN_MESSAGE, GFP_KERNEL)) == NULL) {
		retval = -ENOMEM;
		goto out_error;
	}

	if (strstr(strcfg, "oled ") != NULL) {
		strcpy(buffer, strcfg + 5); // Remove oled command
		dataSize = count - 5;
		wValue = 3;
	} else if (sscanf(strcfg,"7s %x", &digit) == 1) {
		buffer[0] = 4; // ReportID
		buffer[1] = digit;
		dataSize = 2;
		wValue = 4;
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
	
	if (retval < 0 && retval != -EPIPE){
		printk(KERN_ALERT "Executed with retval=%d\n",retval);
		goto out_error;		
	}

	goto ok_path;

ok_path:
	kfree(strcfg);
	kfree(buffer);
	(*ppos)+=count;
	return count;

out_error:
	if (strcfg)
		kfree(strcfg);
	if (buffer)
		kfree(buffer);
	return retval;	
}
```



## LedPartyDriver

*LedPartyDriver* es el driver USB encargado de la comunicación con los periféricos cuando el dispositivo se encuentra flasheado con el perfil *LED_PARTY*.

Permite enviar comandos para controlar el anillo de led, ya sea el anillo completo o  un led concreto.

### Operación de lectura

Cuando este driver recibe una operación de lectura, lanza una transferencia de tipo *CONTROL* al *ReportID* 2 del dispositivo y devuelve la configuración actual de colores del anillo LED.

El dispositivo transfiere un buffer de 4 bytes de datos donde los 3 últimos son el color del anillo codificado en *RGB*. Con esta información, el driver construye la cadena, *R: valor G: valor B: valor*, y la devuelve al usuario (mediante la función `sprintf()` de la línea 38).

```{.c .numberLines}
static ssize_t ledParty_read(struct file *file, char *user_buffer,
			  				size_t count, loff_t *ppos) {
    
	struct usb_pwnedDevice *dev;
	int retval = 0;
	unsigned char* message;	
	int nr_bytes = 0;
	unsigned int wValue = 0;
	int msgSize = 0;

	dev = file->private_data;
	
	if (*ppos>0)
		return 0;

	message = kmalloc(MAX_LEN_MESSAGE, GFP_DMA);

	/* zero fill*/
	memset(message, 0, MAX_LEN_MESSAGE);

	wValue = 0x2;
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

	if (retval)
		goto out_error;

	sprintf(message, "R: %d G: %d B: %d \n", message[1], message[2], message[3]);
	nr_bytes = strlen(message);

	if (copy_to_user(user_buffer, message, strlen(message))){
		retval=-EFAULT;
		goto out_error;
	}

	kfree(message);
	(*ppos) += nr_bytes;

	return nr_bytes;

out_error:
	if (message)
		kfree(message);

	return retval;	
}
```

Cuando se ejecuta la operación de lectura sobre el fichero en `/dev`, la salida que se muestra es la siguiente:

```bash
$ cat /dev/usb/ledParty0
R: 0 G: 0 B: 0
```



### Operación de escritura

Este driver soporta dos comandos de escritura diferentes a través de la función `write()`.

Cuando la función `ledParty_write()` es ejecutada con el comando *setFullColor* seguido de una combinación de color separada por dos puntos, *R:G:B*, se envía una transferencia de tipo *CONTROL* con el color indicado al *ReportID* 1 y el anillo cambia de color por completo, puede observarse esta implementación entre las líneas 28 y 35. Otro de los comandos disponibles es *setLedColor* (líneas de la 36 a la 44), seguido del número de LED a cambiar y el color, *nLed:R:G:B*. Este comando envía la información al *ReportID* 2, que cambia el led indicado por el color que recibe en la transferencia.

```{.c .numberLines}
static ssize_t ledParty_write(struct file *file, const char *user_buffer,
			  				size_t count, loff_t *ppos) {
    
	struct usb_pwnedDevice *dev;
	int retval = 0;
	int r,g,b,led = 0;
	char* strcfg = NULL;
	unsigned int wValue = 0;
	int dataSize = 0;
	char *buffer;

	dev = file->private_data;
	
	if ((strcfg=kmalloc(count + 1, GFP_KERNEL)) == NULL)
		return -ENOMEM;

	if (copy_from_user(strcfg, user_buffer, count)){
		retval=-EFAULT;
		goto out_error;
	}
	strcfg[count] = '\0';

	if ((buffer = kmalloc(MAX_LEN_MESSAGE, GFP_KERNEL)) == NULL) {
		retval = -ENOMEM;
		goto out_error;
	}

	if (sscanf(strcfg,"setFullColor %d:%d:%d", &r, &g, &b) == 3 
			&& (r >= 0 && r <= 255) && (g >= 0 && g <= 255) && (b >= 0 && b <= 255)) {		
		buffer[0] = 1; // ReportID
		buffer[1] = r;
		buffer[2] = g;
		buffer[3] = b;
		dataSize = 4;
		wValue = 1;
	} else if (sscanf(strcfg,"setLedColor %d:%d:%d:%d", &led, &r, &g, &b) == 4 
			&& (r >= 0 && r <= 255) && (g >= 0 && g <= 255) && (b >= 0 && b <= 255) && (led >= 0 && led <= LED_SIZE)) {
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
	
	if (retval < 0 && retval != -EPIPE){
		printk(KERN_ALERT "Executed with retval=%d\n",retval);
		goto out_error;		
	}

	goto ok_path;

ok_path:
	kfree(strcfg);
	kfree(buffer);
	(*ppos)+=count;
	return count;

out_error:
	if (strcfg)
		kfree(strcfg);
	if (buffer)
		kfree(buffer);
	return retval;	
}
```



## PWMDriver

Este driver es compatible cuando el dispositivo se encuentra flasheado con la plantilla *PWM*. Esta plantilla expone un led y un zumbador capaces de ser controlados mediante señales *PWM* generadas por el microcontrolador, y el driver permite comunicar el host con estos dispositivos.

### Operación de lectura

Al igual que en *DisplaysDriver*, este módulo no dispone de información relevante de lectura desde el microcontrolador. Por ello cuando llega una operación `read()` a la función de  `pwm_read()`, se envía una transferencia de tipo *CONTROL* al dispositivo que devuelve una cadena a modo de respuesta (tal y como se puede observar en la línea 24).

```{.c .numberLines}
static ssize_t pwm_read(struct file *file, char *user_buffer,
			  				size_t count, loff_t *ppos) {
    
	struct usb_pwnedDevice *dev;
	int retval = 0;
	unsigned char* message;	
	int nr_bytes = 0;
	unsigned int wValue = 0;
	int msgSize = 0;

	dev = file->private_data;
	
	if (*ppos>0)
		return 0;

	message = kmalloc(MAX_LEN_MESSAGE, GFP_DMA);

	/* zero fill*/
	memset(message, 0, MAX_LEN_MESSAGE);

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

	if (retval)
		goto out_error;

	message[33]='\n';
	message[34]='\0';
	nr_bytes = strlen(message + 1); // Remove reportID
	
	if (copy_to_user(user_buffer, message + 1, strlen(message + 1))){
		retval=-EFAULT;
		goto out_error;
	}


	kfree(message);
	(*ppos) += nr_bytes;

	return nr_bytes;

out_error:
	if (message)
		kfree(message);

	return retval;	
}
```



### Operación de escritura

Para la operación de escritura, este driver también soporta varios comandos que interactúan con los dos periféricos soportados.

El primero de ellos es *blinkLed* seguido de un *booleano* que indica si se desea parar (1) o reanudar (0) el parpadeo del led. Esta transferencia también se hace usando el tipo *CONTROL* y envía un paquete al *ReportID* 5. Se implementa entre las líneas 28 y 32.

El segundo comando que soporta el driver es *buzzer* seguido de la frecuencia deseada, indicando a continuación si se quiere parar (1) o reanudar (0) el sonido. Igual que el comando anterior, envía una transferencia de tipo *CONTROL* pero esta vez al *ReportID* 6. Esta función del driver es muy útil de usar ya que escribiendo un programa de usuario, como el que se puede encontrar en el directorio `scripts` del proyecto, se pueden generar melodías haciendo variar las frecuencias y tiempos de vibración del buzzer. Su implementación en este driver se encuentra entre las líneas 33 y 48.

```{.c .numberLines}
static ssize_t pwm_write(struct file *file, const char *user_buffer,
			  				size_t count, loff_t *ppos) {
    
	struct usb_pwnedDevice *dev;
	int retval = 0;
	int freq,stop = 0;
	char* strcfg = NULL;
	unsigned int wValue = 0;
	int dataSize = 0;
	char *buffer;

	dev = file->private_data;
	
	if ((strcfg=kmalloc(count + 1, GFP_KERNEL)) == NULL)
		return -ENOMEM;

	if (copy_from_user(strcfg, user_buffer, count)){
		retval=-EFAULT;
		goto out_error;
	}
	strcfg[count] = '\0';

	if ((buffer = kmalloc(MAX_LEN_MESSAGE, GFP_KERNEL)) == NULL) {
		retval = -ENOMEM;
		goto out_error;
	}

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
	
	if (retval < 0 && retval != -EPIPE){
		printk(KERN_ALERT "Executed with retval=%d\n",retval);
		goto out_error;		
	}

	goto ok_path;

ok_path:
	kfree(strcfg);
	kfree(buffer);
	(*ppos)+=count;
	return count;

out_error:
	if (strcfg)
		kfree(strcfg);
	if (buffer)
		kfree(buffer);
	return retval;	
}
```



## AllDevicesDriver

Este driver es el más potente de todos, es una combinación de todos los anteriores. Funciona cuando el dispositivo se encuentra flasheado con la plantilla *ALL_DEVICES*. Permite controlar todos los periféricos, así como acceder a las dos operaciones de lectura que ofrece el dispositivo, gestionado mediante compilación condicional que define la macro `#define GET_OPERATION`, que puede tomar los valores 0 o 1, definidos según las macros `GET_LED` y `GET_MSG`. Las implementaciones de `pwnedDevice_read()` y `pwnedDevice_write()`, como hemos mencionado, representan una combinación de todos los drivers explicados en las secciones anteriores. A continuación se muestra la implementación correspondiente a `pwnedDevice_read()`:

```{.c .numberLines}
static ssize_t pwnedDevice_read(struct file *file, char *user_buffer,
			  				size_t count, loff_t *ppos) {
	struct usb_pwnedDevice *dev;
	int retval = 0;
	unsigned char* message;	
	int nr_bytes = 0;
	unsigned int wValue = 0;
	int msgSize = 0;

	dev = file->private_data;
	
	if (*ppos>0)
		return 0;

	message = kmalloc(MAX_LEN_MESSAGE, GFP_DMA);

	/* zero fill*/
	memset(message, 0, MAX_LEN_MESSAGE);

	#if GET_OPERATION == GET_LED
		wValue = 0x2;
		msgSize = 4;
	#elif GET_OPERATION == GET_MSG
		wValue = 0x1;
		msgSize = 33;
	#endif

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

	if (retval)
		goto out_error;

	#if GET_OPERATION == GET_LED
		sprintf(message, "R: %d G: %d B: %d \n", message[1], message[2], message[3]);
		nr_bytes = strlen(message);
	
		if (copy_to_user(user_buffer, message, strlen(message))){
			retval=-EFAULT;
			goto out_error;
		}
	#elif GET_OPERATION == GET_MSG
		message[33]='\n';
		message[34]='\0';
		nr_bytes = strlen(message + 1); // Remove reportID
		
		if (copy_to_user(user_buffer, message + 1, strlen(message + 1))){
			retval=-EFAULT;
			goto out_error;
		}
	#endif


	kfree(message);
	(*ppos) += nr_bytes;

	return nr_bytes;

out_error:
	if (message)
		kfree(message);

	return retval;	
}
```



La implementación de la función `pwnedDevice_write()` se encuentra en el siguiente fragmento de código.

```{.c .numberLines}
static ssize_t pwnedDevice_write(struct file *file, const char *user_buffer,
			  				size_t count, loff_t *ppos) {
	struct usb_pwnedDevice *dev;
	int retval = 0;
	int r,g,b,led = 0;
	int freq,stop = 0;
	unsigned int digit = 0;
	char* strcfg = NULL;
	unsigned int wValue = 0;
	int dataSize = 0;
	char *buffer;

	dev = file->private_data;
	
	if ((strcfg=kmalloc(count + 1, GFP_KERNEL)) == NULL)
		return -ENOMEM;

	if (copy_from_user(strcfg, user_buffer, count)){
		retval=-EFAULT;
		goto out_error;
	}
	strcfg[count] = '\0';

	if ((buffer = kmalloc(MAX_LEN_MESSAGE, GFP_KERNEL)) == NULL) {
		retval = -ENOMEM;
		goto out_error;
	}

	if (sscanf(strcfg,"setFullColor %d:%d:%d", &r, &g, &b) == 3 
			&& (r >= 0 && r <= 255) && (g >= 0 && g <= 255) && (b >= 0 && b <= 255)) {		
		buffer[0] = 1; // ReportID
		buffer[1] = r;
		buffer[2] = g;
		buffer[3] = b;
		dataSize = 4;
		wValue = 1;
		printk(KERN_INFO ">> Full led %d %d %d", r, g, b);
	} else if (sscanf(strcfg,"setLedColor %d:%d:%d:%d", &led, &r, &g, &b) == 4 
			&& (r >= 0 && r <= 255) && (g >= 0 && g <= 255) && (b >= 0 && b <= 255) && (led >= 0 && led <= LED_SIZE)) {
		buffer[0] = 2; // ReportID
		buffer[1] = led;
		buffer[2] = r;
		buffer[3] = g;
		buffer[4] = b;
		dataSize = 5;
		wValue = 2;
		printk(KERN_INFO ">> Led %d %d %d %d", led, r, g, b);
	} else if (strstr(strcfg, "oled ") != NULL) {
		strcpy(buffer, strcfg + 5); // Remove oled command
		dataSize = count - 5;
		wValue = 3;
		printk(KERN_INFO ">> oled %s", buffer);
	} else if (sscanf(strcfg,"7s %x", &digit) == 1) {
		buffer[0] = 4; // ReportID
		buffer[1] = digit;
		dataSize = 2;
		wValue = 4;
		printk(KERN_INFO ">> 7s %x", digit);
	} else if (sscanf(strcfg,"blinkLed %d", &stop) == 1) {
		buffer[0] = 5; // ReportID
		buffer[1] = stop;
		dataSize = 2;
		wValue = 5;
		printk(KERN_INFO ">> LED %d", stop);
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
		printk(KERN_INFO ">> Buzzer %d %d", freq, stop);
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
	
	if (retval < 0 && retval != -EPIPE){
		printk(KERN_ALERT "Executed with retval=%d\n",retval);
		goto out_error;		
	}

	goto ok_path;

ok_path:
	kfree(strcfg);
	kfree(buffer);
	(*ppos)+=count;
	return count;

out_error:
	if (strcfg)
		kfree(strcfg);
	if (buffer)
		kfree(buffer);
	return retval;	
}
```



