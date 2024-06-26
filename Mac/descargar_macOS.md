Descarga de imágenes del sistema macOS, distintas versiones
------------------------------------------------------------

Hay una herramienta que contiene un script de Python para descar las imágenes de macOS de distintas versiones. La herramienta es [OpenCorePkg](https://github.com/acidanthera/OpenCorePkg). En particular el script que nos interesa está en la subcarpeta `Utilities\macrecovery\`. El script se llama `macrecovery.py` y en el archivo `recovery_urls.txt` están los comandos que tendríamos que ejecutar para descargar la versión que necesitamos.

Solo hay que descargar la carpeta `OpenCorePkg\Utilities\macrecovery\`, pero yo no vi como descargar solo esa carpeta, así que se podrían descargar solo los archvos `macrecovery.py` y `recovery_urls.txt`, o descargar todo el repositorio y quedarnos con dicha subcarpeta.

Para poder ejecutar el script necesitamos tener instalado `Python 3`.
Una vez descargado el Script, solo hay que ejecutarlo con Python desde una terminal usando el comando correspondiente a la versión que queremos pero agregando el parámetro `download`, recordemos que el catálogo de comandos está en `recovery_urls.txt`. Por ejemplo, para descargar la versión `Ventura`, que es la `macOS 13`, ejecutaríamos el comando siguiente:
```
> python ./macrecovery.py -b Mac-B4831CEBD52A0C4C -m 00000000000000000 download
```

En el siguiente enlace de Wikipedia podemos encontrar el historial de versiones de [MacOS](https://en.wikipedia.org/wiki/MacOS).

Una vez descargado, el archivo lo encontraremos en el directorio `\com.apple.recovery.boot\BaseSystem.dmg`.

### Convertir el archivo `dmg` a un archivo `img`

Para esto se tiene que usar un programa llamado `dmg2img`, existe una versión para Windows en ejecutable, pero también se puede usar WSL con alguna distribución como Ubuntu:
* En Windows, el programa se descarga desde `http://vu1tur.eu.org/tools/`.
* En Ubuntu lo instalamos con `sudo apt-get install dmg2img`.

Una vez descargado el programa en Windows o instalado en Ubuntu, lo ejecutamos con el siguiente comando:
```
> dmg2img <archivo_entrada_dmg> [archivo_salida_img]
```
Si no podemos un archivo de salida, se creará un archivo `img` con el mismo nombre.
En el caso de Windows, tenemos que usar la ruta completa de donde pusimos el programa, o podríamos agregar la ruta al Path o poner el ejecutable al lado del archivo `dmg`.

