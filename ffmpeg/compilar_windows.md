Compilar FFMPEG para Windows con aceleración por GPU
-----------------------------------------------------

El método que usaremos es mediante *Cross-Compiling*, que consiste en compilar par Windows 32 bit o 64 bit desde Linux. En particular usaremos *Ubuntu 22.04*.

Para lograr esto haremos uso del repositorio [ffmpeg-windows-build-helpers](https://github.com/rdp/ffmpeg-windows-build-helpers), en el cual vienen las indicaciones, pero aquí documentaré solución de problemas y configuraciones específicas.

Lo primero que hay que hacer es clonar el repositorio, y ejecutar el script principal:
```bash
$ git clone https://github.com/rdp/ffmpeg-windows-build-helpers.git
$ cd ffmpeg-windows-build-helpers
$ ./cross_compile_ffmpeg.sh
```

**Importante**: a mi me salieron errores derivados de que los saltos de línea eran `\r\n`, por lo que los errores decían cosas como `/usr/bin/env: invalid file or directory bash\r`, o `Invalid symbol '{\r'`, etc. Para solucionar esto bastó con abrir el script con VS Code y en la barra de estado, del lado derecho, cerca de donde está la codificación, hay una opción que dice el tipo de salto de linea usado `CRLF`, solo hay que dar clic y cambiarlo a `LF`.

Una ves hecho eso podremos ejecutar el script `./cross_compile_ffmpeg.sh`, si hacen falta paquetes nos indicará como instalarlos, pero lo que a mi me dijo fue que realizara lo siguiente:
```bash
$ sudo apt-get update
$ sudo apt-get install subversion ragel curl texinfo g++ ed bison flex cvs yasm automake libtool autoconf gcc cmake git make pkg-config zlib1g-dev unzip pax nasm gperf autogen bzip2 autoconf-archive p7zip-full meson clang python3-distutils python-is-python3 -y
```

`./cross_compile_ffmpeg.sh --gcc-cpu-count=12  --build-libmxf=n --disable-nonfree=n --prefer-stable=y --compiler-flavors=multi`