# Docker compose

La herramienta ***docker compose*** forma parte de la plataforma ***docker***. Su función es automatizar el manejo de los contenedores, de forma que con un solo comando se pueda iniciar una red de contedores (o servicios) y así mismo con un solo comando apagar dicha red.

Es aquí donde podremos por ejemplo automatizar el levantamiento de nuestros servicios, indicando el sistema operativo que va a usar cada contendor, el volumen al que va a estar conectado cada contenedor, los comandos que se van a ejecutar en cada contenedor ya sea para instalar paquetes del sistema o de nuestras utilidades como Python, también aquí se van a poner los comandos necesarios para poner a correr el servicio que corresponde a cada contenedor, entre otras cosas.

## Aclaraciones de V1 vs V2

Actualmente ya está a punto de salir la versión 2 de docker compose. Esta versión remplazará completamente a la versión 1, la cual dejará de recibir soporte en Junio del 2023, por lo que es importante usar solo la versión 2.
* Para usar la versión 1, el comando que se tiene que usar es: `docker-compose`.
* Para usar la versión 2, el comando es: `docker compose`
* Podemos forzar el uso de la versión 2 con cualquiera de los dos comandos anteriores, si marcamos la configuración `Ajustes -> General -> Use Docker Compose V2`.

En esta guía nos enfocaremos a usar solamente la versión 2 con el comando `docker compose`.

# Manejo de servicios

Todas las indicaciones para la automatización con docker compose deben ser especificadas en un archivo compose tipo [`yml`](https://en.wikipedia.org/wiki/YAML).

De preferencia nombrar el arhivo como `"docker-compose.yml"` y guardar dicho archivo en la carpeta donde se va a ejecutar docker compose, ya que es el archivo por defecto que docker compose buscará. Se puede usar un nombre y ruta diferente, pero entonces se tiene que especificar como veremos en la sección de comandos.

Para levantar los servicios especificados en un archivo compose (`"docker-compose.yml"`), podemos hacerlo mediante el comando 
```powershell
docker compose up [OPCIONES]
```
Opciones:
* `--build`: por defecto compose no vuelve a crear la imagen si ya existe, con esta opción forzamos a que se vuelva a crear desde el docker file.

Para bajar los servicios usamos:
```powershell
docker compose up [OPCIONES]
```
Opciones:
* `-v, --volumes`: para eliminar también los volúmenes creados, esto se tiene que usar con cuidado ya que lo volúmenes se usan para preservar datos aunque el contenedor y la 
imágen sean eliminados.

# Archivo compose

Un archivo compose se define mediante un archivo de tipo `yml`, usualmente `"docker-compose.yml"` y se compone principalmente por secciones, y cada sección puede contener subsecciones, etc. A continuación mostramos las principales secciones y sus estructuras:
* ```yml
  version: #la versión del formato compose a utilizar
  ```
  Se puede especificar la sub-versión que queramos y a partir de ahí se tomará la más actual, por ejemplo `3` o `3.9`, etc. Para más información de las versiones del formato compose y la versión de Docker que la soporta podemos ver [`Matriz de compatibilidad`](https://docs.docker.com/compose/compose-file/compose-versioning/#compatibility-matrix).
  Ejemplos:
  ```yml
  version: 3
  ```
* ```yml
  services:
    <nombre_servicio_1>: # por defecto este será el hostname del contenedor en las redes,
    # si queremos asignar otro hostname, tenemos que usar la configuración `hostname`
      hostname: # para especificar un hostname distinto al nombre del servicio
      build: # Si solo vamos a usar el dockerfile podemos especificar aquí la ruta 
      # donde se encuentra que será usada como directorio de contexto, pero si vamos 
      # a necesitar más configuraciones entonces tenemos que desglosarlas dentro 
      # como se muestra a continuación.
        context: # directorio de contexto a usar al momento de construir la imagen
        dockerfile: # dockerfile a utilizar para la creación de la imagen
        args: # build-args a pasar al docker file
          ARG_1: val1
          ARG_2: val2
      image: # si especificamos la sección `build` entonces esto será el 
      # `nombre[:etiqueta]` que se le asignará a la imagen al momento de 
      # crearse (por defecto se le asigna el nombre 
      # <directorio_raiz>_<nombre_servicio>), de otra forma este será el 
      # nombre de la imagen que se buscará en las imagenes ya creadas y de no encontrarse 
      # entonces se buscará en el docker-hub.
      container_name: # nombre que se le asignará al contenedor, por defecto se le
      # asignará el nombre <directorio_raiz>_<nombre_servicio>_<numero>. Donde el 
      # número parece ser un consecutivo que comienza desde 1.
      ports: # Mapeo de puertos
        - [<ip_host>:]<puerto_host>:<puerto_contenedor>
        ...
      volumes: # Volumenes a montar en el contenedor, pueden ser volúmenes vinculados
      # de directorios en el host al contenedor, volúmenes nombrados en docker, 
      # o volúmenes anónimos. Estos últimos se usan para preservar datos aunque
      # el contenedor sea removido y son manejados automáticamente por docker.
      # IMPORTANTE: podemos indicar que el volumen sea de solo lectura si concatenamos
      # `:ro` después del directorio en el contenedor
        # Volúmenes vinculados
        - <directorio_host>:<directorio_contenedor>
        # Volúmenes nombrados (deben estar definidos en la sección `volumes`)
        - <nombre_de_volumne>:<directorio_contenedor>
        # Volúmenes anónimos (solo necesitamos la ruta en el contenedor)
        - <directorio_contenedor>
        ...
      network_mode: # Si queremos usar una de las redes incluidas en docker 
      # (birdge, host, none), debemos especificarla en esta sección
      networks: # Si queremos conectarlo a redes personalizadas debemos especificarlas 
      # en esta sección, podemos conectar el contenedor más de una red:
        # Conectar a una red sin más configuraciones
        - <nombre_red>
        # Conectar a una red y además especificar más configuraciones
        <nombre_red>:
          # Para acceder al contenedor desde cada red a la que se conecta, se usa 
          # el hostname asignado, que puede ser el nombre del servicio o el valor
          # de la configuración `hostname`, pero aún así podemos asignarle uno o más
          # alias a dicho hostname en cada una de las redes de la siguiente forma
          aliases:
            - <alias_1>
            ...
          ipv4_address: # asignar IP v4 estática
          ipv6_address: # asignar IP v6 estática
          priority: # prioridad numérica de la conexión respecto a las demás redes
      env_file:
        # Archivo `*.env` con las variables de entorno que serán pasadas al 
        # contenedor. Parece que solo se puede especificar uno solo por servicio.
        - <ruta_del_archivo_env>
      secrets: # secretos a usar de los definidos en la seccíón `secrets`
        - <nombre_de_secreto>
      configs: # configuraciones a usar de las definidas en la sección `configs`
        - <nombre_configuracion>
    <nombre_servicio_2>:
      ...
  ```
* ```yml
  volumes: # Todas las propiedades de un volumen son opcionales. Los volúmenes
  # pueden reutilizarse en más de un servicio. Solo se crearán la primera vez.
    <nombre_volumen>:
      external: # Si el volumen fue creado fuera de docker compose usamos `true`
      # para que no se intente crear, si no existe se mostrará un error al ejecutar
      # `docker-compose up`. Si queremos el el volumen sea manejado por compose 
      # usamos `false`. Por defecto se usa `false`. En lugar de `true` podemos
      # simplemente definir las propiedades del volumen externo:
        name: # nombre real del volumen, aunque en el archivo compose será referenciado
        # con `<nombre_volumen>`.
      driver: # driver del volumen:
      # local: valor por defecto, para directorios locales en el host
      # nfs: para directorios en red (Network File System)
      # s3: para directorios en Amazon S3
      # azure_file: para directorios en azure
      # gcplogs: para hacer stream the logs hacia Google Cloud Platform (GCP)
      # rexray: plugin externo para habilitar integración con otros provedores
      #   en la nube.
      # IMPORTANTE: cada driver tiene su conjunto de opciones que tiene que ser
      # definidas en el apartado `driver_opts`.
      driver_opts:
        <configuracion>: <valor>
        ...
  ```
* ```yml
  networks:
    <nombre_red_1>: # Nombre de la red, las redes solo se crean la primera vez
      driver: # Driver a usar en la red (bridge, host, overlay, macvlan, and none)
      ipam: # configuración IPAM (IP Address Management) para definir rangos de IPs
      # personalizados y ajustes de Network Gateways.
        config:
          - subnet: # <subred>/<bits_de_subred> en notación CIDR ej. `172.20.0.0/16`
            gateway: # IP del gateway para la red. Esta IP en la subred que será 
            # usada para dirigir el tráfico. Si no se especifica Docker asigna la 
            # primera IP en la subred, por ejemplo, si la subred es `172.20.0.0/16`, 
            # eso quiere decir que el rango de IPs para la red será de `172.20.0.1` 
            # a `172.20.255.254`. Y la primera IP es `172.20.0.1` que es la que 
            # será asignada como gateway.
      external: # `true` o `false` para especificar si la red es externa a compose 
      # y solo se quiere reutilizar aquí
      attachable: # `true` o `false` para especificar si los contenedores de otros
      # servicios pueden conectarse a esta red o no.
  ```
* ```yml
  secrets: # es usada para definir y manejar información sensible como contraseñas,
  # llaves de APIs, certificados, etc, que necesitan ser usados dentro del los 
  # contenedores. Aquí podemos referenciar secretos definidos en el manejador de
  # secretos integrado en Docker, en Docker Swarm o en algunas herramientas de terceros.
  # Para poder usar un secreto debió haber sido definido en el provedor correspondiente.
    <nombre_de_secreto>:
      external: # `true` o `false` para especificar si el secreto es manejado 
      # externamente en Docker Swarm o en un provedor externo.
      file: # archivo donde está el secreto, el archivo solo debe contener el 
      # valor del secreto
      inline: # valor del secreto si queremos ponerlo directamente
  ```
* ```yml
  configs: # Esta sección es usada para manejar archivos de configuración para 
  # los servicios en el stack de aplicaciones.
    <nombre_configuracion>:
      external: # `true` o `false` para especificar si la configuración es externa
      file: # ruta del archivo de configuración, puede ser en formato JSON, YAML, o TOML
  ```


# Comandos

Una ves definido nuestro archivo compose `"docker-compose.yaml"`, podemos proceder a iniciarlo con el comando:
```powershell
docker compose up
```
Si nuestro archivo compose tiene otro nombre o ruta, podemos indicarlo con:
```powershell
docker compose up -f <ruta_archivo_compose>
```

Si queremos que los servicios corran de fondo, podemos usar la bandera `-d` (detached):
```powershell
docker compose up -d
```

Una vez nuestros servicios están corriendo, podemos solo detener los contendores con el comando siguiente, el cual es útil especialmente al usar la bandera `-d`:
```powershell
docker compose stop
```

O podemos detenerlos los contenedores y remover todos los recursos creados con, con `CTRL+C`, o ejecutando el comando:
```powershell
docker compose down
```

