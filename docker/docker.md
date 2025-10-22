# Docker

El flujo de funcionamiento de los contenedores es el siguiente:
* En un archivo llamado `Dockerfile` se definen todas las instrucciones de creación de la imagen de contenedor.
* La imagen del contendor es la que se puede montar o ejecutar en lo que se llama contenedor el cual ya es la máquina virtual.
* La ventaja de usar imágenes es que podemos levantar tantos contenedores (máquinas virtuales) como queramos basados en dicha imágen, y en el comando para levantar cada uno podemos configurar particularidades de cada uno como los puertos desde lo que se podrá comunicar con el contenedor, entre otras cosas.

Además:
* Todo lo que hagamos en el contenedor no afectará a la imagen, ya que es la que nos ayuda a reconstruir el contenedor en su forma original.
* Cada que cerramos (detenemos) el contenedor este se pierde completamente y tenemos que volver a montarlo desde la imagen, y todo lo que hayamos hecho se pierde ya que vuelve a iniciar en su forma original.
* Por eso lo recomendable es preparar todo lo esencial en el Dockerfile, además de definir un almacenamiento persistente para no perder los archivos generados durante la ejecución del contenedor.

## Contenido del Dockerfile

El archivo `Dockerfile` es donde deifniremos las instrucciones de la imagen de contenedor, es decir, ahí podremos indicar cual es el sistema operativo que usará la imagen, ejecutar comandos del sistema operativo para instalar librerías, dependencias, etc. copiar arhivos o carpetas desde el host a la imagen, entre otras cosas.

El `Dockerfile` no tiene que llamarse así necesariamente, podemos ponerle el nombre que queramos e indicarlo durante la compilación de la imagen con la opción `-f, --file`, pero si usamos `Dockerfile` como nombre entonces no tenemos que indicar la opción `-f, --file` ya que por defecto se buscará el archivo llamado `Dockerfile`.

Las instrucciones más comunes en el Dockerfile son:
* `ARG`: para especificar variables de tiempo de ejecución que pueden ser pasadas al momento de crear la imagen con el comando `docker build` en la opción `--build-arg` (esto se revisará más adelante). Si no se especifican en esta instrucción entonces no podrán ser pasadas con `--build-arg`, es una variable por comando `ARG`.
    * Sintaxis: `ARG <name>[=<default value>]`, podemos especificar un valor por defecto a usarse en caso de que no se pase la variable con `--build-arg`.
    * Ejemplo: `ARG PYTHON_VERSION=3.11`.
* `FROM`: especifica la imagen base para comenzar nuestra imagen, 
    * Sintaxis: `FROM <image>[:<tag>] [AS <name>]`, el tag corresponde generalmente a la veresión de la imagen, `latest` para usar la última versión, aunque lo recomendable es usar la versión específica para mantener la reproducibilidad.
    * Ejemplo :`FROM python:3.11-slim`.
* `WORKDIR`: define el directorio de trabajo que se usará desde esta instrucción en adelante dentro de la imagen, si el directorio no existe entonces se crea. A partir de aquí, todos los comandos que involucren rutas relativas (que no inician con `/`) dentro de la imagen iniciarán a partir del directorio especificado en esta instrucción, a menos que se usen rutas absolutas (ej. `/home`). 
    * Sintaxis: `WORKDIR /path/to/workdir`, directorio absoluto a usar.
    * Ejemplo: `WORKDIR /app`
    * `NOTA`: Lo recomendable al trabajar con comandos que copian/crean directorios o archivos es mantener una buena planeación apegándose a la convención del sistema operativo, por ejemplo usar la carpeta `/app` a nivel de raiz del sistema para el contenido de la aplicación, `/home/<usuario>` para almacenar datos específicos del usuario que está siendo usado para correr la aplicación dentro de la imagen, `/usr/local` para meter o genear nuestros propios binarios o librerías, `/data` para montar volúmenes (se verá más adelante) donde almacenar datos peristentes en tiempo de ejecución como bases de datos, logs, etc.
* `RUN`: Ejecuta comandos en una nueva capa encima de la imagen actual, comunmente para actualizar el sistema, instalar paquetes en el sistema, correr scripts de preparación.
    * Sintaxis: `RUN <command>` (shell form) o `RUN ["executable", "param1", "param2", ...]` (exec form). La forma shell ejecuta los comandos con `/bin/sh -c` y la forma exec los ejecuta directamente por lo que es más eficiente.
    * Ejemplo: `RUN apt-get update`.
    * `NOTA`: como esto genera una nueva capa, lo recomendado es ejecutar varios comandos a la vez con `&&`, por ejemplo `RUN apt-get update && apt-get install -y python3`.
* `USER`: especifica el usuario (por nombre o ID) que será usado para ejecutar comandos dentro de la instancia desde este momento en la creación de la imagen en adelante, hasta el final o hasta encontrar otro comando `USER`. Generalmente se suele crear con anterioridad al usuario dentro de la imagen con el comando `RUN useradd nuevousuario`, o podemos usar uno que sepamos que ya existe que fue creado en la imagen base. Lo recomendable es que se genere un usario no root para ejecutar la aplicación y así evitar vulnerabilidades.
    * Sintaxis: `USER <user>[:<group>]`, podemos especificar el grupo
    * Ejemplo: `USER nuevousuario`
    * `NOTA`: aquí hay que tener cuidado de asegurarnos de que los archivos que vaya a usar la aplicaición sean creados por el mismo usuario que va a ejecutar la aplicación, o se le otorgen permisos al usuario sobre los arvhicos (`RUN chown <user> <file or dir>`), para evitar conflictos de autorización. Podemos debuguear cual usuario está siendo usado en cada punto del Dockerfile con WhoAmI `RUN whoami > /user.txt`.
* `COPY`: copia archivos y directorios desde el host hasta el contenedor.
    * Sintaxis: `COPY <src_host> <dest_container>`
    * Ejemplo: `COPY requirements.txt .`, copia el archivo de requerimientos al directorio de trabajo actual.
    * `NOTA`: las rutas del host son relativas y tienen como base al directorio de contexto que se especifica al momento de crear la imagen con `docker build`. Las del destino que sean relativas se toman en base al directorio definido en el último comando `WORKDIR` (incluyendo la imagen base) o a la raiz `/` del sistema si no hay ninguno, en la imagen también podemos especificar una ruta absoluta.
* `ADD`: similar a `COPY` pero para extraer arhcivos comprimidos (`tar`, etc) en algún directorio dentro de la imagen.
    * Sintaxis: `Syntax: ADD <src> <dest>`
    * Ejemplo: `ADD myapp.tar.gz /app/`
* `ENV`: define variables de entorno que serán pasadas al contenedor.
    * Sintaxis: `ENV <key>=<value>`
    * Ejemplo: `ENV APP_ENV=production`
* `EXPOSE`: documenta cuales puertos del contenedor serán usados para escuchar, no publica el puerto hacia el host, ni mejora el rendimiento, ni cierra otros puertos, ni mejora la seguridad ni nada, solo es una forma de mantener informado a quien vaya a mantener el dockerfile de cuales puertos son usados por la aplicación y no tener que andarlos buscándo en el código para poder hacer la publicación hacia el host.
    * Sintaxis: `EXPOSE <port> [<port>/<protocol>]`
    * Ejemplo: `EXPOSE 8000`
* `CMD`: especifica el comando por defecto (aunque puede ser remplazado en el `docker run`) que será ejecutado al iniciar el contenedor, solo puede haber uno y si hay más solo se usará el último.
    * Sintaxis: `CMD ["executable", "param1", "param2"]` (exec form), `CMD command param1` (shell form), `CMD ["param1", "param2"]` (parámetros que serán agregados al comando definido en `ENTRYPOINT`).
    * Ejemplo: `CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]`
* `ENTRYPOINT`: configura al contenedor para actuar como un ejecutable en el sentido que podemos pasarle parámetros en el `docker run` y serán agregados al comando definido aquí.
    * Sintaxis: `ENTRYPOINT ["executable", "param1", "param2"]` (exec form), `ENTRYPOINT command param1` (shell form).
    * Ejemplo: `ENTRYPOINT ["python3"]`, así podríamos por ejemplo indicar cual script de Python ejectuar al levantar el contenedor mediante `docker run myimage script.py`.


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
docker run [OPCIONES] <imagen> [CMD o ARGS]
```
Donde:
* imagen: es el nombre[:tag] o el ID de la imagen. El tag es opcional ya que suele ser usado para manejar varias versiones de una misma imagen.
* `OPCIONES` más usadas:
  * `--name`: para asignar un nombre al contenedor, ej. `--name mi_contenedor`.
  * `-p, --publish`: publica o mapea puertos en el host del contenedor en formato `[ip:]puerto_host:puerto_contenedor` donde la IP es opcional, ej. `-p 8080:8000` o `-p 127.0.0.1:8080:8000`. Creo que podemos usar la IP de cualquier máquina a la que le queramos dar acceso al puerto del contenedor. Podemos mapear tantos puertos como queramos usando varias veces la opción `-p`, ej. `-p 8080:8000 -p 3036:3036`.
    * Importante: Si solo especificamos el puerto, entonces dicho puerto queda accesible para todo el mundo, lo cual es inseguro. Lo mejor es especificar la IP local para que solo el host tenga acceso al puerto del contenedor, ej. `-p 127.0.0.1:8080:8000`.
  * `-v, --volume`: monta un directorio desde el host al contenedor con el formato `ruta/host:ruta/contenedor`. La ruta del host puede ser relativa o absoluta, pero la del contenedor debe ser absoluta. Podemos montar el mismo directorio en varios contenedores para compartir almacenamiento. Al ser un volument montado, todo lo que se genere en este directorio será persistente. Lo recomendable es apegarse a la convensión de los archivos de sitema del sistema operativo, como usar `/data` para montar volúmenes donde almacenar bases de datos, logs, archivos estáticos, etc.
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
* `CMD o ARGS`: en este apartado podemos especificar un comando para remplazar el definido en `CMD` o para definir argumentos que serán añadidos al comando definido en `ENTRYPOINT` (del Dockerfile).
    * `NOTA`: esto lo opdemos aprovechar para debuguear nuestra imagen ya que podemos ejectuar el contenedor en modo `-it` con `docker run -it [OPCIONES] <imagen> /bin/bash`, o podemos ejecutar `sleep infinity` en modo `-d` con `docker run -d [OPCIONES] <imagen> sleep infinity` y luego conectarnos al contenedor con `docker exec -it <container> /bin/bash`.


## Redes

Al instalar Docker, se crearán algunas redes virtuales, podemos ver la lista de redes que tiene docker con el siguiente comando:
```powershell
docker network ls
```
Donde en una instalación limpia de Docker, deberían estar las siguientes 3 redes:
* `bridge`: de tipo bridge, es un puente intermediario entre la red del host y los contenedores que se conecten a dicha red `bridge`. Dicho puente se encarga de interconectar todas las interfaces de los contenedores que sean conectados a `bridge`. Maneja sus propias IP ya que todas las interfaces conectadas al puente están aisladas de la red del host. Cada contenedor conectado a esta red tendrá asignada su propia interfaz e IP. Todos los contenedores conectados a esta red pueden interactuar entre ellos directamente usando sus nombres o sus IP asignadas. Los contenedores que no se les asigne una red, por defecto serán conectados a `bridge`. Podemos exponer puertos de la red del contenedor a la red del host con la opción `-p` al ejecutar el comando `docker run`. En esta red cada contenedor tiene su espacio de nombres de red.
* `host`: un contenedor conectado a esta red comparte el espacio de nombres con el host, es decir, el contenedor usa directamente la red del host y por lo tanto no obtiene una interfaz ni IP únicas, sino que comparte las mismas que el host. El contenedor puede acceder directamente a servicios que estén corriendo en el host, usando la misma IP y puertos que el host. Es útil cuando queremos evitar la sobrecarga del Network Address Translation (NAT) o cuando queremos acceder a servicios que están accesibles en la red desde el host. En resumen, en la red `host`, no hay diferencia entre el host y el contenedor, en términos de la red.
* `none`:

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

