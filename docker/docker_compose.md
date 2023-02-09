# Docker compose

La herramienta ***docker compose*** forma parte de la plataforma ***docker***. Su función es automatizar el manejo de los contenedores, de forma que con un solo comando se pueda iniciar una red de contedores (o servicios) y así mismo con un solo comando apagar dicha red.

Es aquí donde podremos por ejemplo automatizar el levantamiento de nuestros servicios, indicando el sistema operativo que va a usar cada contendor, el volumen al que va a estar conectado cada contenedor, los comandos que se van a ejecutar en cada contenedor ya sea para instalar paquetes del sistema o de nuestras utilidades como Python, también aquí se van a poner los comandos necesarios para poner a correr el servicio que corresponde a cada contenedor, entre otras cosas.

## Aclaraciones de V1 vs V2

Actualmente ya está a punto de salir la versión 2 de docker compose. Esta versión remplazará completamente a la versión 1, la cual dejará de recibir soporte en Junio del 2023, por lo que es importante usar solo la versión 2.
* Para usar la versión 1, el comando que se tiene que usar es: `docker-compose`.
* Para usar la versión 2, el comando es: `docker compose`
* Podemos forzar el uso de la versión 2 con cualquiera de los dos comandos anteriores, si marcamos la configuración `Ajustes -> Genera -> Use Docker Compose V2`.

En esta guía nos enfocaremos a usar solamente la versión 2 con el comando `docker compose`.


