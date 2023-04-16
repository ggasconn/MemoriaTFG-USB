<!-- Leave a blank line before the title -->

# Drivers desarrollados en el proyecto

En este capítulo se describe los drivers propuestos para la interacción con los periféricos conectados a la placa, además de los detalles de bajo nivel como los endpoints utilizados en cada uno.


## BasicInterrupt - Driver asíncrono con comunicación INTERRUPT IN

### Descripción del funcionamiento 

Este módulo del kernel implementa un driver USB utilizando el endpoint de tipo IN del dispositivo USB, así como la API asíncrona del kernel. [@drivers-linux]

El funcionamiento del módulo es muy sencillo, una vez cargado en el kernel, expone un dispositivo de caracteres que tiene implementada la operación de lectura. Por ello, si lanzamos una llamada `read()`, por ejemplo con `cat`, iniciaremos la rutina que envía el URB de tipo *INTERRUPT IN* al dispositivo pidiéndole que le rellene el buffer enviado con datos.

Como se usa la API asíncrona de USB la llamada a `read()` devuelve siempre 0 bytes, ya que no se sabe con exactitud cuándo se va a devolver el URB relleno con datos y al ser una función no bloqueante no esperamos a que llegue. El resultado de esta solicitud de datos al dispositivo se mostrará a través de la función de callback que se invoca una vez el URB vuelve desde el dispositivo.

### Aspectos relevantes del código desarrollado
Función callback del Driver, se ejecuta cuando se devuelven datos en un URB de tipo INT para procesarlo (después de haber reservado memoria para el URB, y que éste haya sido enviado con `usb_submit_urb`:
```C
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



Función open (ejecutada nada más cargar el módulo en el kernel), se inicializa un URB de tipo INT en `usb_fill_int_urb` inicial y se envía con la función `usb_submit_urb`:

```C
/* Called when a user program invokes the open() system call on the device */
static int pwnedDevice_open(struct inode *inode, struct file *file)
{
	struct usb_pwnedDevice *dev;
	struct usb_interface *interface;
	int subminor;
	int retval = 0;

	subminor = iminor(inode);
	
	/* Obtain reference to USB interface from minor number */
	interface = usb_find_interface(&pwnedDevice_driver, subminor);
	if (!interface) {
		pr_err("%s - error, can't find device for minor %d\n",
			__func__, subminor);
		return -ENODEV;
	}

	/* Obtain driver data associated with the USB interface */
	dev = usb_get_intfdata(interface);
	if (!dev)
		return -ENODEV;

	/* Initialize URB */
	usb_fill_int_urb(dev->int_in_urb, dev->udev,
					usb_rcvintpipe(dev->udev,
								dev->int_in_endpoint->bEndpointAddress),
					dev->int_in_buffer,
					le16_to_cpu(0x0008),
					pwnedDevice_int_in_callback,
					dev,
					dev->int_in_endpoint->bInterval);
	retval = usb_submit_urb(dev->int_in_urb, GFP_KERNEL);

	/* increment our usage count for the device */
	kref_get(&dev->kref);

	/* save our object in the file's private structure */
	file->private_data = dev;

	return retval;
}
```



Función read() ejecutada cuando se hace un cat en el fichero de  /dev (se puede observar que se envía un URB tipo INT con datos al dispositivo):

```C
#define MAX_LEN_MSG 35
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



Función probe() que se ejecuta cuando se conecta al puerto USB un dispositivo de tipo `pwnedDevicestick` (se reserva memoria inicial para el URB, que posteriormente será rellenado y enviado):

El punto donde se reserva memoria para el URB sin datos es el siguiente:

```C 
dev->int_in_urb = usb_alloc_urb(0, GFP_KERNEL);
```

Ésta es la función del Driver completa:

```C
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



### Uso

Para usar este driver tan solo hay que compilarlo y cargarlo en el kernel.

```bash
make
sudo insmod BasicInterrupt.ko
```

Una vez cargado en el kernel, podemos ver en kern.log que ya está disponible y si tenemos conectado el dispositivo USB también será reconocido.

![Salida dmesg. Muestra la carga del módulo y el dispositivo reconocido](img/cargaBasicInterrupt.png)

Y si ejecutamos una llamada a `read()`, por ejemplo usando el comando `cat`, veremos la información que devuelve el dispositivo.

![Salida dmesg. Muestra la carga del módulo y el dispositivo reconocido](img/SalidaBasicDriver.png)
