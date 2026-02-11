# WiX v4

WiX es una herramienta moderna para generar instaladores para aplicaciones en Windows. La versión `v4` marca un cambio drástico respecto a la versión `v3`.

El primer cambio es el método de instalación, la versión `v3` se distribuye mediante un instalador `.exe`, mientras que la versión `v4` se distribuye como una herramienta `.NET`, así que necesitamos el `.NET SDK` para poder instalarlo. 

### Instalar .NET SDK

Descargar el instalador de `.NET SDK` (no el `.NET Runtime`) desde [https://dotnet.microsoft.com/es-es/download/dotnet/thank-you/sdk-10.0.102-windows-x64-installer]() e instalarlo.

Validar que se instaló bien con el siguiente comando:
```bash
dotnet --version
```

Si el comando da algún error, entonces no se instaló correctamente o se instaló la herramienta equivocada, revisar que se haya instaldo el `SDK` y no el `Runtime`. O reiniciar la aplicación desde la cual se pretende usar `dotnet` (terminal, VS Code, etc), algunas aplicaciones solo se reinican al cerrar todas sus ventanas.

### Instalar WiX 4

Instalamos WiX v4 como una herramienta de `.NET`, con el siguiente comando:

```bash
dotnet tool install --global wix
```

Esto instalará todo `WiX v4`:

* El compilador
* El linker
* El harvester
* etc

Reiniciamos la aplicación desde la cual queremos usar la herramienta `wix` y ejecutamos el siguiente comando para verificar la correcta instalación:

```bash
wix --version
```

O el comando:

```bash
wix --help
```

## Diferencias respecto a v3

En WiX v3 se usaba:

* `candle`
* `light`
* `heat`

En WiX v4 TODO se hace con un solo comando: `wix`

Ejemplos:

* `wix build`
* `wix extension add`
* `wix harvest`

WiX v4 es un CLI moderno, no tres ejecutables sueltos.

