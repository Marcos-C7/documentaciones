# Windows Sandbox (Espacio Aislado de Windows)

El Windows Sandbox es una herramienta itegrada en el sistema Windows que funciona como un mecanismo de virtualización de máquinas Windows dentro del mismo sistema Windows.

Basicamente es un mini-Windows limpio, temporal y aislado que corre dentro de tu Windows

Es muy útil para probar aplicaciones en un entorno completamente aislado que al cerrarse se eliminará por completo.

Algunas de los elementos que podemos probar en dicho entorno aislado son los siguientes:

* Probar como funciona una aplicación que tiene riesgo de corromper algún componente importante del sistema.
* Probar que los paquetes de instalación de nuestras aplicaciones funcionen correctamente:
    * Probar que queden bien agregados los registros del sistema.
    * Que se generen correctamente las variables entorno necesarias por la aplicación.
    * Que se generen bien los accesos directos.
* Ejecutar un `.exe` sospechoso que hayamos bajado de internet.
* Instalar un software que queremos probar pero no dejar instalado en nuestro sistema.


Las ventajas respecto a otros sistemas de vistualización son:

* Arranca muy rápido.
* Requiere muy poca RAM y espacio en disco.
* Viene integrado en el sistema, no hay que instalar nada, solamente habilitarlo.
* Funciona de manera programática la cual se configura mediante un archivo XML con un amplio soporte de instrucciones como:
    * Copiar carpetas del host al Sanbox.
    * Abrir instancias del explorador de archivos en ciertos directorios.
    * Abrir cualquier aplicación.
    * Entre otras.

Cuando se ejecuta un Sandbox:

* Windows crea una instancia ligera virtualizada.
* Usa una copia limpia de tu propio sistema.
* Aísla:
    * Sistema de archivos
    * Registro
    * Procesos
    * Red
* Corre dentro de un entorno separado por Hyper-V.

**Nota**: Sandbox no es un entorno de análisis forense extremo, para estudiar malware avanzado mejor usar una VM dedicada sin red.

## Habilitación

* Primero guardar y cerrar cualquier aplicación ya que hay que reiniciar después de habilitar la herramienta.
* Si no tenemos habilitada la virtualización desde Bios/UEFI, hay que habilitarla, para esto tenemos que buscar de acuerdo a nuestra placa base, pero casi siempre es mediante el apartado de CPU/Procesador.
* Para habilitar esta herramienta simplemente abrimos `Activar o desactivar las características de Windows`.
* Buscar el elemento `Espacio aislado de Windows` y marcar la casilla correspondiente.
* Aceptar y al finalizar reiniciar el equipo.

## Ejemplo básico

Para hacer uso de Windows Sandbox solo tenemos que configurar un archivo `.wsb` y ejecutarlo. A continuación tenemos una configuración básica que simplemente copia el conteniso de un directorio desde el host hacia el Sandbox:

```xml
<!-- app-sandbox.wsb -->
<Configuration>
  <MappedFolders>
    <MappedFolder>
      <HostFolder>C:\Users\miusuario\app</HostFolder>
      <SandboxFolder>C:\app</SandboxFolder>
    </MappedFolder>
  </MappedFolders>
</Configuration>
```

Dicho ejemplo simplemente copia la carpeta `C:\Users\miusuario\app` del host en la carpeta `C:\app` del Sandbox. Simplemente ejecutamos el archivo que contiene la configuración:

```bash
./app-sandbox.wsb
```

Al ejecutarlo se abrirá una ventana con una máquina virtual de Windows completamente ligera y limpia, como recién instalado, donde podemos hacer lo que queramos y al final solo cerrar la ventana para eliminar todo. Cuando vovamos a ejecutar el Sandbox iniciará desde cero nuevamente.


## Configuraciones

En esta sección se presentará un catálogo de configuraciones válidad para un archivo `.wsb`. Con fines de facilitar el seguimiento de las configuraciones, en cada código de ejemplo podremos toda la linea de ancestros desde la configuración en cuestión hasta el elemento raiz, esto para saber exactamente donde debe ir insertada dicha configuración.

### \<MappedFolders\>

La configuración `<MappedFolders>` es para definir cuales folders serán copiados desde el host al Sandbox. 

La estructura general es la siguiente:
```xml
<Configuration>
  <!-- Declaración de la sección de cipiado de folders -->
  <MappedFolders>
    <!-- Sección correspondiente a un folder -->
    <MappedFolder>
      <!-- Ruta del folder origne en el host (relativa o absoluta) -->
      <HostFolder>{ruta-host}</HostFolder>
      <!-- Ruta destino en el sandbox (absoluta) -->
      <SandboxFolder>{ruta-sandbox}</SandboxFolder>
    </MappedFolder>
  </MappedFolders>
</Configuration>
```

### \<LogonCommand\>

La configuración `<LogonCommand>` permite ejecutar un solo comando al iniciar el Sandbox. Puede ser un comando regular o si requerimos la ejecución de varios comandos, podemos usar `<MappedFolders>` para copiar un script dentro del Sandbox y ejecutar dicho script con `<LogonCommand>`.

La sintaxis es:
```xml
<Configuration>
  <LogonCommand>
    <Command>{comando}</Command>
  </LogonCommand>
</Configuration>
```

`Ejemplo`: abrir el explorador de archivos en cierto directorio
```xml
<Configuration>
  <LogonCommand>
    <Command>explorer.exe c:\app</Command>
  </LogonCommand>
</Configuration>
```

`Ejemplo`: ejecutar un archivo de comandos `.cmd`
```xml
<Configuration>
  <MappedFolders>
    <MappedFolder>
      <HostFolder>C:\Users\miusuario\app\comandos</HostFolder>
      <SandboxFolder>C:\app\comandos</SandboxFolder>
    </MappedFolder>
  </MappedFolders>
  <LogonCommand>
    <Command>c:\app\comandos\comandos.cmd</Command>
  </LogonCommand>
</Configuration>
```

### \<Networking\>

La configuración `<Networking>` permite habilitar/deshabilitar el acceso a la red.

La sintaxis es:
```xml
<Configuration>
  <Networking>{valor}</Networking>
</Configuration>
```

Los valores posibles son:

* Default: Tiene red NAT
* Disable: Totalmente aislado

### \<VGpu\>

La configuración `<VGpu>` controla el uso de aceleración GPU.

La sintaxis es:
```xml
<Configuration>
  <VGpu>{valor}</VGpu>
</Configuration>
```

Los valores posibles son:

* Default: permite aceleración GPU
* Disable: deshabilita aceleración GPU

### \<AudioInput\>

La configuración `<AudioInput>` controla el uso de micrófono.

La sintaxis es:
```xml
<Configuration>
  <AudioInput>{valor}</AudioInput>
</Configuration>
```

Los valores posibles son:

* Default: permite acceso al micrófono
* Disable: deshabilita acceso al micrófono

### \<VideoInput\>

La configuración `<VideoInput>` controla el uso de cámara.

La sintaxis es:
```xml
<Configuration>
  <VideoInput>{valor}</VideoInput>
</Configuration>
```

Los valores posibles son:

* Default: permite acceso al cámara
* Disable: deshabilita acceso al cámara


