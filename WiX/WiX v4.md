# WiX v6

WiX es una herramienta moderna para generar instaladores para aplicaciones en Windows. A partir de la versión `v4` se marca un cambio drástico respecto a la versión `v3`.

El primer cambio es el método de instalación, la versión `v3` se distribuye mediante un instalador `.exe`, mientras que a partir de la versión `v4` en adelante, se distribuye como una herramienta `.NET`, así que necesitamos el `.NET SDK` para poder instalarlo. 

Actualmente la versión más nueva es la `v6`, aún no hay mucho soporte de la comunidad y es demasiado dificil implementar algo con la documentación oficial si no eres un usuario avanzado, pero eso debería cambiar con el tiempo. 

La página oficial de la documentación es [https://docs.firegiant.com/wix/](), y la referencia está en [https://docs.firegiant.com/wix/schema/wxs/]().

## Instalación

A partir de `WiX 4`, la instalación es atravéz de `.NET`, por eso necesitamos instalar este primero.

### Instalar .NET SDK

Descargar el instalador de `.NET SDK` (no el `.NET Runtime`) desde [https://dotnet.microsoft.com/es-es/download]() e instalarlo.

Reiniciar la terminal o aplicación desde la que se va a usar `dotnet`, para validar que se instaló bien con el siguiente comando:
```bash
dotnet --version
```

Si el comando da algún error, entonces no se instaló correctamente o se instaló la herramienta equivocada, revisar que se haya instaldo el `SDK` y no el `Runtime`. O reiniciar la aplicación desde la cual se pretende usar `dotnet` (terminal, VS Code, etc), algunas aplicaciones solo se reinican al cerrar todas sus ventanas.

### Instalar WiX

Con el siguiente comando podemos instalar la versión de nuestra preferencia, como esta documentación está basada en la versión `v6`, entonces usamos el siguiente comando:
```bash
dotnet tool install --global wix --version 6.*
```

Ya que el comando `dotnet tool install --global wix` instalará la versión más nueva y de existir alguna es probable que contenga cambios drásticos respecto a la versión `v6`.

Podemos revisar la versión instalada con el siguiente comando (posiblemente se necesite reiniciar la terminal o aplicación):
```bash
wix --version
```

Podemos desinstalar WiX con:
```bash
dotnet tool uninstall --global wix
```

Con el siguiente comando podemos ver la ayuda, la cual muestra la herramientas que vienen integradas:

```bash
wix --help
```

## Teoría

WiX es una herramienta para generar instaladores tipo `.msi` (MicroSoft Installer) que son los preferidos por Microsoft en Windows. 

Estos instaladores sin simplemente una base de datos empaquetada en un archivo `.msi` que incorpora todos los archivos, reglas, configuraciones, etc de la instalación. Este archivo es procesado por el **Windows Installer Engine** para realizar la instalación. Este mecanismo es el más adecuado en Windows porque el engine de instalación trabaja en conjunto con el sistema para poder llevar un rastro muy puntual de cada archivo, registro, variable de entorno, acceso directo, etc que se produce durante la instalación, de esta forma la desinstalación/actualización/reparación se lleva a cabo con mucha precisión.

WiX no es el engine de instalación, solo es una herramienta que facilita la generación de instaladores MSI mediante una especificación basada en archivos XML donde se define todo lo relacionado a la instalación de una aplicación. WiX procesa esos archivos XML y genera un instalador `.msi`, es decir, solo genera la base de datos. Todo bajo los estándares más modernos de instalación para Windows.

Un MSI internamente tiene:

* Tablas (Directory, Component, Feature, File, etc.)
* Relaciones entre tablas
* Secuencia de acciones (InstallExecuteSequence)
* Propiedades (variables del instalador)

WiX otorga una forma más cómoda de definir esas tablas.

### Arquitectura de un proyecto WiX

La especificación en WiX es completamente modular y bien separada, ya que cada archivo XML se encarga de una parte, lo cual permite mantener una separación muy útil de los distintos componentes de instalación como veremos más adelante. 

Aunque algunos archivos tienen la misma extensión, WiX distingue cada uno no por el nombre sino por el elemento raiz de cada archivo XML, entre otras cosas.

Antes de entrar en la arquitectura de un proyecto WiX, tenemos que entender muy bien el proceso de instalación de una aplicación:
* Regularmente durante la instalación se nos muestra una ventana que va desplegando distintas páginas de configuración, por ejemplo elegir la carpeta de instalación, elegir si correr el software al iniciar el sistema, elegir si crear un acceso directo en el escritorio, etc. Pero lo importante es que este proceso no instala nada, simplemente recolecta la confiuración.
* Una vez que se termina la configuración, entonces se ejecuta el proceso de instalación, el cual se realiza con las configuraciones elegidas por el usuario en la UI.

Entender esta separación nos ayudará a entender mejor la arquitectura de WiX. Esta separación es de utilidad en la práctica ya que podemos ejecutar el instalador resultante en modo silencioso, suprimiendo la UI, con un comando donde las configuraciones se dan como parámetros o usar los valores configurados por defecto. Así podemos usar el mismo instalador en modo silencioso para distribucion masiva sin tener que implementar otro aparte.

La arquitectura básica de un proyecto WiX consiste en los siguientes archivos, aunque en realidad los nombres de los archivos no importan ya que las reglas de WiX le permiten identificar cada cosa:
* Un archivo `.wixproj`: normalmente llamado `<app-name>Installer.wixproj`, que contiene las especificaciones generales del proyecto, como las extensiones (algo así como librerías) que serán usadas en el proyecto, la versión de WiX que será usada, enlace hacia los archivos que componen el proyecto, entre otras cosas, pero generalmente es muy pequeño en comparación con los otros archivos. En cierta forma WiX puede compilar todo correctamente porque este archivo los incluye.
* `Product.wxs`: aunque el nombre puede ser cualquiera, pero su elemento raiz es `<wix>`. En este archivo se describe unicamente el proceso de instalación, es decir, asumiendo que ya se tienen las configuraciones de instalación, se describe el proceso de instalación, obviamente soporta condicionales, aquí se pueden definir los valores por defecto de las configuraciones.
* `UI.wxs`: aunque el nombre puede ser cualquiera, su elemento raiz es `<UI>`. En este archivo se implementa la interfaz de usuario, que como vimos anteriormente puede ser suprimida al momento de ejecutar el instalador mediante comando. La interfaz de usuario es para dare la posibilidad al usuario de configurar la instalación.
* `CustomActions.wxs`: aunque el nombre puede ser cualquiera, su elemento raiz es `<<<<FALTA>>>>`. Contiene la implementación de lógica personalizada para por ejemplo consultar propiedades del sistema donde ese está instalando la aplicación, etc.
* Archivos `.wxl` de localización: su elemento raiz es `<WixLocalization>`, en este se implementan los textos de localización (traducción) para los mensajes mostrados durante la instalación.

## Extensiones

Para expandir el funcionamiento de WiX, este permite la integración de extensiones, algunas proveen funcionamiento extra (como detectar la arquitectura del procesador donde se está instalando la aplicación, etc), otras proveen interfaces gráficas, etc. 

WiX tiene su catálogo oficial de extensiones el cual podemos encontrar en [https://www.nuget.org/packages?q=wixtoolset+wixext](), para usar una solamente debemos abrir el perfil de la extensión para ver el comando de instalación, el tenemos que ejecutar en una terminal que esté posicionada en la carpeta que contiene los archivos WiX, ya que lo único que hace es editarlos agregando de manera correcta los elementos que importan la extensión. Aunque en la misma página de la extensión suele estar lo que tenemos que agregar manualmente si no queremos usar el comando.

Algunas de las extensiones más conocidas son:

* `WixToolset.UI.wixext`: Interfaz gráfica estándar
* `WixToolset.Util.wixext`: Registro, servicios, usuarios
* `WixToolset.Firewall.wixext`: Reglas firewall
* `WixToolset.Bal.wixext`: Bundles / bootstrapper
* `WixToolset.NetFx.wixext`: Detectar .NET

## Constantes predefinidas

WiX ya incorpora constantes predefinidas para usar durante el proceso de instalación, las cuales podemos utilizar para facilitar la instalación, algunas de ellas son las siguientes:

* `ProgramFilesFolder`: Ruta a Program Files
* `DesktopFolder`: Escritorio
* `Installed`: Si ya está instalada la aplicación
* `NOT`: para negar una condición por ejemplo `NOT Installed`.

WiX también incorpora constantes para ser usadas durante el proceso de empaquetamiento:

* `$(ProjectDir)`: Carpeta del .wixproj
* `$(SolutionDir)`: Carpeta solución
* `$(Configuration)`: Debug/Release
* `$(Platform)`: x64/x86

Un ejemplo de uso para obtener la ruta de un archivo ejecutable (nótese que podemos subir en la jerarquía de carpetas):
```xml
<File Source="$(ProjectDir)..\..\app\app.exe" />
```

## Modelo Component/Feature/Directory

### Component

Un MSI no ve las cosas a nivel de archivos, sino a nivel de componentes. Un componente es la unidad mínima que puede ser instalada y puede contener uno o más archivos, configuraciones de registro, accesos directos, variables de entorno para el sistema destino, entre otros. 

El objetivo es encapsular una unidad de instalación, de forma que podamos configurar por ejemplo todo lo necesario para que un archivo quede bien instalado, ya que quizas el archivo es un ejecutable que requiere alguna configuración en el registro del sistema y una variable de entorno. De esta forma aseguramos que se instale todo lo necesario y también al desinstalar se desintale todo lo necesario, de esta forma el sistema sabe como instalar/desinstalar/reparar correctamente cada componente incluso detectar problemas con un componente para repararlo automáticamente. Es por eso que no podemos instalar nada que no forme parte de un componente, así sea un único archivo.

Como los componentes son elementos que necesitan ser rastreados por el sistema, estos necesitan:
* Tener un GUID que no cambie nunca.
    * Si el GUID cambia, entonces Windows perderá el rastro del componente original ya instalado y lo tratará como un componente nuevo futuras instalaciones/actualizaciones.
* Tener marcado exactamente un elemento como principal `KeyPath="yes"`.
    * Si no marcamos ninguno entonces WiX elegira uno.
    * Si marcamos más de uno marcará error la generación del MSI.
    * Si este archivo se elemento corrompido (archivo, registro, variable de entorno,etc) entonces Windows lo repara automáticamente.

**Nota**: la reparación automática funciona porque para los instaladores MSI, Windows guarda una copia completa de todos los `.msi` instalados, aunque esto ocupa espacio en el disco principalmente en instaladores muy pesados.

Un ejemplo de componente es:
```xml
<Component Id="CoreFiles" Guid="AF2305F5-82F4-477D-A14A-C849493B8956">
    <File Source="app\appe.exe" KeyPath="yes"/>
    <File Source="app\config.json"/>
    <File Source="app\helper.dll"/>
</Component>
```

Un Component debe cumplir:

* Todos sus recursos deben instalarse juntos.
* Todos deben estar en el mismo Directory.
* No se deben mover entre versiones.
* Si cambias cosas drásticamente → Cambias el GUID del component.

**Nota**: podemos generar GUIDs con la librería estándar de Python `uuid` mediante `uuid.uuid4()`.

### Feature

Un feature es un elemento que agrupa componentes, de esta forma podemos separar la instalación en features y permitirle al usuario elegir cuales funcionalidades instalar/desinstalar.

### Directory 

Este elemento simplemente define un directorio que será creado durante la instalación en el cual podremos poner los archivos que definamos en los componentes.

### ¿Qué ocurre en desinstalación?

El engine:

* Revisa cada Feature.
* Ve qué Components estaban instalados en el Feature.
* Elimina KeyPath de cada component.
* Elimina todo el Component.
* Nunca elimina archivos sueltos.
* Siempre elimina por Component.

## IDs

Los IDs en los elementos son útiles para poder definirlos en un lugar pero usarlos en otro. En el siguiente ejemplo definimos un Component dentro de un directorio, y más adelante referenciamos el Component en un Feature el cual ya formaliza la instalación del componente, mientras un componente no esté dentro de un Feature no será instalado:

```xml
<Directory Id="ProgramFilesFolder">
    <Directory Id="INSTALLFOLDER" Name="SysApp">
        <Component Id="MainExeComponent" Guid="AF2305F5-82F4-477D-A14A-C849493B8956">
            <File Source="sysapp\sysappe.exe" KeyPath="yes"/>
        </Component>
    </Directory>
</Directory>

<Feature Id="MainFeature">
    <ComponentRef Id="MainExeComponent"/>
</Feature>
```

En otras palabras:

* Directory define dónde.
* Component define qué.
* Feature define si se instala.
* Obviamente podemos meter condicionales en el Feature para que este se instale solamente si se cumple la condición.

## Files

En versiones `<= v3` de WiX, para incorporar todos los archivos de la apliación en la instalación, se tenían que enumerar uno por uno en Components o se podía hacer uso de una herramienta de WiX llamada `harvester` para incorporalos en un archivo separado. En WiX moderno se puede usar `<files>` para incorporarlos directamente, dicho elemento genera automáticamente los Components y los GUIDs para cada archivo:

<Directory Id="INSTALLFOLDER" Name="SysApp">
    <Files Include="$(ProjectDir)..\..\app\**\*.*" />
</Directory>

Esto hace que WiX:

* Genere automáticamente Components.
* Genere GUIDs automáticos.
* Agrupe todo en un ComponentGroup.
* Pero internamente sigue existiendo:
    * Directory → Component → File.
    * Solo que no los escribimos manualmente.

## UI

Para la parte de la UI se maneja separada del modelo de instalación y estos se conectan mediante `Properties`.

Para poder usar la interfaz gráfica podemos usar una extensión con platillas predefinidas muy comunes `WixToolset.UI.wixext`, aunque también podemos crear las nuestras propias.

El modelo de instalación es lo que realmente instala cosas, define:

* Directory
* Component
* Feature
* Files
* Registry
* Services
* CustomActions

La UI (interfaz gráfica) solo recolecta las configuraciones de los Properties que serán usados en el modelo de instalación. Esta define:

* Diálogos
* Botones
* Checkboxes
* Textboxes
* Secuencia de pantallas

### Properties

Una propiedad (property) es como una variable global. Imagina que tienes un checkbox `"Iniciar la aplicación con Windows"`, ese checkbox podría estar ligado a una propiedad `STARTUPWITHWINDOWS`.

Cuando el usuario marca o desmarca el checkbox cambia el valor de esa propiedad y luego, en el modelo de instalación, tú usas esa propiedad como condición para instalar un componente o un Feature:

```xml
<Component Id="StartupRegistry" Guid="PUT-GUID">
  <Condition>STARTUPWITHWINDOWS = 1</Condition>

  <RegistryValue
      Root="HKCU"
      Key="Software\Microsoft\Windows\CurrentVersion\Run"
      Name="SysApp"
      Value="[INSTALLFOLDER]sysappe.exe"
      Type="string"
      KeyPath="yes"/>
</Component>
```

```xml
<UI>
  <Dialog Id="CustomDlg" ...>

    <Control
        Id="StartupCheckbox"
        Type="CheckBox"
        Property="STARTUPWITHWINDOWS"
        CheckBoxValue="1"
        Text="Iniciar con Windows" />

  </Dialog>
</UI>
```

### Suprimir la UI

Podemos invocar el engine de instalación de MSI para realizar la instalación de manera sileciosa pasando los valores de las propiedades mediante parámetros en el comando:

```bash
msiexec /i App.msi /quiet STARTUPWITHWINDOWS=1
```

