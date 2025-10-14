# Módulo `ctypes` de Python

Este módulo es una especie de interfaz para invocar funciones desde librerías dinámicas como `dll` en windows o `so` en Linux, por lo tanto funciona tanto en Windows como en Linux.

Resulta que cuando uno programa en C/C++, las funciones manejan dos tipos de convenciones de invocación (calling conventions), `cdecl` (usual para funciones en C/C++) y `stdcall` (usual para la Windows API). La primera permite argumentos variables ya que el invocador se encarga de limpiarlos del Stack y la segunda no porque la función misma se encarga de limpiarlos así que está programada para limpiar un número fijo de elementos.

Una nota importante es que las dlls `win32` como son `kernel32` o `user32` suelen exportar dos versiones de la misma función, una para parámetros ANSI (cadena binaria `b"hola"`) y otra para parámetros Unicode (texto normal `"hola"`). Las funciones ANSI suelen tener una `A` en el nombre y las Unicode una `W` en el nombre, así que por lo general una función que tenga una `W` corresponderá a paráemtros unicode. Por ejemplo:
```python
windll.kernel32.GetModuleHandleA(modulo)
windll.kernel32.GetModuleHandleW(modulo)
```

Las funciones de este módulo solo soportan que se les envíe así directamente parámetros `int`, `bytes`, `str` y `None`, donde None será pasado como `null` y tanto `bytes` como `str` serán pasados como apuntador `char*` or `wchar_t*`. Para otros tipos de parámetros hay que envolverlos en el tipo de dato compatible de `ctypes`. Podemos conocer los tipos de datos soportados por `ctypes` en [tipos de datos fundamentales](https://docs.python.org/3/library/ctypes.html#fundamental-data-types). A continuación vemos algunos ejemplos de creación de distintos tipos de datos para `ctypes`:
```python
from ctypes import c_ulong, c_wchar_p, c_ushort, create_string_buffer, cdll
id = c_ulong(10) # unsigned long
text = c_wchar_p("Hello, World") # unicode string (apuntador al texto)
val = c_ushort(-3) # será 65533 ya que es unsigned short y no contempla negativos
buff = create_string_buffer(20) # texto mutable para funciones necesitan modificarlo
# Podemos consultar o cambiar su valor mediante el atributo `value`
id.value = 20
# Notemos como el entero puede pasarse directamente pero el float tiene que envolverse
# La función `printf` imprime automáticamente el tamaño del texto resultante al final
cdll.msvcrt.printf(b"Entero %d, flotante %f, fin.", 1234, c_double(3.14))
```

El módulo `ctypes` contiene los objetos `cdll` (Windows/Linux) para funciones con convención `cdecl` y además `windll` (Windows) y `oledll` (Windows) para funciones con convención `stdcall`.

Según se ve de los ejemplos de la documentación, parece que estos objetos simplemente especifican la convención que se usará al invocar las funciones por lo que al invocar una función tenemos que usar primero el objeto que indica la convención correcta para la función que vamos a invocar. Por ejemplo, la función `msvcrt.printf` es de convención `cdecl` así que podemos invocarla de cualquiera de las siguientes maneras:
``` python
from ctypes import cdll, windll

cdll.msvcrt.printf(b"spam") # Convención cdecl correcta
windll.msvcrt.printf(b"spam") # Convención stdcall incorrecta (ValueError)
```

Por esta razón, antes de usar una función, tenemos que saber qué convención usa para invocarla mediante el objeto correcto `cdll`, `windll` o `oledll`.


### Módulo `msvcrt`

Este módulo contiene la mayoría de funciones de la librería estándar de C de Microsoft que proporciona funciones estándar de C como `printf`, `malloc`, `memcpy`, etc. 

Python también tiene un módulo integrado llamado `msvcrt` que puede ser importado directamente (`import msvcrt`), el cual contiene algunas pocas de las funciones de `cdll.msvcrt` pero mejor adaptadas para Python.

Según la documentación oficial, el módulo `cdll.msvcrt` puede ser inconpatible el módulo `msvcrt` que viene integrado con Python y recomienda se use el módulo nativo siempre que sea posible. Además algunas de las funciones ya tienen su equivalente directamente en Python, como `cdll.msvcrt.printf()` tiene el equivalente en  `print()` o el módulo `io`.

Esta incompatibilidad ocurre porque `cdll.msvcrt` accede a la dll `msvcrt.dll` que está disponible en el sistema operativo generalmente en `"C:\Windows\System32"`, mientras que Pyton incluye en la compilación una versión específica que puede ser versión superior o inverior lo que podría ocasionar 

```python
# Ejemplo

# En lugar de usar lo siguiente para mostrar texto en la terminal
from ctypes import cdll
cdll.msvcrt.printf(b"Hola")

# Mejor usar
import msvcrt
msvcrt.putch(b'H')
msvcrt.putch(b'o')
msvcrt.putch(b'l')
msvcrt.putch(b'a')

# O mejor aún
print('Hola')
```




