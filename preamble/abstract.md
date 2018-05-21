<!-- Leave a blank line before the title -->

# Resumen {.unnumbered}

Los procesadores multicore integran en un mismo chip múltiples núcleos de procesamiento 
que co mparten recursos, como niveles de la jerarquía cache o el controlador de memoria. 
Sin embargo, el hardware por sí mismo no otorga a las aplicaciones una fracción de los 
recursos compartidos proporcional a la prioridad que establece el usuario. Esto supone un 
serio problema para el sistema operativo, ya que la contención por recursos compartidos 
puede afectar muy negativamente a la calidad del servicio que el sistema ofrece al usuario. 
Para mitigar este problema, los procesadores más recientes de Intel, como 
los de la famila Xeon E5-v4, incoporan la tecnología Intel CAT (Cache Allocation Technology), que permite 
que el sistema operativo asigne espacio en cache a las aplicaciones en base a sus características y prioridad. 
El objetivo de este proyecto es incluir el soporte necesario en el planificador de Linux para explotar el potencial de Intel CAT 
y de otras tecnologías relacionadas como Intel Cache Monitoring (CMT) e Intel Memory Bandwidth Monitoring  (MBM)

**palabras clave**: Intel MBM, Intel CAT, planificación, kernel Linux, procesadores Multicore



# Abstract {.unnumbered}

Blah Blah ...
