# ¿Dónde puedo obtener la plantilla?


La plantilla puede obtenerse haciendo un fork del siguiente repositorio (opción recomendable): https://github.com/jcsaezal/plantilla-TFG-pandoc-markdown

Alternativamente, se puede descargar el siguiente archivo ZIP: <https://github.com/jcsaezal/plantilla-TFG-pandoc-markdown/archive/refs/heads/master.zip>




# ¿Cómo se compila la plantilla?

```
$ cd $RAIZ_DEL_REPO
$ panbuild PDF
```

Se genera un fichero main.pdf, con la versión en PDF de la memoria. Para que todo funcione correctamente, es preciso instalar las herramientas que se describen a continuación. 



# Instrucciones de instalación de las herramientas necesarias para trabajar con la plantilla

En Linux se ha de instalar cada una de las siguientes utilidades:

- Pandoc v2.4 (la versión específica es crítica)
- TeX live versión completa (distribución de LaTeX para GNU/Linux)
- Python 3.x, pip, y panflute 
- Herramienta Panbuild 
- Filtros (extensiones) de Pandoc
- Opcionalmente
	- Editor Typora (ojo, no es grauito)
	- Editor Sublime Text 3
	- _Plugin_ para usar Panbuild desde Sublime Text

A continuación se proporciona información sobre cómo llevar a cabo la instalación de cada herramienta sobre distribuciones de Linux como Ubuntu o Debian, que están basadas en paquetes ".deb".

## Instalación de Pandoc v2.4 

1. Descargar el paquete DEB disponible [aquí](https://github.com/jgm/pandoc/releases/download/2.4/pandoc-2.4-1-amd64.deb). 

2. Instalar pandoc ejecutando el siguiente comando desde una ventana de terminal (en el mismo directorio donde se encuentre el fichero descargado):

	```bash
	> sudo dpkg -i pandoc-2.4-1-amd64.deb
	```


## Instalación del TeX Live

Para evitar errores al generar ficheros PDF con pandoc (debido a la complejidad de las plantillas por defecto) se recomienda instalar la versión completa de TeX Live usando los siguientes comandos:

```bash
> sudo apt-get update
> sudo apt-get install texlive-full
```

## Instalación de Pip y git

Para instalar Panbuild y los filtros básicos de Pandoc que usa la plantilla, es preciso instalar las herramientas `pip` y `git` en nuestro sistema. Para ello ejecutaremos el siguiente comando:

```bash
> sudo apt-get install python-pip git
```

**Nota:** se recomienda la instalación de la extensión `pandoc-crossref` para hacer referencia cómodamente a figuras y tablas con Markdown. Usar el ejecutable descargable del siguiente enlace para instalarlo (en /usr/local/bin):

https://github.com/lierdakil/pandoc-crossref/releases/download/v0.4.0.0-alpha4/linux-ghc86-pandoc24.tar.gz

## Instalación de Panbuild y filtros (extensiones) de Pandoc

Existen dos métodos para instalar estas herramientas: (1) Mediante la creación de un virtual env de Python 3 --opción recomendable--, (2) instalando las herramientas de forma nativa en el sistema

### Método 1: Virtual env

Por comodidad, es aconsejable crear este virtual env de Python dentro del directorio principal de la plantilla. De este modo, cuando estemos editando el documento, tendremos el entorno en la misma ubicación. 

Los pasos a seguir son los siguientes:

1. Desde el directorio correspondiente (p.ej, el de la plantilla), crear virtual env (Python 3.8 o superior)

   ```bash
   > python3.8 -m venv venv3
   
   > . venv3/bin/activate
   ```

2. Instalar dependencias desde dentro del virtual env (el comando anterior debería haber modificado el prompt)

   ```bash
   > pip install --upgrade pip
   
   > pip install panflute==1.11.1 
   ```

3. Una vez hecho esto, procederemos a instalar Panbuild --que reside en un repositorio de GitHub--, como sigue:

   ```bash
   > pip install git+https://github.com/jcsaezal/panbuild
   ```

4. Para instalar los filtros básicos de Panbuild es preciso ejecutar el siguiente comando:

   ```bash
   > pip install git+https://github.com/jcsaezal/panbuild
   ```

### Método 2: Instalación nativa

1. Instalación de *panflute* 

   ```bash
   > sudo pip install panflute==1.11.1
   ```

2. Procederemos ahora a instalar Panbuild, como sigue:

   ```bash
   > sudo pip install git+https://github.com/jcsaezal/panbuild
   ```

3. Finalmente se instalan los filtros (extensiones) de Pandoc:

   ```bash
   > sudo pip install git+https://github.com/dualmarkdown/dualmarkdown
   ```

   

## Instalación del editor Sublime Text 3

Para instalar este editor pueden seguirse los pasos de cualquiera de estos tutoriales:

* [Sublime Text's official documentation](https://www.sublimetext.com/docs/3/linux_repositories.html#apt)
* [Cómo instalar Sublime Text 3 en Ubuntu y derivados](https://ubunlog.com/instalar-sublime-text-3-ubuntu/)
* [Como instalar Sublime Text 3 en Ubuntu de la manera oficial](https://ayudalinux.com/como-instalar-sublime-text-3-en-ubuntu/)

Se recomienda la instalación de la extensión "Package Control" de Sublime Text, para poder añadir fácilmente _plugins_.



## _Plugin_ de Panbuild para Sublime Text

Las instrucciones para instalar este _plugin_ de forma manual se encuentran en el siguiente enlace:

* <https://github.com/jcsaezal/SublimeText-Panbuild#installation>




# Algunos enlaces interesantes

* [Extensiones de Teaching markdown](https://dualmarkdown.github.io/documentation/)
* [Pandoc's Markdown](https://pandoc.org/MANUAL.html)
* [Documentación de Panbuild](https://github.com/jcsaezal/panbuild)

