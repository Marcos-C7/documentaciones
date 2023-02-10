## Introducción

El flujo de funcionamiento de los contenedores es el siguiente:
* En un archivo llamado `Dockerfile` se definen todas las instrucciones 
  de creación del contenedor.
* A partir del Dockerfile se crea una imagen de contenedor.
* La imagen del contendor es la que se puede montar para dar forma al
  contenedor para ya poder trabajar sobre este. 

Todo lo que hagamos en el contenedor no afectará a la imagen, ya que
es la que nos ayuda a reconstruir el contenedor en su forma original.
Porque cada que cerramos (detenemos) el contenedor este se pierde 
completamente y tenemos que volver a montarlo desde la imagen, y todo
lo que hayamos hecho se pierde ya que vuelve a iniciar en su forma original.
Por eso lo recomendable es preparar todo lo esencial en el Dockerfile, además
de definir un almacenamiento persistente para no perder los archivos generados
en el contenedor.

## Contenido del Dockerfile

* El archivo `Dockerfile` básicamente contiene las siguientes indicaciones:
  * La instrucción `FROM` indica que hereda desde una imágen base, de las 
    cuales hay muchas implementadas, incluso hay plataformas que proveen 
    imágenes base desde las cuales heredar. Por lo general la imagen base
    es una que contiene el sistema operativo que necesitamos.
  * Las instrucciones 

## Creación de una imagen de contenedor

Una vez está listo el archivo Docker file con las instrucciones de creación
del contenedor, se tiene que crear una imagen desde la cual se puede ejecutar
el contenedor.