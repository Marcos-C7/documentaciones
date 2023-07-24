# Docker

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

Para esto usamos el comando 
```powershell
docker build [OPCIONES] PATH|URL|-
```

Donde:
* `PATH|URL|-`: es el `directorio de contexto`, todo su contenido será accesible directamente durante el proceso de creación. Si no se indica una ruta para el archivo dockerfile, entonces se buscará un archivo llamado `'Dockerfile'` en este directorio. Este directorio puede ser la ruta de una carpeta del sistema, `'.'` indica la carpeta actual, puede ser la URL de un repositorio Git, o puede ser `'-'` para indicar un archivo `tar` pasado via `STDIN`.
* `OPCIONES`: las opciones más usadas son:
  * `-t, --tag`: para asignar un nombre y una etiqueta opcional a la imagen, por ejemplo `-t mi_app:latest`, o también `-t mi_app:2.0`.
  * `-f, --file`: para indicar la ruta de un archivo dockerfile específico.
  * `--build-arg`: para definir variables que podrán ser usadas en el archivo dockerfile, con el formato `KEY=VALUE`. Por ejemplo `--build-arg PYVERSION=3.11`.
  * `--no-cache`: para no construir desde cero sin usar el cache.
  * `--pull`: forzar a jalar siempre la versión más reciente de la imagen base.
  * `--progress`: tipo de informe de progreso `auto, plain, tty o json`.

Notas:
* `directorio de contexto`:
  * Es ideal solo mantener los archivos necesarios en el `directorio de contexto`, ya que todo su contenido es enviado al docker deamon para que estén accesibles durante la creación de la imagen. Al referenciar un archivo dentro del dockerfile, su ruta debe ser relativa respexto al directorio de contexto.
  * Por defecto algunos archivos son excluidos del directorio de contexto, por ejemplo los archivos o carpetas que comiencen con un punto como `.git` y todos los que especifiquemos en el archivo `.dockerignore`.
  * No incluir archivos con datos sensibles ya que pueden ser copiados o a la imagen o expuestos durante la creación de la imagen.
  * El archivo `.dockerignore` debe ser puesto al lado del archivo dockerfile, ya sea que este último se encuentre dentro del directorio de contexto o no.
* `build-arg`: 
  * Las variables definidas mediante esta opción se pueden usar dentro del archivo dockerfile mediante la sintaxis `${VARIABLE}`, lo que hayamos definido en la variable se remplazará, por ejemplo al definir la imagen base `FROM python:${PYVERSION}`, así podemos indicar desde fuera del dockerfile qué versión de Python queremos usar. 
  * También podemos indicar un valor por defecto en el dockerfile, en caso de que la variable no sea definida, mediante `${VARIABLE:-VALOR_DEFECTO}`, notemos que hay un guión antes del valor por defecto, por ejemplo `FROM python:${PYVERSION:-3.10}`.
  * Si quisieramos manejar los `build-args` en un archivo separado, podríamos definirlos en un archivo digamos `build-args.txt` uno en cada linea con el formato `KEY=VALUE`, y después indicar dicho arhcivo mediante `--build-arg $(cat build-args.txt)`.

## Creación de un contenedor desde una imagen

Una vez tenemos creada nuestra imagen, podremos generar contenedores o instancias de la imagen. Para revisar los contenedores existentes usamos el commando:
```powershell
docker images
docker images -a
```
Para ver las opciones disponibles:
```powershell
docker images --help
```

Con el comando `docker images` podremos ver los datos de la imagen, en particular el Nombre, el tag y el ID. Así podremos ejecutar un contenedor mediante el comando siguiente:
```powershell
docker run [OPCIONES] <imagen>
```
Donde:
* imagen: es el nombre[:tag] o el ID de la imagen. El tag es opcional ya que suele ser usado para manejar varias versiones de una misma imagen.
* `OPCIONES` más usadas:
  * `--name`: para asignar un nombre al contenedor, ej. `--name mi_contenedor`.
  * `-p, --publish`: publica o mapea puertos en el host del contenedor en formato `puerto_host:puerto_contenedor`, ej. `-p 8080:8000`.
    * Importante: Si solo especificamos el puerto, entonces dicho puerto queda accesible para todo el mundo, lo cual es inseguro. Lo mejor es especificar la IP local para que solo el host tenga acceso al puerto del contenedor, ej. `-p 127.0.0.1:8080:8000`.
  * `-v, --volume`: monta un directorio o archivo desde el host al contenedor con el formato `ruta/host:ruta/contenedor`. La ruta del contenedor debe ser absoluta.
  * `--rm`: eliminar el contenedor automáticamente cuando sea detenido.
  * `-e, --env`: establecer variables de entorno dentro del contenedor en formato `KEY=VALUE`.
  * `--env-file`: para indicar las variables de entorno que serán pasadas al contenedor, desde un archivo, con cada variable en una linea con el formato `KEY=VALUE`, ej. `--env-file ruta/env_file.env`.
* `OPCIONES` de límites de recursos:
  * `--cpus`: para indicar cuantos CPUs puede usar el contenedor, ej. `--cpus=2`.
  * `--cpu-shares`: indicar un peso relativo de tiempo de CPU con respecto a otros contenedores, ej. `--cpu-shares=350`.
  * `-m, --memory`: indicar límite de uso de RAM en `GB` o `MB`, ej. `--memory 128MB` o `--memory 2GB`.
  * `--memory-swap`: límite total de memoria incluyendo la swap, por defecto es el doble de la asignada con `--memory`. Se puede saignar `-1` para nque no haya límite.
  * `--gpus`: permitir el uso de GPUs NVIDIA (ver [gpus](https://docs.docker.com/engine/reference/commandline/run/#gpus)).
* `OPCIONES` de redes:
  * `--network`: conectar el contenedor a una red (previamente creada con `docker network create`), ej. `--network=mi_red`.
  * `--ip, --ip6`: para asignar una IP estática al contenedor, la cual puede ser v4 con `--ip` o v6 con `--ip6`, ej. `--ip=192.10.50.10`.
* `OPCIONES` de modo de ejecución:
  * `-d, --detach` (detach o deamon): ejecuta el contenedor en segundo plano (modo deamon), el problema es que si no se queda corriendo ningún proceso entonces el contenedor se dentendrá apenas sea creado, lo cual no nos sirve.
  * `-t` (terminal): asigna una terminal pseudo-TTY (Tele-Typewriter) lo que permite tener interacción de entrada y salida con el contenedor, es decir, que podemos conectarnos al contenedor para ejecutar comandos.
  * `-t -d` (terminal y detach): si queremos que el contenedor quede corriendo para poder conectarnos aunque dentro del contenedor no haya quedado ningún proceso corriendo, entonces usamos ambas opciones `-t` y `-d`. Si se queda un proceso corriendo, como una app web, etc, es suficiente con la opción `-d`.
  * `-i` (interactivo): mantiene abierta una conexión de entrada estandar (STDIN) aunque el contenedor no esté conectado a una terminal.
  * `-it` (interactivo y terminal): combina las opciones `-i` y `-t`, por lo que al crear el contendor la terminal queda conectada al contenedor y el comando especificado en la instrucción `CMD` del Dockerfile se ejecuta. Al desconectar la terminal del contenedor, si no se queda ningún proceso corriendo en el contenedor entonces este se detendrá automáticamente.

## Redes

Al instalar Docker, se creará una red virtual tipo bridge llamada `bridge`. Podemos ver la lista de redes que tiene docker con el siguiente comando:
```powershell
docker network ls
```
Donde en una instalación limpra de Docker, deberían estar 3 redes:
* `bridge`: de tipo bridge, es un puente intermediario entre la red del host y la red del contenedor, dicho puente se encarga de interconectar todas las interfaces que están conectadas a esta. Maneja sus propias IP ya que  todas las interfaces conectadas al puente están aisladas de la red del host. Cada contenedor conectado a esta red tendrá asignado su propia interfaz e IP. Todos los contenedores conectados a esta red pueden interactuar entre ellos directamente usando sus nombres o sus IP asignadas.
* `host`: 
 la cual podemos ver  llamada `docker0`, la cual podemos ver cone el siguiente comando en Ubuntu:
```powershell
ip address show
```



Cuando creamos un contenedor, Docker crea una Intefaz Ethernet Virtual asignada a dicho contenedor, la cual podemos ver con el mismo comando `ip address show`. Dicha red virtual será conectada a un un switch que dependerá del tipo de red que le hayamos asignado al contenedor, por defecto 


Ver cuales 

## Administrar imágenes

Para revisar la lista de imágenes creadas usamos el siguiente comando que nos mostrará las imágenes y sus propiedades:
```powershell
docker images
```

Para eliminar una imagen primero tenemos que asegurarnos de que no tiene contenedores existentes, y entonces podemos ejecutar alguno de los siguientes comandos:
```powershell
docker image rm <imagen>
docker rmi <imagen>
```

Suele ocurrir que cuando volvemos a construir una imágen, la antigua no se elimna sino que solo se le quita el nombre y el ID pero sigue existiendo, y eso hace que terminemos con una larga lista de imágenes sin nombre (`dangling images`). Para eliminar todas las imágenes colgantes, usamos el siguiente comando:
```powershell
docker image prune [OPCIONES]
```
Opciones:
* `-f, --force`: por defecto se nos pedirá confirmación, al usar esta opción no se pedirá confirmación.
* `-a, --all`: por defecto solo se eliminan las imágenes sin nombre, al usar este comando se eliminarán también las imágenes que no tengan ningún contenedor asignado.

Podemos escanear las imágenes para buscar vulnerabilidades de una base de datos en linea desde un servicio llamado `snyk`. Necesitamos tener cuenta en Docker-hub y además estar logeados:
* Para logearnos desde la terminal:
  ```powershell
  docker login
  ```
* Para escanear:
  ```powershell
  docker scan [OPTIONS] <image_name>
  ```
NOTA: Podemos escanear incluso las imágenes que están en Docker-hub, como las imágenes base que queremos usar.

## Administrar contenedores

Una vez un creado un contenedor, podemos manejarlo con los siguientes comandos:
* `docker stop <contenedor>`: para detener un contenedor.
* `docker start <contenedor>`: para volver a iniciar un contenedor.
* `docker restart <contenedor>`:
* `docker rm <contenedor>`: para eliminar un contenedor que esté detenido.


Salir de un contedor en modo consola:
```powershell
exit
```

## Administrar volúmenes

## Comandos


Mostrar los contenedores que etán corriendo:
```powershell
docker ps
```

Entrar a un contenedor que está corriendo, en modo consola:
```powershell
docker exec -it <nombre_o_ID> <consola>
```
* `<nombre_o_ID>`: nombre o ID del contedor, que puede ser consultado con le comando `docker ps`.
* `<consola>`: nombre de la consola a usar, depende del sistema operativo del contenedor, unos usan `powershell`, otros `sh`, etc.


Para detener un contenedor:
```powershell
docker stop <nombre_o_ID>
```

Para iniciar un contenedor:
```powershell
docker start <nombre_o_ID>
```

Para monitorear el consumo de recursos de los contenedores que están corriendo:
```powershell
docker stats
```

Eliminar un contenedor (tuvo que haber sido detenido previamente):
```powershell
docker rm <nombre_o_ID>
```

