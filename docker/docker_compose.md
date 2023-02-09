# Docker compose

La herramienta ***docker compose*** forma parte de la plataforma ***docker***. Su función es automatizar el manejo de los contenedores, de forma que con un solo comando se pueda iniciar una red de contedores (o servicios) y así mismo con un solo comando apagar dicha red.

Es aquí donde podremos por ejemplo automatizar el levantamiento de nuestros servicios, indicando el sistema operativo que va a usar cada contendor, el volumen al que va a estar conectado cada contenedor, los comandos que se van a ejecutar en cada contenedor ya sea para instalar paquetes del sistema o de nuestras utilidades como Python, también aquí se van a poner los comandos necesarios para poner a correr el servicio que corresponde a cada contenedor, entre otras cosas.

## Aclaraciones de V1 vs V2

Actualmente ya está a punto de salir la versión 2 de docker compose. Esta versión remplazará completamente a la versión 1, la cual dejará de recibir soporte en Junio del 2023, por lo que es importante usar solo la versión 2.
* Para usar la versión 1, el comando que se tiene que usar es: `docker-compose`.
* Para usar la versión 2, el comando es: `docker compose`
* Podemos forzar el uso de la versión 2 con cualquiera de los dos comandos anteriores, si marcamos la configuración `Ajustes -> General -> Use Docker Compose V2`.

En esta guía nos enfocaremos a usar solamente la versión 2 con el comando `docker compose`.

# Archivo de configuración

Todas las indicaciones para la automatización con docker compose deben ser especificadas en un archivo compose tipo [`yaml`](https://en.wikipedia.org/wiki/YAML).

De preferencia nombrar el arhivo como `"docker-compose.yaml"` y guardar dicho archivo en la carpeta donde se va a ejecutar docker compose, ya que es el archivo por defecto que docker compose buscará. Se puede usar un nombre y ruta diferente, pero entonces se tiene que especificar como veremos en la sección de comandos.

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

